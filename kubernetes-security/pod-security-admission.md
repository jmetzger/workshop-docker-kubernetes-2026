# Pod Security Admission — Namespace-weite Durchsetzung (CIS 5.2.1)

## Hintergrund: Warum PSA?

Die `securitycontext-exercise` zeigt, *wie* man Pods korrekt konfiguriert.
Das Problem: SecurityContext ist ein **"Trust the Developer"**-Ansatz. Jeder muss
es jedes Mal richtig machen — und in einem Team mit vielen Entwicklern, unter
Zeitdruck oder bei neuen Mitarbeitern passiert es unvermeidlich, dass jemand
`runAsNonRoot` vergisst, Capabilities nicht droppt oder `allowPrivilegeEscalation`
nicht setzt. Das Ergebnis ist ein root-Container in Produktion, ohne dass jemand
es bemerkt.

PSA ist die **technische Durchsetzung** dieser Anforderungen auf Cluster-Ebene:

- **Kein Vertrauen in die Konfiguration des Einzelnen** — der Cluster lehnt
  nicht-konforme Pods ab, bevor sie starten
- **Sofortiges Feedback** — der Entwickler sieht beim `kubectl apply` exakt,
  was fehlt, nicht erst im Monitoring
- **Defense in Depth** — auch wenn ein Angreifer einen Pod einschleusen kann,
  verhindert PSA, dass er privilegiert laeuft

PSA ist seit Kubernetes 1.25 fest eingebaut — kein zusaetzlicher Controller,
kein Helm Chart, nur Namespace-Labels.

| CIS | Anforderung | PSA-Profil |
|-----|-------------|------------|
| **5.2.1** | Mindestens ein aktiver Policy-Enforcement-Mechanismus | PSA aktivieren |
| 5.2.2 | Keine privilegierten Container | `baseline` / `restricted` |
| 5.2.6 | Kein `allowPrivilegeEscalation` | `restricted` |
| 5.2.7 | Keine Root-Container | `restricted` |

### Die drei Modi

| Modus | Label | Wirkung |
|-------|-------|---------|
| `enforce` | `pod-security.kubernetes.io/enforce` | Pod wird **abgelehnt** |
| `warn`    | `pod-security.kubernetes.io/warn`    | Pod laeuft, aber **Warnung** im Terminal |
| `audit`   | `pod-security.kubernetes.io/audit`   | Pod laeuft, Verstoss landet im **Audit-Log** |

### Die drei Profile

| Profil | Beschreibung |
|--------|-------------|
| `privileged` | Keine Einschraenkungen |
| `baseline` | Verhindert bekannte Privilege-Escalation-Vektoren |
| `restricted` | Streng: non-root, no caps, seccomp, kein hostPath |

> **Wichtig:** PSA greift nur auf neue Pods — bereits laufende Container werden nicht nachtraeglich beendet.

---

## Schritt 1: Namespace mit PSA-Labels anlegen

```
cd
mkdir -p manifests/psa
cd manifests/psa
```

```
nano 01-ns.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-restricted
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 01-ns.yml
kubectl get namespace psa-restricted --show-labels
```

**Erwartete Ausgabe (Labels sichtbar):**
```
NAME          STATUS   AGE   LABELS
psa-restricted   Active   5s    pod-security.kubernetes.io/enforce=restricted,...
```

---

## Schritt 2: Root-Container wird abgelehnt

```
nano 02-nginx-root.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-root
  namespace: psa-restricted
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 02-nginx-root.yml
```

**Erwartete Ablehnung:**
```
Error from server (Forbidden): pods "nginx-root" is forbidden:
violates PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false (container "nginx" must set
  securityContext.allowPrivilegeEscalation=false),
  unrestricted capabilities (container "nginx" must set
  securityContext.capabilities.drop=["ALL"]),
  runAsNonRoot != true (pod or container "nginx" must set
  securityContext.runAsNonRoot=true),
  seccompProfile (pod or container "nginx" must set
  securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

PSA blockiert den Pod direkt beim `kubectl apply` — der Container wird nie gestartet.

---

## Schritt 3: Compliant Pod laeuft durch

```
nano 03-nginx-ok.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ok
  namespace: psa-restricted
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: nginx
    image: nginxinc/nginx-unprivileged:1.27
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

