# Kubescape - Control Plane YAML-Dateien direkt scannen (offline)

## Hintergrund

Kubescape kann nicht nur gegen einen laufenden Cluster scannen,
sondern auch **lokale YAML-Dateien offline analysieren** — ganz ohne
API-Server-Verbindung.

Das ist besonders nuetzlich fuer die Control-Plane-Komponenten:
Die Static-Pod-Manifests unter `/etc/kubernetes/manifests/` liegen direkt
auf dem Node als YAML-Dateien. Kubescape liest sie und prueft sie
gegen Security-Frameworks — kein kubeconfig noetig.

**Warum direkt auf dem Node scannen?**

| Methode | Vorteil |
|---------|---------|
| `kubescape scan` gegen Cluster | Vollstaendiges Bild: alle Namespaces, RBAC, Secrets |
| `kubescape scan <datei/pfad>` | Kein Cluster-Zugang noetig, funktioniert auch im CI/CD |

---

## Schritt 1: Auf den Control-Plane-Node wechseln

Von der Bastion aus:

```
ssh -i /home/tln1/.ssh/id_rsa_k8s_do root@cp
```

---

## Schritt 2: Kubescape auf dem CP-Node installieren

```
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | bash
```

Das Binary landet in `~/.kubescape/bin/`. PATH fuer diese Session setzen:

```
export PATH=$PATH:/root/.kubescape/bin
kubescape version
```

**Erwartete Ausgabe:**
```
Your current version is: v4.0.6
```

---

## Schritt 3: Verfuegbare Frameworks anzeigen

Welche Frameworks kennt kubescape?

```
kubescape list frameworks
```

**Erwartete Ausgabe:**
```
╭──────────────────────╮
│ Supported frameworks │
├──────────────────────┤
│      AllControls     │
│       ArmoBest       │
│      DevOpsBest      │
│         MITRE        │
│          NSA         │
│         SOC2         │
│    cis-aks-t1.2.0    │
│    cis-aks-t1.8.0    │
│    cis-eks-t1.7.0    │
│    cis-eks-t1.8.0    │
│      cis-v1.10.0     │
│      cis-v1.12.0     │
╰──────────────────────╯
```

Kurze Orientierung — welches Framework fuer welchen Zweck:

| Framework | Fokus |
|-----------|-------|
| NSA | Breite Grundhaertung, guter Einstiegspunkt |
| MITRE | Angreiferverhalten (Taktiken und Techniken) |
| cis-v1.10.0 / cis-v1.12.0 | Detaillierte nummerierte CIS-Controls fuer Kubernetes |
| ArmoBest | NSA + MITRE kombiniert, empfohlen fuer schnellen Ueberblick |
| cis-aks-* / cis-eks-* | Cloud-spezifische Varianten (Azure AKS, AWS EKS) |
| SOC2 | Regulatorische Compliance |
| DevOpsBest | Reliabilitaet und Operabilitaet |

---

## Schritt 5: Control-Plane-Manifests anschauen

Was liegt dort ueberhaupt?

```
ls /etc/kubernetes/manifests/
```

**Erwartete Ausgabe:**
```
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Das sind die **Static Pod Manifests** — der kubelet startet diese Pods direkt,
ohne dass ein API-Server noetig ist. Sie definieren kube-apiserver, etcd,
controller-manager und scheduler vollstaendig.

---

## Schritt 6: Alle Manifests auf einmal scannen

```
kubescape scan /etc/kubernetes/manifests/
```

kubescape erkennt automatisch alle YAML-Dateien im Verzeichnis und
gibt eine kompakte Uebersicht der Findings:

**Erwartete Ausgabe:**
```
Security posture overview for repo: '/etc/kubernetes/manifests/'

