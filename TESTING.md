# Pruefprotokoll - Workshop Uebungen

**Cluster:** 161.35.210.204 (Bastion/Control Plane)
**CNI:** Calico
**Kubernetes:** v1.32.x
**Getestet als:** tln1 (unprivilegierter User)
**Zuletzt geprueft:** 2026-05-06

---

## kubernetes-security/

| Datei | Getestet | Ergebnis | Anmerkungen |
|-------|----------|----------|-------------|
| `cis-benchmark-overview.md` | - | - | Nur Theorie/Links |
| `cis-benchmark-exercise-control-plane.md` | - | - | Morgen Tag 4 |
| `cis-benchmark-exercise.md` | - | - | Morgen Tag 4 |
| `hardening-control-plane-profiling.md` | - | - | Nur Theorie |
| `cluster-access.md` | - | - | Nur Infos |
| `audit-logging-exercise.md` | - | - | Morgen Tag 4 |
| `rbac-overview.md` | - | - | Nur Theorie |
| `rbac-exercise-scanning.md` | ja | PASS | rbac-lookup v0.10.3 installiert als tln1 in ~/bin |
| `rbac-exercise-least-privilege.md` | - | - | Morgen Tag 4 |
| `securitycontext-exercise.md` | ja | PASS | Alle 5 Schritte verifiziert (siehe unten) |
| `networkpolicy-exercise.md` | ja | PASS | Alle 3 Szenarien verifiziert (siehe unten) |
| `pod-security-admission.md` | - | - | Morgen Tag 4 |
| `pod-hardening-lab.md` | - | - | Freitag Abschluss-Lab |

---

## Detailergebnisse

### rbac-exercise-scanning.md

```
# Installation als unprivilegierter User (tln1)
mkdir -p ~/bin
curl -sL https://github.com/FairwindsOps/rbac-lookup/releases/download/v0.10.3/rbac-lookup_0.10.3_Linux_x86_64.tar.gz \
  | tar -xz -C ~/bin rbac-lookup

rbac-lookup version
# Version:0.10.3 Commit:46385c500f47267607a2cdde63feef59247615f5  PASS

~/bin ist im PATH von tln1 vorhanden  PASS
```

---

### securitycontext-exercise.md

Getestet in Namespace `sc-test` als tln1:

| Schritt | Test | Ergebnis |
|---------|------|----------|
| Schritt 1 | `kubectl exec pod-root -- id` | `uid=0(root)` PASS |
| Schritt 2 | `kubectl exec pod-nonroot -- id` | `uid=1000` PASS |
| Schritt 3 | `touch /test-file` in pod-readonly | `Read-only file system` PASS |
| Schritt 3 | `touch /tmp/test-file` in pod-readonly | Schreiben erlaubt PASS |
| Schritt 5 | `cat /proc/1/status \| grep Cap` | Alle Werte `0000000000000000` PASS |
| Schritt 5 | `curl -s http://localhost:8080` | nginx-Antwort PASS |
| Image | `nginxinc/nginx-unprivileged:1.27` | Startet als UID 1000, Port 8080 PASS |

---

### networkpolicy-exercise.md

Getestet in Namespace `np-test` als tln1:

| Szenario | Test | Ergebnis |
|----------|------|----------|
| Baseline | Frontend curl Backend | `<title>Welcome to nginx!</title>` PASS |
| Default-Deny | Frontend curl Backend | Timeout (exit 28) PASS |
| DNS + Freigabe | Frontend curl Backend | `<title>Welcome to nginx!</title>` PASS |
| Isolation | Backend curl Frontend | Timeout (exit 6) PASS |
| Isolation | Frontend curl example.com | Timeout (exit 28) PASS |

---

## Noch zu testen (Tag 4)

- [ ] `cis-benchmark-exercise-control-plane.md` — kube-bench auf Control Plane
- [ ] `cis-benchmark-exercise.md` — kube-bench auf Worker Node
- [ ] `audit-logging-exercise.md` — Audit Policy + Logfile
- [ ] `rbac-exercise-least-privilege.md` — SA + Role + RoleBinding
- [ ] `pod-security-admission.md` — PSA Namespace-Label
- [ ] `pod-hardening-lab.md` — Abschluss-Lab komplett

---

## Umgebung

```
# Bastion
ssh -i ~/.ssh/id_ed25519_nopass root@161.35.210.204

# Als tln1 testen
su - tln1

# kubectl funktioniert ohne KUBECONFIG
kubectl get nodes
```

### Cluster-Nodes

```
kubectl get nodes
# NAME        STATUS   ROLES           AGE   VERSION
# k8s-cp      Ready    control-plane   ...   v1.32.x
# k8s-worker  Ready    <none>          ...   v1.32.x
```
