# RBAC - Nutzer anlegen und als zweites kubectl-Profil einrichten

## Hintergrund

In der Praxis greifen verschiedene Personen oder Systeme auf denselben Cluster zu —
jeder mit eigenen Rechten. Dieses Beispiel zeigt den vollstaendigen Weg:

1. ServiceAccount + Token anlegen
2. Rolle und Binding definieren (Least Privilege)
3. Token als zweites Profil in kubeconfig eintragen
4. Context wechseln und Rechte verifizieren
5. Zurueck zum Admin-Context

> Ab Kubernetes 1.25 werden Tokens **nicht** mehr automatisch beim Anlegen eines
> ServiceAccounts erstellt. Das Secret mit dem Token muss manuell mit einer
> Annotation angelegt werden.

---

## Schritt 1: Vorbereitung

```
cd
mkdir -p manifests/rbac-user
cd manifests/rbac-user
```

---

## Schritt 2: ServiceAccount und Token-Secret anlegen

```
# vi 01-serviceaccount.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: training<nr>
  namespace: default
```

```
# vi 02-secret.yml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: trainingtoken<nr>
  namespace: default
  annotations:
    kubernetes.io/service-account.name: training<nr>
```

```
kubectl apply -f .
```

Token wurde erzeugt?

```
kubectl get secret trainingtoken<nr> -o jsonpath='{.data.token}' | base64 --decode | head -c 50
```

Wenn eine JWT-Zeichenkette erscheint: Token ist da.

---

## Schritt 3: ClusterRole mit minimalen Rechten definieren

```
# vi 03-clusterrole.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-clusterrole<nr>
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list", "create"]
```

```
kubectl apply -f 03-clusterrole.yml
```

---

## Schritt 4: ClusterRole per RoleBinding dem ServiceAccount zuweisen

RoleBinding statt ClusterRoleBinding — Zugriff nur im `default`-Namespace:

```
# vi 04-rolebinding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-default-pods<nr>
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pods-clusterrole<nr>
subjects:
- kind: ServiceAccount
  name: training<nr>
  namespace: default
```

```
kubectl apply -f 04-rolebinding.yml
```

---

## Schritt 5: Rechte pruefen (ohne Context-Wechsel)

```
kubectl auth can-i get pods -n default \
  --as system:serviceaccount:default:training<nr>
```

```
yes
```

```
kubectl auth can-i get deployments -n default \
  --as system:serviceaccount:default:training<nr>
```

```
no
```

Pods: erlaubt. Deployments: verboten. Least Privilege funktioniert.

---

## Schritt 6: Token auslesen und als zweites kubectl-Profil eintragen

Token in Variable laden:

```
TOKEN=$(kubectl get secret trainingtoken<nr> \
  -o jsonpath='{.data.token}' | base64 --decode)
echo $TOKEN
```

Neuen Context und Credentials in die kubeconfig eintragen.
Der Cluster-Name in einem kubeadm-Cluster lautet `kubernetes`:

```
kubectl config set-context training-ctx \
  --cluster kubernetes \
  --user training<nr>

kubectl config set-credentials training<nr> --token=$TOKEN
```

Beide Contexts anzeigen:

```
kubectl config get-contexts
```

```
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
          training-ctx                  kubernetes   training<nr>
```

---

## Schritt 7: Context wechseln und Rechte testen

```
kubectl config use-context training-ctx
```

Erlaubte Aktion — Pods auflisten:

```
kubectl get pods
```

Verbotene Aktion — Deployments auflisten:

```
kubectl get deployments
```

**Erwarteter Fehler:**
```
Error from server (Forbidden): deployments.apps is forbidden:
User "system:serviceaccount:default:training<nr>" cannot list resource
"deployments" in API group "apps" in the namespace "default"
```

---

## Schritt 8: Zurueck zum Admin-Context

```
kubectl config use-context kubernetes-admin@kubernetes
```

Verifizieren:

```
kubectl config get-contexts
```

```
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
          training-ctx                  kubernetes   training<nr>
```

---

## Aufraeumen

```
kubectl config delete-context training-ctx
kubectl config unset users.training<nr>
```

```
kubectl delete -f .
```

---

## Zusammenfassung

| Was | Wie |
|-----|-----|
| SA + Token anlegen | ServiceAccount + Secret mit Annotation |
| Rechte vergeben | ClusterRole + RoleBinding (namespace-scoped) |
| Rechte testen | `kubectl auth can-i ... --as system:serviceaccount:...` |
| Token auslesen | `kubectl get secret ... -o jsonpath='{.data.token}' | base64 --decode` |
| Zweites Profil | `kubectl config set-context` + `set-credentials` |
| Context wechseln | `kubectl config use-context <name>` |
| Zurueck zu Admin | `kubectl config use-context kubernetes-admin@kubernetes` |