```
kubectl apply -f 03-nginx-ok.yml
kubectl get pod nginx-ok -n psa-restricted
```

**Erwartete Ausgabe:**
```
NAME       READY   STATUS    RESTARTS   AGE
nginx-ok   1/1     Running   0          5s
```

---

## Schritt 4: warn-Modus verstehen

Im `warn`-Modus (`enforce: baseline`) laeuft der Pod durch — der Entwickler sieht
aber sofort die Warnung im Terminal. Gut fuer eine Einfuehrungsphase vor echtem Enforcement.

```
nano 04-ns-warn.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-warn
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 04-ns-warn.yml
```

```
nano 05-nginx-warn.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-warn
  namespace: psa-warn
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 05-nginx-warn.yml
```

**Erwartete Ausgabe — Pod laeuft, aber Warnung:**
```
Warning: would violate PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false, ...
pod/nginx-warn created
```

---

## Grenzen von PSA — und wann Kyverno oder OPA Gatekeeper

PSA ist in Kubernetes eingebaut und braucht keine zusaetzlichen Komponenten.
Fuer alles darueber hinaus braucht man ein externes Admission-Control-Framework.

| Was PSA nicht kann | Loesung |
|-------------------|---------|
| Nur Images aus erlaubten Registries zulassen | Kyverno / OPA |
| Fehlende SecurityContext automatisch ergaenzen (Mutation) | Kyverno |
| Cluster-weite Policies ohne Namespace-Labels | Kyverno / OPA |
| Custom-Rules (z.B. Labelpflicht fuer alle Deployments) | Kyverno / OPA |
| Nicht-Pod-Ressourcen validieren (Ingress, ConfigMap) | Kyverno / OPA |
| Verschiedene Regeln pro Team / Projekt | Kyverno / OPA |

### Kyverno vs. OPA Gatekeeper

**Kyverno** — empfohlen fuer die meisten Teams:
- Policies sind normales Kubernetes-YAML — keine neue Sprache
- Unterstuetzt Validation, Mutation und Generation
- Einfach installiert (Helm), grosse Community-Bibliothek mit fertigen Policies
- Gut geeignet fuer: Image-Registry-Restriction, Auto-Labels, Seccomp-Enforcement

**OPA Gatekeeper** — fuer komplexe Anforderungen:
- Policies in Rego (eigene Sprache) — maechtiger, aber steile Lernkurve
- Sinnvoll wenn: OPA bereits fuer andere Systeme im Einsatz ist (z.B. Terraform, Envoy)
- Besser fuer sehr individuelle Unternehmensregelwerke

**Empfehlung:** PSA zuerst aktivieren (kein Overhead, sofort wirksam), dann Kyverno
fuer alles was PSA nicht kann. OPA Gatekeeper nur wenn Rego bereits bekannt ist
oder OPA bereits im Stack vorhanden ist.

---

## Aufraeumen

```
kubectl delete namespace psa-restricted psa-warn
```

---

## Zusammenfassung

| Szenario | Ergebnis |
|----------|----------|
| Root-Container in `enforce: restricted` Namespace | Abgelehnt |
| Root-Container in `warn: restricted` / `enforce: baseline` Namespace | Warnung, laeuft |
| Compliant Pod in `enforce: restricted` Namespace | Laeuft |
| Namespace ohne PSA-Labels | Keine Einschraenkung |

## Referenzen

  * https://kubernetes.io/docs/concepts/security/pod-security-admission/
  * https://kubernetes.io/docs/concepts/security/pod-security-standards/
  * https://kyverno.io/
  * https://open-policy-agent.github.io/gatekeeper/
  * CIS Kubernetes Benchmark V1.12.0, Sektion 5.2
