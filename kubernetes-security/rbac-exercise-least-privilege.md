# RBAC - Least Privilege fuer Entwickler-Namespace

## Hintergrund: Prinzip der minimalen Rechte

Ein ServiceAccount sollte **nur** die Rechte haben, die er tatsaechlich benoetigt.
Typische Fehler:

| Problem | CIS Referenz | Risiko |
|---------|--------------|--------|
| SA hat cluster-admin | 5.1.1 | Kompletter Cluster-Zugriff bei Kompromittierung |
| Token automatisch gemountet | 5.1.6 | Jeder Pod kann API nutzen |
| Wildcard-Verbs (`*`) | 5.1.3 | Unbeabsichtigte Rechte |

---

## Schritt 1: Vorbereitung

```
cd
mkdir -p manifests/rbac-leastpriv
cd manifests/rbac-leastpriv
```

```
kubectl create namespace rbac-leastpriv
```

---

## Schritt 2: ServiceAccount mit deaktiviertem Auto-Mount erstellen (CIS 5.1.6)

```
# vi 01-serviceaccount.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-readonly
  namespace: rbac-leastpriv
automountServiceAccountToken: false
```

```
kubectl apply -f 01-serviceaccount.yml -n rbac-leastpriv
```

Ohne `automountServiceAccountToken: false` wird in jeden Pod automatisch ein Token
gemountet — selbst wenn die App die Kubernetes API gar nicht braucht.

---

## Schritt 3: Role mit minimalen Rechten (nur Pods lesen)

```
# vi 02-role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: rbac-leastpriv
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

```
kubectl apply -f 02-role.yml -n rbac-leastpriv
```

**Bewusste Einschraenkung:**
- Kein `*` bei verbs (CIS 5.1.3)
- Nur `pods`, keine Secrets, ConfigMaps, Deployments
- Nur im eigenen Namespace (Role, nicht ClusterRole)

---

## Schritt 4: RoleBinding erstellen

```
# vi 03-rolebinding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-readonly-binding
  namespace: rbac-leastpriv
subjects:
- kind: ServiceAccount
  name: dev-readonly
  namespace: rbac-leastpriv
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f 03-rolebinding.yml -n rbac-leastpriv
```

---

## Schritt 5: Rechte testen mit kubectl auth can-i

```
# Was darf der ServiceAccount?
kubectl auth can-i get pods \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n rbac-leastpriv
```

**Erwartet: yes**

```
# Darf er Secrets lesen? (sollte nicht!)
kubectl auth can-i get secrets \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n rbac-leastpriv
```

**Erwartet: no**

```
# Darf er Pods in kube-system sehen? (sollte nicht!)
kubectl auth can-i list pods \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n kube-system
```

**Erwartet: no**

```
# Vollstaendige Rechteuebersicht
kubectl auth can-i --list \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n rbac-leastpriv
```

---

## Schritt 6: Pod mit korrektem ServiceAccount deployen

```
# vi 04-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: dev-app
spec:
  serviceAccountName: dev-readonly
  automountServiceAccountToken: false
  containers:
  - name: app
    image: nginx:alpine
    resources:
      requests:
        cpu: "50m"
        memory: "64Mi"
      limits:
        cpu: "100m"
        memory: "128Mi"
```

```
kubectl apply -f 04-pod.yml -n rbac-leastpriv
kubectl get pods -n rbac-leastpriv
```

Pruefen ob kein Token gemountet ist:

```
kubectl exec -n rbac-leastpriv dev-app -- ls /var/run/secrets/kubernetes.io/serviceaccount/ 2>/dev/null || echo "Kein Token gemountet - korrekt!"
```

---

## Schritt 7: Scan mit rbac-lookup zur Verifikation

```
rbac-lookup -k serviceaccount | grep rbac-leastpriv
```

**Erwartete Ausgabe:**
```
SUBJECT                          SCOPE             ROLE
rbac-leastpriv:dev-readonly      rbac-leastpriv    Role/pod-reader
```

Kein cluster-wide Scope, kein cluster-admin — der ServiceAccount ist
korrekt eingeschraenkt.

---

## Aufraumen

```
kubectl delete namespace rbac-leastpriv
```

---

## Zusammenfassung

| Was | Warum | CIS |
|-----|-------|-----|
| `automountServiceAccountToken: false` | Kein automatischer API-Zugang | 5.1.6 |
| Role statt ClusterRole | Scope auf Namespace begrenzen | 5.1.1 |
| Nur spezifische Verbs | Keine Wildcard-Rechte | 5.1.3 |
| Keine Secrets im Role | Geringste Rechte | 5.1.2 |

## Referenzen

  * https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  * https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1)
