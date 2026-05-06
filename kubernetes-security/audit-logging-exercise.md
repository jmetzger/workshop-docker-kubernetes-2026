# Übung: API Server Audit Logging aktivieren

## Hintergrund

Kubernetes Audit Logging zeichnet alle Anfragen an den API-Server auf — wer hat was wann gemacht. Das ist wichtig für Compliance, Forensik und Sicherheitsmonitoring.

**Audit Levels:**

| Level | Was wird geloggt |
|-------|------------------|
| `None` | Nichts |
| `Metadata` | Nur Metadaten (User, Zeitstempel, Ressource) |
| `Request` | Metadaten + Request Body |
| `RequestResponse` | Metadaten + Request + Response Body |

**Warum diese Policy-Struktur?**

Die Audit Policy filtert gezielt:

| Ressource | Level | Warum |
|-----------|-------|-------|
| `secrets`, `configmaps` | `Metadata` | Inhalt (Passwörter!) darf nicht geloggt werden — nur wer, wann, was |
| `pods` | `RequestResponse` | Vollständig loggen, um Pod-Erstellungen forensisch nachzuvollziehen |
| kube-proxy watches | `None` | Sehr häufige Systemanfragen — würden Log fluten ohne Mehrwert |
| node gets | `None` | Routineanfragen von kubelets — kein Sicherheitsrelevanz |
| events | `None` | Kubernetes-interne Events — kein Audit-Wert |
| Alles andere | `Metadata` | Sicherheitsnetz: Wer hat was gemacht, ohne Inhalte zu speichern |

## Schritt 1: Aktuellen Zustand prüfen

```
# Vom Bastion-Server aus: auf Control Plane wechseln
ssh cp
```

```
# Läuft Audit Logging bereits?
ls -la /var/log/kubernetes/audit/ 2>/dev/null || echo "Kein Audit Logging aktiv"
```

Erwartete Ausgabe (ohne Audit Logging):
```
Kein Audit Logging aktiv
```

## Schritt 2: Audit Log-Verzeichnis und Policy erstellen

```
# Verzeichnis für Audit Logs anlegen
mkdir -p /var/log/kubernetes/audit
```

```
# Audit Policy in das bereits im API-Server gemountete pki-Verzeichnis legen
# WICHTIG: Nicht /etc/kubernetes/audit-policy.yaml verwenden — das Verzeichnis
# ist im API-Server-Container nicht gemountet!
cat > /etc/kubernetes/pki/audit-policy.yaml << 'EOF'
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Secrets und ConfigMaps: nur Metadaten (kein Inhalt — enthält Passwörter!)
  - level: Metadata
    omitStages:
      - RequestReceived
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]

  # Pods: vollständig loggen (Request + Response) für forensische Analyse
  - level: RequestResponse
    omitStages:
      - RequestReceived
    resources:
      - group: ""
        resources: ["pods"]

  # kube-proxy watches ausblenden — sehr häufig, kein Sicherheitswert
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
      - group: ""
        resources: ["endpoints", "services", "services/status"]

  # Node gets ausblenden — Routine-Kubelet-Anfragen
  - level: None
    userGroups: ["system:nodes"]
    verbs: ["get"]
    resources:
      - group: ""
        resources: ["nodes", "nodes/status"]

  # Events komplett ignorieren
  - level: None
    resources:
      - group: ""
        resources: ["events"]

  # Alles andere: Metadaten (wer, wann, was — ohne Inhalte)
  - level: Metadata
    omitStages:
      - RequestReceived
EOF
```

```
# Policy prüfen
cat /etc/kubernetes/pki/audit-policy.yaml | head -5
```

## Schritt 3: Kube-Apiserver Manifest sichern und anpassen

```
# Backup ins Home-Verzeichnis (NICHT ins manifests-Verzeichnis!)
# Wichtig: Backups in /etc/kubernetes/manifests/ werden von kubelet
# als weitere Static-Pod-Manifeste erkannt und verursachen Konflikte!
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.orig
```

