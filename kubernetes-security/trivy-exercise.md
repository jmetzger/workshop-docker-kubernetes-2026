# Laufzeitanalyse mit trivy (CIS 1.x / 4.x / 5.x)

## Szenario

Der Cluster laeuft — aber wie sicher ist er wirklich?
Wir spielen den Security-Audit: erst schauen was da ist, dann CVEs pruefen,
dann den ganzen Cluster gegen den CIS Benchmark pruefe.

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

trivy ist ein CNCF Graduated Project (Apache 2.0) — dasselbe Tool das viele Teams
bereits fuer Image-Scanning in der CI/CD-Pipeline nutzen.

---

## Schritt 2: Was laeuft ueberhaupt im Cluster?

Bevor wir scannen: einen Ueberblick verschaffen welche Images im Einsatz sind.

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u
```

Gibt es Images mit `latest`-Tag — das Anti-Pattern schlechthin?

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

`latest` bedeutet: niemand weiss welche Version wirklich laeuft,
Updates sind unkontrolliert. In Produktion: immer feste Tags verwenden.

---

## Schritt 3: Ein Image auf CVEs pruefen

Wir nehmen `nginx:1.27` — das Image das im Workshop bereits laeuft.
Erst nur Critical und High:

```
trivy image --severity CRITICAL,HIGH nginx:1.27
```

> Beim ersten Aufruf laedt trivy die Vulnerability-Datenbank (~90 MB).
> Danach: `--skip-db-update` nutzen.

Die Ausgabe zeigt pro CVE:

```
Library    Vulnerability   Severity   Installed   Fixed in   Title
openssl    CVE-2024-xxxx   HIGH       3.0.2       3.0.9      ...
```

**Wie viele Findings gibt es?** Viele — das ist normal fuer ein Standard-Image.
Entscheidend ist: welche sind **fixbar**?

---

## Schritt 4: Nur fixbare CVEs anzeigen

Mit `--ignore-unfixed` zeigt trivy nur Pakete fuer die bereits ein Fix existiert —
das ist die direkt umsetzbare Arbeitsliste:

```
trivy image --ignore-unfixed --severity CRITICAL,HIGH --skip-db-update nginx:1.27
```

Die Spalte `FIXED-VERSION` zeigt genau auf welche Version man upgraden muesste.

**Vergleich:** Wie sieht das beim gehaerteten Image aus?

```
trivy image --ignore-unfixed --severity CRITICAL,HIGH --skip-db-update \
  nginxinc/nginx-unprivileged:1.27
```

Weniger Findings — `nginx-unprivileged` hat ein schlankeres Basisimage.

---

## Schritt 5: Cluster-weiter Sicherheitsscan

Jetzt nicht nur ein Image, sondern den ganzen Cluster:

```
trivy k8s --report=summary --skip-db-update
```

Die Ausgabe gliedert sich in drei Bereiche:

```
Workload Assessment    <- Pods, Deployments, Jobs (unsere Workloads)
Infra Assessment       <- kube-system, etcd, API-Server
RBAC Assessment        <- Roles, ClusterRoles, Bindings
```

Spalten: **C**ritical / **H**igh / **M**edium / **L**ow / **U**nknown

Schau dir die `default`-Namespace-Zeile an:

```
│ default │ Pod/nginx │ 1 │ 15 │ 90 │ 117 │ 10 │  │ 3 │ 5 │ 11 │
```

Ein nginx-Pod: 1 Critical, 15 High CVEs — und 3 High Misconfigurations.

---

## Schritt 6: Einen Namespace gezielt scannen — nur Misconfigs

Schneller Check des eigenen Namespaces ohne CVE-Download:

```
trivy k8s --report=summary --skip-db-update \
  --scanners misconfig \
  --include-namespaces tln<nr>
```

Was bedeuten die Misconfig-Findings konkret? Detailansicht:

```
trivy k8s --skip-db-update \
  --scanners misconfig \
  --severity HIGH,CRITICAL \
  --include-namespaces tln<nr>
