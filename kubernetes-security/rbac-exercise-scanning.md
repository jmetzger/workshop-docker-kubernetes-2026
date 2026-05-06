# RBAC Scanning - Schwachstellen mit rbac-lookup finden

## Hintergrund: CIS Benchmark - RBAC Controls (Sektion 5.1)

| CIS ID | Kontrolle |
|--------|-----------|
| 5.1.1 | cluster-admin nur wo noetig |
| 5.1.3 | Keine Wildcards in Roles/ClusterRoles |
| 5.1.5 | default ServiceAccount nicht an aktive Rolle binden |
| 5.1.6 | SA-Tokens nicht automatisch mounten |
| 5.1.7 | system:masters Gruppe vermeiden |

`rbac-lookup` scannt den Cluster und zeigt, welche Subjekte (User, Gruppen,
ServiceAccounts) welche Rollen haben — ideal um CIS-Findings zu verifizieren.

---

## Schritt 1: rbac-lookup installieren

Das Tool wird im eigenen Home-Verzeichnis installiert — kein root noetig.
`~/bin` ist bereits im PATH aller Workshop-User.

```
mkdir -p ~/bin
curl -sL https://github.com/FairwindsOps/rbac-lookup/releases/download/v0.10.3/rbac-lookup_0.10.3_Linux_x86_64.tar.gz \
  | tar -xz -C ~/bin rbac-lookup
```

Version pruefen:

```
rbac-lookup version
```

Erwartete Ausgabe:
```
Version:0.10.3 Commit:46385c500f47267607a2cdde63feef59247615f5
```

---

## Schritt 2: Alle cluster-admin Bindings finden (CIS 5.1.1)

```
rbac-lookup -o wide system:masters
```

```
rbac-lookup -o wide cluster-admin
```

Alle Gruppen mit Rollen anzeigen — hier ist `system:masters` besonders kritisch:

```
rbac-lookup -k group
```

**Erwartete Findings:**
```
SUBJECT                  SCOPE          ROLE
system:masters           cluster-wide   ClusterRole/cluster-admin
kubeadm:cluster-admins   cluster-wide   ClusterRole/cluster-admin
```

CIS 5.1.7: Die Gruppe `system:masters` umgeht RBAC komplett und sollte
**niemals** fuer normale Nutzer verwendet werden.

---

## Schritt 3: Alle ServiceAccount-Bindungen scannen

```
rbac-lookup -k serviceaccount
```

Gezielt nach einem Namespace suchen:

```
rbac-lookup -k serviceaccount | grep kube-system | head -20
```

---

## Schritt 4: Wildcards in ClusterRoles finden (CIS 5.1.3)

```
kubectl get clusterroles -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data.get('items', []):
    name = item['metadata']['name']
    if name.startswith('system:'):
        continue
    for rule in item.get('rules', []):
        if '*' in rule.get('verbs', []) or '*' in rule.get('resources', []):
            print(f'WILDCARD in: {name}')
            print(f'  rule: {rule}')
"
```

---

## Schritt 5: Absichtlich eine unsichere Bindung erstellen

Jetzt demonstrieren wir das Problem: ein overprivileged ServiceAccount.

```
cd
mkdir -p manifests/rbac-demo
cd manifests/rbac-demo
```

```
# vi 01-bad-sa.yml
apiVersion: v1
kind: Namespace
metadata:
  name: rbac-demo
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: overprivileged-app
  namespace: rbac-demo
```

```
# vi 02-bad-binding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: overprivileged-app-admin
subjects:
- kind: ServiceAccount
  name: overprivileged-app
  namespace: rbac-demo
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f .
```

---

## Schritt 6: Scan wiederholen — Finding nachweisen

```
rbac-lookup -k serviceaccount | grep overprivileged
```

**Erwartete Ausgabe:**
```
SUBJECT                           SCOPE          ROLE
rbac-demo:overprivileged-app      cluster-wide   ClusterRole/cluster-admin
```

Das ist ein klarer CIS 5.1.1-Verstoss: ein normaler App-ServiceAccount
mit cluster-admin Rechten.

Testen — was kann dieser ServiceAccount alles?

```
kubectl auth can-i --list --as=system:serviceaccount:rbac-demo:overprivileged-app | head -10
```

**Erwartete Ausgabe: Fast alles erlaubt (`*`)**

---

## Schritt 7: Unsichere Bindung entfernen

```
kubectl delete clusterrolebinding overprivileged-app-admin
```

Verifizieren:

```
rbac-lookup -k serviceaccount | grep overprivileged
# Keine Ausgabe mehr
```

---

## Aufraumen

```
kubectl delete namespace rbac-demo
kubectl delete clusterrolebinding overprivileged-app-admin 2>/dev/null || true
```

---

## Zusammenfassung

| CIS Kontrolle | Wie pruefen |
|---------------|-------------|
| 5.1.1 cluster-admin | `rbac-lookup system:masters` |
| 5.1.3 Wildcards | `kubectl get clusterroles -o json` + Filter |
| 5.1.5 default SA | `rbac-lookup default` |
| 5.1.7 system:masters | `rbac-lookup -k group` |

## Referenzen

  * https://github.com/FairwindsOps/rbac-lookup
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1)