Workload
| Control name        | Resources | View details                                        |
| HostNetwork access  |     4     | $ kubescape scan control C-0041 /etc/.../           |
| HostPath mount      |     4     | $ kubescape scan control C-0048 /etc/.../           |
| Non-root containers |     4     | $ kubescape scan control C-0013 /etc/.../           |
```

Und ganz unten: die **Highest-stake workloads** — kubescape bewertet,
welche Pods das groesste Risiko darstellen wuerden wenn sie kompromittiert werden.

> **Hinweis zu HostNetwork access (C-0041):**
> Alle vier Control-Plane-Pods (kube-apiserver, scheduler, controller-manager, etcd)
> sind mit `hostNetwork: true` konfiguriert — das ist **by Design** und kein Fehler.
> Sie muessen direkt an die Host-Netzwerk-Ports binden. Dieses Finding ist ein
> bekanntes strukturelles False-Positive fuer Control-Plane-Komponenten.

---

## Schritt 7: Mit NSA-Framework scannen

Jetzt gezielt gegen das NSA-Framework pruefen und den Compliance-Score sehen:

```
kubescape scan framework NSA /etc/kubernetes/manifests/
```

**Erwartete Ausgabe:**
```
Framework scanned: NSA

| Controls | 20 |
| Passed   | 12 |
| Failed   |  8 |

| Severity | Control name                   | Failed | Compliance |
| High     | HostNetwork access             |   4    |     0%     |
| High     | Ensure CPU limits are set      |   4    |     0%     |
| High     | Ensure memory limits are set   |   4    |     0%     |
| Medium   | Non-root containers            |   4    |     0%     |
| Medium   | Allow privilege escalation     |   4    |     0%     |
| Medium   | Ingress and Egress blocked     |   4    |     0%     |
| Low      | Immutable container filesystem |   4    |     0%     |

Resource Summary   4 / 4   60.00%
```

CPU/Memory limits und Non-root containers bei Control-Plane-Pods:
Das sind reale Findings — Control-Plane-Pods haben standardmaessig keine
Resource Limits und laufen als root.

---

## Schritt 8: CIS-Framework gezielt pruefen (online und offline)

### Online-Modus

```
kubescape scan framework cis-v1.10.0 /etc/kubernetes/manifests/
```

Das CIS-Framework hat feinere Controls speziell fuer API-Server-Konfiguration:

**Erwartete Ausgabe:**
```
| Severity | Control name                                          | Compliance |
| High     | CIS-5.2.5 Minimize the admission of containers wi...  |     0%     |
| High     | CIS-5.7.3 Apply Security Context to Your Pods...      |     0%     |

Resource Summary   78.79%
```

CIS hat viele Controls die nur manuell geprueft werden koennen
(markiert als `Action Required *` — kein automatischer Check moeglich).

### Offline-Modus

In air-gapped Umgebungen oder wenn kein Internet verfuegbar ist, koennen
alle Frameworks und Controls vorab heruntergeladen werden.

**Einmalig: Alle Daten herunterladen (benoetigt Internetzugang):**

```
kubescape download artifacts
```

**Erwartete Ausgabe:**
```
Downloaded  attack-tracks    /root/.kubescape/attack-tracks.json
Downloaded  controls-inputs  /root/.kubescape/controls-inputs.json
Downloaded  exceptions       /root/.kubescape/exceptions.json
Downloaded  framework  NSA           /root/.kubescape/nsa.json
Downloaded  framework  cis-v1.10.0   /root/.kubescape/cis-v1.10.0.json
Downloaded  framework  cis-v1.12.0   /root/.kubescape/cis-v1.12.0.json
Downloaded  framework  MITRE         /root/.kubescape/mitre.json
...
```

Was wurde gespeichert?

```
ls ~/.kubescape/
```

**Danach: Scan ohne Internetzugang mit `--use-artifacts-from`:**

```
kubescape scan framework cis-v1.10.0 /etc/kubernetes/manifests/ \
  --use-artifacts-from ~/.kubescape/