```
# Manifest bearbeiten
nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Im Abschnitt `spec.containers[0].command` nach dem letzten `--tls-...` Eintrag einfügen:

```
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --audit-policy-file=/etc/kubernetes/pki/audit-policy.yaml
```

Im Abschnitt `spec.containers[0].volumeMounts` ergänzen:

```
    - mountPath: /var/log/kubernetes/audit
      name: audit-log
```

Im Abschnitt `spec.volumes` ergänzen:

```
  - hostPath:
      path: /var/log/kubernetes/audit
      type: DirectoryOrCreate
    name: audit-log
```

## Schritt 4: Auf den Neustart warten

Der Kubelet erkennt die Manifest-Änderung automatisch und startet den API-Server neu. Das dauert ca. 30–60 Sekunden.

```
# Warten bis der API-Server wieder antwortet
watch -n2 "kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes"
```

```
# Verifizieren: Audit-Flags im laufenden Prozess
pgrep -a kube-apiserver | grep -o -- "--audit-log-path[^ ]*"
```

Erwartete Ausgabe:
```
--audit-log-path=/var/log/kubernetes/audit/audit.log
```

## Schritt 5: Audit Log verifizieren

```
ls -la /var/log/kubernetes/audit/
```

```
# Erste Einträge anzeigen (JSON-Format)
tail -3 /var/log/kubernetes/audit/audit.log
```

```
# Anzahl der Log-Einträge
wc -l /var/log/kubernetes/audit/audit.log
```

## Schritt 6: Events erzeugen und analysieren

```
# Zurück auf den Bastion-Server
exit
```

```
# Aktionen ausführen um Events zu erzeugen
kubectl get secrets -A
kubectl create namespace audit-test
kubectl delete namespace audit-test
```

```
# Wieder auf den Control Plane
ssh cp
```

## Schritt 7: Log-Analyse mit jq

```
# jq installieren falls nicht vorhanden
apt-get install -y jq 2>/dev/null || true
```

```
# Alle Zugriffe auf Secrets (nur Metadaten — Inhalte sind geschützt)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="secrets") | "\(.stageTimestamp) \(.user.username) \(.verb) \(.objectRef.namespace)/\(.objectRef.name)"' | tail -10
```

```
# Namespace-Erstellungen finden
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="namespaces" and .verb=="create") | "\(.stageTimestamp) CREATE NS: \(.objectRef.name) by \(.user.username)"'
```

```
# Alle Aktionen von kubectl-Usern (nicht System-Komponenten)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.user.username | startswith("system:") | not) | "\(.stageTimestamp) [\(.user.username)] \(.verb) \(.objectRef.resource)/\(.objectRef.name)"' | tail -15
```

```
# Fehlgeschlagene Requests (4xx, 5xx)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.responseStatus.code >= 400) | "HTTP\(.responseStatus.code) \(.user.username) \(.verb) \(.objectRef.resource)/\(.objectRef.name)"' | tail -10
```

```
# Verb-Statistik: Was wird am häufigsten gemacht?
cat /var/log/kubernetes/audit/audit.log | jq -r '.verb' | sort | uniq -c | sort -rn
```

## Aufräumen: Audit Logging deaktivieren

```
# Auf dem Control Plane:
# Original Manifest wiederherstellen
cp /root/kube-apiserver.yaml.orig /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
# Warten bis API-Server neu gestartet ist
watch -n2 "kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes"
```

```
# Prüfen: Audit Logging deaktiviert
pgrep -a kube-apiserver | grep -c audit-log || echo "Audit Logging deaktiviert"
```

```
# Optionales Aufräumen
rm -f /etc/kubernetes/pki/audit-policy.yaml
rm -rf /var/log/kubernetes/audit/
```

```
# Zurück auf den Bastion-Server
exit
```

## Referenzen

  * https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/
  * https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/
  * [CIS Kubernetes Benchmark V1.12.0 - PDF (Link vom Trainer)](http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf) → Abschnitt 3.2
