# Trivy: Image-Scanning und CIS Compliance Check

## Hintergrund

Trivy (Aqua Security) ist ein umfassendes Security-Scanner-Tool. Im Kubernetes-Kontext
deckt es zwei unterschiedliche Bereiche ab — mit unterschiedlichem CIS-Bezug:

| Scan-Modus | Was es prueft | CIS-Bezug |
|------------|--------------|-----------|
| `trivy image <image>` | CVEs in OS-Paketen und Libraries | Kein CIS Kubernetes Kapitel (Supply Chain / NIST SP 800-190) |
| `trivy k8s --compliance=k8s-cis` | Cluster-Konfiguration gegen CIS Benchmark | Direkt CIS 1.x, 4.x, 5.x |

> Trivy ist ein **Pruefw­erkzeug** — es hat selbst kein CIS-Kapitel.
> Die Findings aus `trivy k8s` referenzieren CIS-Checks, die wir in diesem Workshop
> bereits haertet haben (kube-bench, SecurityContext, PSA).

---

## Schritt 1: Trivy installieren (auf dem Control-Plane-Node)

```
ssh root@<control-plane-ip>
```

```
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sudo sh -s -- -b /usr/local/bin
trivy --version
```

---

## Schritt 2: Image scannen — CVEs, kein CIS-Bezug

```
trivy image nginx:1.27
```

Ausgabe zeigt CVEs in OS-Paketen — aber keine CIS-Benchmark-Nummern.
Das ist Supply-Chain-Security, nicht Kubernetes-Cluster-Compliance.

Vergleich: minimales non-root Image hat deutlich weniger Findings:

```
trivy image nginxinc/nginx-unprivileged:1.27
```

**Fazit:** Je groesser das Base-Image, desto mehr Angriffsfläche.
Weniger Pakete = weniger CVEs. Deshalb minimal-images und multi-stage builds.

---

## Schritt 3: CIS Compliance Check gegen den Cluster

Der Scan laeuft gegen die Kubernetes API — kein Node-Zugriff noetig,
aber Cluster-Admin-Rechte erforderlich.

```
trivy k8s --compliance=k8s-cis --report summary cluster
```

**Erwartete Ausgabe (Auszug):**
```
Summary Report for compliance: cis-k8s

┌──────────┬──────────────────────────────────────────────────────────┬────────────┐
│ ID       │ Title                                                    │ Status     │
├──────────┼──────────────────────────────────────────────────────────┼────────────┤
│ 5.2.1    │ Ensure that the cluster has at least one active policy   │ FAIL       │
│          │ control mechanism in place                               │            │
├──────────┼──────────────────────────────────────────────────────────┼────────────┤
│ 5.2.6    │ Minimize the admission of root containers                │ FAIL       │
├──────────┼──────────────────────────────────────────────────────────┼────────────┤
│ 5.2.7    │ Minimize the admission of containers with                │ FAIL       │
│          │ allowPrivilegeEscalation                                 │            │
└──────────┴──────────────────────────────────────────────────────────┴────────────┘
```

Detaillierter Report mit allen Findings:

```
trivy k8s --compliance=k8s-cis --report all cluster 2>/dev/null | less
```

---

## Schritt 4: Eigenen Namespace scannen

Fuer den eigenen Namespace koennen Teilnehmer ohne Cluster-Admin-Rechte scannen:

```
trivy k8s --namespace tln<nr> --report summary all
```

Was findet Trivy in eurem Namespace — und was hat sich veraendert nachdem ihr
die pods aus der securitycontext-exercise deployed habt?

```
# Gehaerteten Pod deployen
kubectl apply -f ~/manifests/securitycontext/05-pod-hardened.yml -n tln<nr>

# Scan erneut ausfuehren
trivy k8s --namespace tln<nr> --report summary all
```

---

## Vergleich: Trivy k8s vs. kube-bench

Beide Tools pruefen CIS-Compliance — aber an unterschiedlichen Stellen:

| | Trivy k8s | kube-bench |
|-|-----------|------------|
| Zugriff | Kubernetes API | Direkter Node-Zugriff (SSH / DaemonSet) |
| Prueft | Workloads, RBAC, Namespace-Konfiguration | Node-Dateiberechtigungen, Prozess-Flags, kubelet-Config |
| CIS-Abdeckung | Sektion 5 (Policies, Workloads) | Sektion 1 (Control Plane), Sektion 4 (Worker Nodes) |
| Einsatz | Von der Bastion / aus CI-CD | Auf dem Node selbst |
| Besonderheit | Findet unsichere laufende Pods | Findet unsichere Node-Konfiguration |

**Zusammen ergeben sie eine vollstaendige CIS-Pruefung:**
- kube-bench → Infrastruktur-Ebene (Nodes, API-Server-Flags)
- trivy k8s → Workload-Ebene (Pods, RBAC, Namespaces)

---

## Aufraeumen

Trivy hinterlaesst keine Ressourcen im Cluster.
Falls gehaertete Pods noch laufen:

```
kubectl delete pod pod-hardened -n tln<nr>
```

---

## Zusammenfassung

| Scan | CIS-Kapitel | Ergebnis |
|------|------------|---------|
| `trivy image nginx:1.27` | Keins (NIST / Supply Chain) | CVE-Liste mit Schweregrad |
| `trivy k8s --compliance=k8s-cis` | 1.x, 4.x, 5.x | Pass/Fail pro CIS-Check |
| `trivy k8s -n tln<nr>` | 5.2.x (Workloads) | Unsichere Pods im Namespace |

## Referenzen

  * https://aquasecurity.github.io/trivy/
  * https://aquasecurity.github.io/trivy/latest/docs/compliance/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.2)