```

Das Ergebnis ist identisch — kubescape liest alle Policy-Daten aus dem
lokalen Cache statt sie vom Server abzurufen.

> **Tipp fuer CI/CD:** Die `~/.kubescape/*.json`-Dateien koennen ins
> Container-Image eingebaut oder als Artefakt weitergegeben werden.
> So laeuft kubescape komplett offline ohne externe Abhaengigkeit.

---

## Schritt 9: Einzelne Datei scannen — nur der API-Server

```
kubescape scan /etc/kubernetes/manifests/kube-apiserver.yaml -v
```

`-v` zeigt welche Controls genau fehlschlagen und an welcher Stelle
in der YAML-Datei das Problem liegt:

```
Security posture overview for repo: '/etc/kubernetes/manifests/kube-apiserver.yaml'

Workload
| HostNetwork access  | 1 | $ kubescape scan control C-0041 .../kube-apiserver.yaml -v |
| HostPath mount      | 1 | $ kubescape scan control C-0048 .../kube-apiserver.yaml -v |
| Non-root containers | 1 | $ kubescape scan control C-0013 .../kube-apiserver.yaml -v |
```

---

## Schritt 10: Ein einzelnes Control mit Details

Konkret nachschauen was HostPath-Mounts bei den Control-Plane-Pods bedeutet:

```
kubescape scan control C-0048 /etc/kubernetes/manifests/ -v
```

**Assisted remediation** zeigt den genauen Pfad in der YAML-Datei:
`spec.volumes[x].hostPath` — die Control-Plane-Pods mounten z.B.
`/etc/kubernetes/pki` und `/var/lib/etcd` direkt vom Host.

Auch das ist bei Control-Plane-Komponenten by Design und kein Fehler —
sie muessen auf TLS-Zertifikate und Datenbankdaten zugreifen.

---

## Schritt 11: API-Server haerten — Finding beheben und verifizieren

Wir beheben jetzt das NSA-Finding **"Allow privilege escalation"** (C-0016)
direkt in der kube-apiserver.yaml.

### Schritt 11a: Backup — immer zuerst

```
cp /etc/kubernetes/manifests/kube-apiserver.yaml \
   /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```

> **Wichtig:** Die Datei enthaelt eine kubeadm-verwaltete Annotation:
> `kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint`
> Diese darf nicht entfernt werden — kubeadm benoetigt sie fuer Upgrades.
> Das Backup sichert den exakten Originalzustand inkl. aller Annotations.

### Schritt 11b: Container-securityContext ergaenzen

Aktuelle Situation in der Datei — kein container-level securityContext:

```
grep -n "securityContext\|resources:" /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
# Erwartete Ausgabe:
69:    resources:
101:  securityContext:      <- das ist der Pod-Level securityContext
102:    seccompProfile:
```

`allowPrivilegeEscalation: false` auf Container-Ebene ergaenzen.
Der Eintrag kommt direkt nach dem `resources:`-Block:

```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Vor dem `startupProbe:` einfuegen:

```
    securityContext:
      allowPrivilegeEscalation: false
```

Das vollstaendige Ergebnis im Bereich resources sollte so aussehen:

```
    resources:
      requests:
        cpu: 250m
    securityContext:
      allowPrivilegeEscalation: false
    startupProbe:
```

> **Achtung:** Einrueckung exakt 4 Leerzeichen — das ist Container-Level,
> nicht Pod-Level. Der Pod-Level-Block steht weiter unten bei `hostNetwork`.

### Schritt 11c: API-Server-Neustart abwarten

Der kubelet erkennt die Aenderung und startet den Static Pod automatisch neu.
Von der Bastion aus (anderes Terminal):

```
# auf Bastion:
until KUBECONFIG=/home/tln1/.kube/config kubectl cluster-info &>/dev/null; \
  do sleep 3; done && echo "API-Server wieder erreichbar"
```

### Schritt 11d: Scan nach dem Fix

```
kubescape scan framework NSA /etc/kubernetes/manifests/kube-apiserver.yaml \
  --use-artifacts-from ~/.kubescape/
```

**Vorher:**
```
| Medium   | Allow privilege escalation     |   1    |     0%     |
...
Resource Summary   60.00%
```

**Nachher:**
```
# "Allow privilege escalation" taucht nicht mehr auf

Resource Summary   65.00%
```

Das Control C-0016 ist verschwunden — Finding behoben.

---

## Schritt 11e: Zuruecksetzen (Reset)

Nach der Uebung den Originalzustand wiederherstellen.
Das Backup enthaelt den genauen Stand vor der Aenderung —
inklusive der originalen kubeadm-Annotation:

```
cp /etc/kubernetes/manifests/kube-apiserver.yaml.bak \
   /etc/kubernetes/manifests/kube-apiserver.yaml
```

Warten bis der API-Server mit dem Original neu gestartet ist:

```
until KUBECONFIG=/home/tln1/.kube/config kubectl cluster-info &>/dev/null; \
  do sleep 3; done && echo "Reset erfolgreich"
```

Verifizieren dass keine securityContext-Zeile mehr im Container-Block steht:

```
grep -A2 "resources:" /etc/kubernetes/manifests/kube-apiserver.yaml | head -6
```

```
# Erwartete Ausgabe (kein securityContext-Block zwischen resources und startupProbe):
    resources:
      requests:
        cpu: 250m
    startupProbe:
```

Backup aufraeumen:

```
rm /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```

---

## Schritt 12: Scan-Ergebnis als Report speichern

```
kubescape scan framework NSA /etc/kubernetes/manifests/ \
  --format json \
  --output /tmp/cp-nsa-report.json

kubescape scan framework cis-v1.10.0 /etc/kubernetes/manifests/ \
  --format json \
  --output /tmp/cp-cis-report.json
```

Reports anschauen:

```
cat /tmp/cp-nsa-report.json | python3 -m json.tool | head -40
```

JSON-Reports eignen sich gut fuer Weiterverarbeitung in CI/CD oder
Security-Dashboards.

---

## Schritt 12: Aufraumen auf dem CP-Node

```
rm -f /tmp/cp-nsa-report.json /tmp/cp-cis-report.json
```

Zurueck zur Bastion:

```
exit
```

---

## Zusammenfassung

| Befehl | Was er tut |
|--------|-----------|
| `kubescape list frameworks` | Alle verfuegbaren Frameworks anzeigen |
| `kubescape download artifacts` | Alle Frameworks/Controls lokal speichern (Offline-Vorbereitung) |
| `kubescape scan <pfad>` | Alle YAML-Dateien im Pfad scannen |
| `kubescape scan <datei>` | Einzelne YAML-Datei scannen |
| `kubescape scan framework NSA <pfad>` | Scan gegen NSA-Framework |
| `kubescape scan framework cis-v1.10.0 <pfad>` | Scan gegen CIS v1.10.0 |
| `kubescape scan framework cis-v1.10.0 <pfad> --use-artifacts-from ~/.kubescape/` | Offline-Scan aus lokalem Cache |
| `kubescape scan control C-0041 <pfad> -v` | Ein Control detailliert pruefen |
| `--format json --output report.json` | Ergebnis als JSON speichern |

**Wann welche Methode:**

| Szenario | Empfehlung |
|----------|-----------|
| CI/CD Pipeline (kein Cluster) | `kubescape scan <pfad>` gegen Manifest-Verzeichnis |
| Live-Cluster-Audit | `kubescape scan framework NSA` (ohne Pfad) |
| Control-Plane-Haertung | direkt auf CP-Node: `kubescape scan /etc/kubernetes/manifests/` |
| Einzelne Komponente pruefen | `kubescape scan /etc/kubernetes/manifests/kube-apiserver.yaml -v` |

## Referenzen

  * https://kubescape.io/docs/scanning/
  * https://hub.armosec.io/docs/c-0041 (HostNetwork access)
  * https://hub.armosec.io/docs/c-0048 (HostPath mount)
