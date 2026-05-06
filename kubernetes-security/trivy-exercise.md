# Laufzeitanalyse mit trivy - CVEs, Misconfigs und CIS Compliance

## Hintergrund

Haerten allein reicht nicht — man muss auch wissen, was **im laufenden System**
gerade unsicher ist. Drei typische Fragen:

| Frage | Tool |
|-------|------|
| Welche Pods laufen privilegiert oder als root? | `kubectl` Queries |
| Haben Images bekannte CVEs? | `trivy image` |
| Gesamtbild: Misconfigs + CVEs + RBAC? | `trivy k8s --report=summary` |
| Wo stehe ich gegen den CIS Benchmark? | `trivy k8s --compliance=k8s-cis-1.23` |
| Wer hat kritische RBAC-Rechte? | `rbac-lookup` |

**trivy** ist ein CNCF Graduated Project (Apache 2.0) von Aqua Security —
dasselbe Tool das viele Teams bereits fuer Image-Scanning nutzen.

---

## Schritt 1: trivy installieren

Installation ins Home-Verzeichnis — kein root noetig:

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

Gesamtuebersicht — Vulnerabilities, Misconfigs und RBAC in einem Lauf:

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
> Danach immer `--skip-db-update` nutzen um Zeit zu sparen.

---

## Schritt 3: CIS Benchmark Compliance-Report

Der eigentliche Highlight: trivy prueft den Cluster direkt gegen den
**CIS Kubernetes Benchmark** und zeigt PASS/FAIL pro Control-ID:

```
trivy k8s --compliance=k8s-cis-1.23 --report=summary --skip-db-update
```

**Auszug aus der Ausgabe unseres Clusters:**

```
ID       Severity  Control Name                                          Status  Issues
1.2.18   LOW       --profiling argument is set to false                  FAIL    1
1.2.19   LOW       --audit-log-path argument is set                      FAIL    1
1.2.30   LOW       --encryption-provider-config argument is set          FAIL    1
5.1.1    HIGH      cluster-admin role only used where required            FAIL    2
5.1.2    HIGH      Minimize access to secrets                            FAIL    14
5.1.3    HIGH      Minimize wildcard use in Roles and ClusterRoles       FAIL    8
5.2.6    HIGH      allowPrivilegeEscalation minimized                    FAIL    15
5.2.7    MEDIUM    Minimize the admission of root containers             FAIL    16
5.7.3    HIGH      Apply Security Context to Pods and Containers         FAIL    46
```

Bekannte Findings aus unseren Uebungen:

| CIS ID | Finding | Unsere Uebung |
|--------|---------|--------------|
| 1.2.18 | Profiling aktiv | hardening-control-plane-profiling.md |
| 1.2.19 | Kein Audit Logging | audit-logging-exercise.md |
| 1.2.30 | Keine etcd Encryption at Rest | - |
| 5.1.x  | RBAC-Schwachstellen | rbac-exercise-scanning.md |
| 5.2.x  | Kein SecurityContext | securitycontext-exercise.md |

Verfuegbare Compliance-Profile:

```
trivy k8s --help | grep -A8 compliance
```

```
--compliance string   Built-in compliance's:
  - k8s-nsa-1.0          (NSA Kubernetes Hardening Guide)
  - k8s-cis-1.23         (CIS Kubernetes Benchmark)
  - eks-cis-1.4          (CIS Amazon EKS)
  - rke2-cis-1.24        (CIS RKE2)
  - k8s-pss-baseline-0.1 (Pod Security Standards Baseline)
```

> Warum `k8s-cis-1.23` und nicht 1.32? Trivy liefert versionierte
> Compliance-Specs mit — neuere Versionen kommen mit kuenftigen Releases.
> Die Kernkontrollen von 1.23 gelten aber unveraendert fuer 1.32.

---

## Schritt 4: Nur Misconfigs scannen (schneller)

Ohne Image-CVE-Scan — ideal fuer den taeglichen Check:

```
trivy k8s --report=summary --skip-db-update --scanners misconfig,rbac
```

Eigenen Namespace scannen:

```
trivy k8s --report=summary --skip-db-update \
  --scanners misconfig \
  --include-namespaces tln<nr>
```

---

## Schritt 5: Image auf CVEs pruefen

```
trivy image --skip-db-update --severity CRITICAL,HIGH nginx:1.27
```

Image eines laufenden Pods direkt pruefen:

```
kubectl get pod nginx -o jsonpath='{.spec.containers[*].image}'
trivy image --skip-db-update --severity CRITICAL,HIGH nginx:1.27
```

**Was du siehst:** CVE-ID, betroffenes Paket, installierte Version,
ob ein Fix verfuegbar ist (`fixed in: ...`).

---

