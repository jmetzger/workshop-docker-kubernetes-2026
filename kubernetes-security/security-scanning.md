# Laufzeitanalyse - Was laeuft gerade im Cluster?

## Hintergrund

Haerten allein reicht nicht — man muss auch wissen, was **im laufenden System**
gerade unsicher ist. Drei typische Fragen:

| Frage | Tool |
|-------|------|
| Welche Pods laufen privilegiert oder als root? | `kubectl` |
| Haben Images bekannte CVEs? | `trivy` |
| Welche ServiceAccounts haben zu viele Rechte? | `rbac-lookup` |
| Gesamtbild: Misconfigs + CVEs + RBAC auf einen Blick? | `trivy k8s` |

---

## Schritt 1: trivy installieren

```
curl -sL https://github.com/aquasecurity/trivy/releases/download/v0.70.0/trivy_0.70.0_Linux-64bit.tar.gz \
  | tar -xz -C ~/bin trivy

trivy --version
```

**Erwartete Ausgabe:**
```
Version: 0.70.0
```

---

## Schritt 2: Cluster-weiter Sicherheitsscan

Gesamtuebersicht ueber den Cluster — Vulnerabilities, Misconfigs und RBAC:

```
trivy k8s --report=summary
```

Die Ausgabe zeigt drei Bereiche:

```
Workload Assessment      <- Pods, Deployments, Jobs
Infra Assessment         <- kube-system, etcd, API-Server
RBAC Assessment          <- Roles, ClusterRoles, Bindings
```

Spalten: **C**ritical / **H**igh / **M**edium / **L**ow / **U**nknown

> Beim ersten Aufruf laedt trivy die Vulnerability-Datenbank (~90 MB).
> Danach mit `--skip-db-update` starten um Zeit zu sparen.

---

## Schritt 3: Nur Misconfigs und RBAC scannen (schneller)

Ohne Image-CVE-Scan — ideal fuer schnelle Checks:

```
trivy k8s --report=summary --skip-db-update --scanners misconfig,rbac
```

Einen einzelnen Namespace scannen:

```
trivy k8s --report=summary --skip-db-update \
  --scanners misconfig,rbac \
  --include-namespaces tln<nr>
```

---

## Schritt 4: Image auf CVEs pruefen

Das `nginx:1.27`-Image aus dem Workshop scannen:

```
trivy image --skip-db-update nginx:1.27
```

Nur Critical und High anzeigen:

```
trivy image --skip-db-update --severity CRITICAL,HIGH nginx:1.27
```

**Was du siehst:** CVEs mit CVE-ID, betroffenes Paket, installierte Version
und ob ein Fix verfuegbar ist (`fixed in: ...`).

Ein einzelnes Image aus dem Cluster scannen:

```
# Image eines laufenden Pods herausfinden
kubectl get pod nginx -o jsonpath='{.spec.containers[*].image}'

# Image direkt scannen
trivy image --skip-db-update --severity CRITICAL,HIGH nginx:1.27
```

---

## Schritt 5: Kubectl-Queries - schnelle manuelle Checks

### Welche Pods laufen ohne SecurityContext?

```
kubectl get pods -A -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data['items']:
    ns = item['metadata']['namespace']
    name = item['metadata']['name']
    psc = item['spec'].get('securityContext', {})
    if not psc.get('runAsNonRoot') and psc.get('runAsUser', 0) == 0:
        print(f'KEIN SC: {ns}/{name}')
" | grep -v "^kube-system\|^calico"
```

### Welche Pods mounten automatisch ein SA-Token?

```
kubectl get pods -A -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data['items']:
    ns = item['metadata']['namespace']
    name = item['metadata']['name']
    if item['spec'].get('automountServiceAccountToken', True):
        print(f'TOKEN GEMOUNTET: {ns}/{name}')
" | grep -v "^kube-system\|^calico"
```

### Welche ServiceAccounts haben cluster-admin Rechte?

```
rbac-lookup cluster-admin -o wide
```

**Ausgabe aus unserem Cluster:**
```
SUBJECT                         SCOPE          ROLE
Group/kubeadm:cluster-admins    cluster-wide   ClusterRole/cluster-admin
```

Alle Gruppen mit Cluster-Rechten:

```
rbac-lookup -k group
```

---

## Schritt 6: Findings aus dem eigenen Namespace analysieren

```
trivy k8s --report=summary --skip-db-update \
  --scanners misconfig \
  --include-namespaces tln<nr>
```

Ein konkretes Finding genauer anschauen:

```
trivy k8s --scanners misconfig \
  --skip-db-update \
  --include-namespaces tln<nr> \
  --severity HIGH,CRITICAL
```

---

## Zusammenfassung: Wann welches Tool?

| Situation | Befehl |
|-----------|--------|
| Schneller Gesamtueberblick | `trivy k8s --report=summary --skip-db-update` |
| Image auf CVEs pruefen | `trivy image nginx:1.27` |
| Nur Misconfigs | `trivy k8s --scanners misconfig --report=summary` |
| Wer hat cluster-admin? | `rbac-lookup cluster-admin -o wide` |
| Pods ohne SecurityContext | kubectl + python3 Query |
| SA-Token gemountet? | kubectl + python3 Query |

## Referenzen

  * https://github.com/aquasecurity/trivy
  * https://trivy.dev/latest/docs/target/kubernetes/
  * https://github.com/FairwindsOps/rbac-lookup