```

Du siehst jetzt genau: "Container should not run as root", "readOnlyRootFilesystem
not set" — das sind die Dinge die wir in der SecurityContext-Uebung gehaertet haben.

---

## Schritt 7: CIS Benchmark Compliance-Report

Der Hoehepunkt: trivy prueft den ganzen Cluster direkt gegen den
CIS Kubernetes Benchmark und zeigt PASS/FAIL pro Control-ID:

```
trivy k8s --compliance=k8s-cis-1.23 --report=summary --skip-db-update
```

Erkennst du die Findings wieder?

```
ID       Severity  Control Name                                  Status  Issues
1.2.18   LOW       --profiling argument is set to false          FAIL    1
1.2.19   LOW       --audit-log-path argument is set              FAIL    1
1.2.30   LOW       --encryption-provider-config argument is set  FAIL    1
5.1.1    HIGH      cluster-admin only used where required        FAIL    2
5.2.6    HIGH      allowPrivilegeEscalation minimized            FAIL    15
5.2.7    MEDIUM    Minimize the admission of root containers     FAIL    16
5.7.3    HIGH      Apply Security Context to Pods and Containers FAIL    46
```

Ordne die Findings den Uebungen zu die wir bereits gemacht haben:

| CIS ID | Finding | Behandelt in |
|--------|---------|-------------|
| 1.2.18 | Profiling aktiv | hardening-control-plane-profiling.md |
| 1.2.19 | Kein Audit Logging | audit-logging-exercise.md |
| 5.1.x | RBAC-Schwachstellen | rbac-exercise-scanning.md |
| 5.2.x | Kein SecurityContext | securitycontext-exercise.md |
| 5.7.3 | 46 Pods ohne SecurityContext | securitycontext-exercise.md |

> Warum `k8s-cis-1.23` und nicht 1.32? Trivy liefert versionierte Compliance-Specs mit.
> Die Kernkontrollen gelten unveraendert fuer 1.32.

Verfuegbare Profile:

```
trivy k8s --help | grep -A8 "compliance string"
```

---

## Schritt 8: RBAC-Lage pruefen

Wer hat cluster-admin?

```
rbac-lookup cluster-admin -o wide
```

Welche Pods mounten automatisch ein SA-Token (sollten sie nicht):

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

---

## Exkurs: Warum CIS 5.1.2 nicht vollstaendig fixbar ist

5.1.2 zeigt 14 Findings — Rollen mit Secrets-Zugriff. Die Ursache:

| ClusterRole | Wer stellt sie wieder her? |
|-------------|---------------------------|
| `cluster-admin`, `admin`, `edit` | kube-controller-manager (by Design) |
| `tigera-operator` | tigera-operator Reconciliation Loop |

Beide Mechanismen setzen Aenderungen zurueck. Das ist **kein Bug**, das ist
Kubernetes-Design.

Der einzige echte Hebel: pruefen ob diese Rollen unnoetig gebunden sind:

```
rbac-lookup admin -o wide
rbac-lookup edit -o wide
```

Sind sie an niemanden gebunden der sie nicht braucht — ist das Finding ein
strukturelles **False-Positive**.

In der Praxis: **Risk Acceptance** — dokumentieren warum das Finding nicht fixbar
ist. Das ist legitimer Security-Prozess.

---

## Zusammenfassung

| Was pruefen | Befehl |
|-------------|--------|
| Welche Images laufen? | `kubectl get pods -A -o jsonpath=...` |
| Image auf CVEs | `trivy image --severity CRITICAL,HIGH <image>` |
| Nur fixbare CVEs | `trivy image --ignore-unfixed ...` |
| Cluster-Gesamtueberblick | `trivy k8s --report=summary --skip-db-update` |
| CIS Compliance | `trivy k8s --compliance=k8s-cis-1.23 --report=summary` |
| Namespace Misconfigs | `trivy k8s --scanners misconfig --include-namespaces tln<nr>` |
| Wer hat cluster-admin? | `rbac-lookup cluster-admin -o wide` |

## Referenzen

  * https://github.com/aquasecurity/trivy
  * https://trivy.dev/latest/docs/target/kubernetes/
  * https://trivy.dev/latest/docs/compliance/
  * https://github.com/FairwindsOps/rbac-lookup
