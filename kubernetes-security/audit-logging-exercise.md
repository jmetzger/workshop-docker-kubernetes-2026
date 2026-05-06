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

## Schritt 1: Aktuellen Zustand prüfen

```
# Vom Bastion-Server aus: auf Control Plane wechseln
ssh cp
```

```
# Läuft Audit Logging bereits?
ls -la /var/log/kubernetes/audit/
```

Erwartete Ausgabe (ohne Audit Logging):
```
ls: cannot access '/var/log/kubernetes/audit/': No such file or directory
```

## Schritt 2: Audit Policy erstellen

Die Policy bestimmt, welche Ereignisse auf welchem Level geloggt werden.

```
# Auf dem Control Plane Node ausführen
mkdir -p /var/log/kubernetes/audit
```

```
cat > /etc/kubernetes/audit-policy.yaml << 'EOF'
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Secrets und ConfigMaps: nur Metadaten (kein Inhalt!)
  - level: Metadata
    omitStages:
      - RequestReceived
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]

  # Pods: Vollständiges Request + Response loggen
  - level: RequestResponse
    omitStages:
      - RequestReceived
    resources:
      - group: ""
        resources: ["pods"]

  # Kube-Proxy watches rausfiltern (zu viel Lärm)
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
      - group: ""
        resources: ["endpoints", "services", "services/status"]

  # Node gets rausfiltern
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

  # Alles andere: Metadaten
  - level: Metadata
    omitStages:
      - RequestReceived
EOF
```

```
# Prüfen ob die Datei korrekt erstellt wurde
cat /etc/kubernetes/audit-policy.yaml
```

## Schritt 3: Kube-Apiserver Manifest sichern und anpassen

```
# Backup des aktuellen Manifests
cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```

```
# Audit-Parameter zum API-Server hinzufügen
# ACHTUNG: Nach dem Speichern startet der API-Server automatisch neu!
```

```
# Folgende Zeilen im Abschnitt "command:" nach dem letzten --tls-... Eintrag einfügen:
#   - --audit-log-path=/var/log/kubernetes/audit/audit.log
#   - --audit-log-maxage=30
#   - --audit-log-maxbackup=10
#   - --audit-log-maxsize=100
#   - --audit-policy-file=/etc/kubernetes/audit-policy.yaml

nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Der `spec.containers[0].command` Abschnitt soll am Ende so aussehen:

```
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
```

Außerdem müssen zwei `volumeMounts` und zwei `volumes` ergänzt werden.

Im Abschnitt `volumeMounts:` ergänzen:

```
    - mountPath: /var/log/kubernetes/audit
      name: audit-log
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit-policy
      readOnly: true
```

Im Abschnitt `volumes:` ergänzen:

```
  - hostPath:
      path: /var/log/kubernetes/audit
      type: DirectoryOrCreate
    name: audit-log
  - hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
    name: audit-policy
```

## Schritt 4: Auf den Neustart warten

Der Kubelet erkennt die Manifest-Änderung und startet den API-Server automatisch neu. Das dauert ca. 30–60 Sekunden.

```
# Warten bis der API-Server wieder antwortet (vom Control Plane Node)
watch -n2 "kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes"
```

```
# Alternativ: Prozessargumente prüfen
pgrep -a kube-apiserver | grep audit-log
```

Erwartete Ausgabe wenn Audit Logging aktiv:
```
12345 kube-apiserver ... --audit-log-path=/var/log/kubernetes/audit/audit.log ...
```

## Schritt 5: Audit Log verifizieren

```
# Log-Verzeichnis prüfen
ls -la /var/log/kubernetes/audit/
```

```
# Erste Log-Einträge anzeigen (JSON-Format)
tail -f /var/log/kubernetes/audit/audit.log | head -5
```

```
# Vom Bastion-Server: Ereignisse erzeugen
exit
```

```
# Einige Aktionen ausführen um Events zu erzeugen
kubectl get secrets -A
kubectl get pods -A
kubectl create namespace audit-test
kubectl delete namespace audit-test
```

```
# Zurück auf den Control Plane
ssh cp
```

```
# Wie viele Log-Einträge gibt es?
wc -l /var/log/kubernetes/audit/audit.log
```

## Schritt 6: Log-Analyse mit jq

```
# jq installieren falls nicht vorhanden
apt-get install -y jq 2>/dev/null || true
```

```
# Alle User/ServiceAccounts die auf Secrets zugegriffen haben
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="secrets") | "\(.stageTimestamp) \(.user.username) \(.verb) \(.objectRef.namespace)/\(.objectRef.name)"' | tail -20
```

```
# Alle Pod-Erstellungen (create-Events mit vollständigem Body)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="pods" and .verb=="create") | "\(.stageTimestamp) \(.user.username) create pod: \(.objectRef.name) in \(.objectRef.namespace)"' | tail -10
```

```
# Alle Aktionen eines bestimmten Users
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.user.username != "system:node:k8s-tln1-cp") | "\(.stageTimestamp) [\(.user.username)] \(.verb) \(.objectRef.resource)/\(.objectRef.name)"' | tail -20
```

```
# Fehlgeschlagene Requests (403, 404, 409...)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.responseStatus.code >= 400) | "\(.stageTimestamp) HTTP\(.responseStatus.code) \(.user.username) \(.verb) \(.objectRef.resource)/\(.objectRef.name)"' | tail -10
```

```
# Zusammenfassung: Verb-Statistik
cat /var/log/kubernetes/audit/audit.log | jq -r '.verb' | sort | uniq -c | sort -rn
```

## Schritt 7: Namespace-Erstellung im Log finden

```
# Namespace-Operationen suchen
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="namespaces") | "\(.stageTimestamp) \(.user.username) \(.verb) \(.objectRef.name)"'
```

## Aufräumen: Audit Logging deaktivieren

```
# Backup wiederherstellen (deaktiviert Audit Logging)
cp /etc/kubernetes/manifests/kube-apiserver.yaml.bak /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
# Warten bis API-Server neu gestartet ist
watch -n2 "kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes"
```

```
# Prüfen: Audit Logging ist wieder deaktiviert
pgrep -a kube-apiserver | grep -c audit-log || echo "Audit Logging deaktiviert"
```

```
# Log-Verzeichnis aufräumen (optional)
rm -rf /var/log/kubernetes/audit/
```

```
# Zurück zum Bastion-Server
exit
```

## Referenzen

  * https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/
  * https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/
  * [CIS Kubernetes Benchmark V1.12.0 - PDF (Link vom Trainer)](http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf) → Abschnitt 3.2
