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

## Schritt 3: Control-Plane-Manifests anschauen

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

## Schritt 4: Alle Manifests auf einmal scannen

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

## Schritt 5: Mit NSA-Framework scannen

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

## Schritt 6: CIS-Framework gezielt pruefen

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

---

## Schritt 7: Einzelne Datei scannen — nur der API-Server

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

## Schritt 8: Ein einzelnes Control mit Details

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

## Schritt 9: Scan-Ergebnis als Report speichern

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

## Schritt 10: Aufraumen auf dem CP-Node

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
| `kubescape scan <pfad>` | Alle YAML-Dateien im Pfad scannen |
| `kubescape scan <datei>` | Einzelne YAML-Datei scannen |
| `kubescape scan framework NSA <pfad>` | Scan gegen NSA-Framework |
| `kubescape scan framework cis-v1.10.0 <pfad>` | Scan gegen CIS v1.10.0 |
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