## Schritt 6: Kubectl-Queries fuer schnelle manuelle Checks

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
" | grep -v "kube-system\|calico"
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
" | grep -v "kube-system\|calico"
```

### Wer hat cluster-admin?

```
rbac-lookup cluster-admin -o wide
```

**Ausgabe aus unserem Cluster:**
```
SUBJECT                         SCOPE          ROLE
Group/kubeadm:cluster-admins    cluster-wide   ClusterRole/cluster-admin
```

---

## Exkurs: CIS 5.1.2 - Warum es nicht vollstaendig fixbar ist

CIS 5.1.2 ("Minimize access to secrets") zeigt in unserem Cluster 14 Findings.
Die betroffenen Rollen:

| ClusterRole | Secrets-Zugriff | Wer stellt sie wieder her? |
|-------------|----------------|---------------------------|
| `cluster-admin` | `*` auf `*` | kube-controller-manager |
| `admin` | `get/list/watch` | kube-controller-manager |
| `edit` | `get/list/watch` | kube-controller-manager |
| `tigera-operator` | `get/list/watch` | tigera-operator selbst |

**kube-controller-manager** stellt Built-in Rollen bei jedem Start automatisch
wieder her — das ist by Design. Eine Aenderung waere nach dem naechsten
Controller-Neustart weg.

**tigera-operator** hat seinen eigenen Reconciliation Loop und braucht
Secrets-Zugriff genuinen fuer TLS-Zertifikate und Calico-Credentials.
Auch hier: Aenderung wird wiederhergestellt.

**Der einzige echte Hebel** ist zu pruefen, ob diese Rollen unnoetig gebunden sind:

```
rbac-lookup admin -o wide
rbac-lookup edit -o wide
```

Sind diese Rollen an niemanden gebunden der sie nicht braucht — ist das Finding
ein strukturelles **False-Positive** des Standard-Setups.

In der Praxis: **Risk Acceptance** — dokumentieren warum das Finding nicht fixbar
ist und dass das Risiko bekannt und bewertet wurde. Das ist legitimer
Security-Prozess.

---

## Schritt 7: Veraltete Images im Cluster finden

"Veraltet" hat zwei Bedeutungen — beide sind relevant:

| Was | Bedeutung | Tool |
|-----|-----------|------|
| Pakete mit bekannten CVEs die gefixt wurden | Image hat CVEs, Upgrade wuerde sie beheben | `trivy image --ignore-unfixed` |
| Neuere Image-Tags verfuegbar (z.B. 1.24 → 1.27) | Kein eingebautes Kubernetes-Tool, braucht Registry-Abfrage | Renovate / Dependabot |

### Alle laufenden Images auflisten

Erst mal sehen, was ueberhaupt im Cluster laeuft:

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u
```

### latest-Tags finden — Anti-Pattern

`latest` bedeutet: man weiss nie welche Version genau laeuft, Updates sind unkontrolliert.

```
kubectl get pods -A -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data['items']:
    ns = item['metadata']['namespace']
    name = item['metadata']['name']
    for c in item['spec'].get('containers', []):
        img = c['image']
        if img.endswith(':latest') or ':' not in img:
            print(f'LATEST TAG: {ns}/{name} -> {img}')
"
```

### Nur fixbare CVEs anzeigen (`--ignore-unfixed`)

Mit `--ignore-unfixed` zeigt trivy ausschliesslich Pakete, fuer die ein Update
bereits verfuegbar ist — das ist die direkt umsetzbare Liste:

```
trivy image --ignore-unfixed --severity CRITICAL,HIGH --skip-db-update nginx:1.27
```

Die Spalte `FIXED-VERSION` zeigt genau, auf welche Paketversion man upgraden muesste.
Ist das Basisimage zu alt, ist ein Image-Update der einzige Weg.

### Alle Cluster-Images automatisch scannen

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u \
  | while read img; do
      result=$(trivy image --ignore-unfixed --severity CRITICAL,HIGH \
               --skip-db-update --quiet "$img" 2>/dev/null \
               | grep -c "CRITICAL\|HIGH" || true)
      if [ "$result" -gt 0 ]; then
          echo "FINDINGS ($result):  $img"
      else
          echo "OK:                 $img"
      fi
  done
```

**Beispielausgabe:**
```
FINDINGS (12):  nginx:1.27
OK:             nginxinc/nginx-unprivileged:1.27
FINDINGS (3):   bitnami/nginx:1.25
```

### Grenze von Trivy: Image-Tag-Updates

Trivy prueft **Paket-CVEs innerhalb eines Images** — es weiss aber nicht, dass
z.B. `nginx:1.24` existiert und `nginx:1.27` aktueller ist.
Fuer Tag-Level-Updates (Dockerfile, Kubernetes-Manifeste) braucht man:

```
Renovate (Open Source, selbst-hostbar)
Dependabot (GitHub, nur GitHub-Repos)
```

Beide eroeffnen automatisch Pull Requests wenn neue Image-Tags verfuegbar sind.

---

## Zusammenfassung: Wann welches Tool?

| Situation | Befehl |
|-----------|--------|
| CIS Compliance-Report | `trivy k8s --compliance=k8s-cis-1.23 --report=summary` |
| Schneller Gesamtueberblick | `trivy k8s --report=summary --skip-db-update` |
| Image auf CVEs | `trivy image --severity CRITICAL,HIGH nginx:1.27` |
| Nur Misconfigs | `trivy k8s --scanners misconfig --report=summary` |
| Wer hat cluster-admin? | `rbac-lookup cluster-admin -o wide` |
| Pods ohne SecurityContext | kubectl + python3 Query (siehe oben) |
| Alle laufenden Images auflisten | kubectl jsonpath Query (siehe oben) |
| latest-Tags im Cluster finden | kubectl + python3 Query (siehe oben) |
| Fixbare CVEs in einem Image | `trivy image --ignore-unfixed --severity CRITICAL,HIGH <image>` |
| Alle Cluster-Images auf fixbare CVEs | kubectl + while-Schleife + trivy (siehe oben) |
| Neuere Image-Tags verfuegbar? | Renovate / Dependabot (kein Trivy-Feature) |

## Referenzen

  * https://github.com/aquasecurity/trivy
  * https://trivy.dev/latest/docs/target/kubernetes/
  * https://trivy.dev/latest/docs/compliance/
  * https://github.com/FairwindsOps/rbac-lookup
