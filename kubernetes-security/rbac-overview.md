# Kubernetes RBAC - Grundlagen

## Was ist RBAC?

**Role-Based Access Control (RBAC)** steuert, wer was im Kubernetes-Cluster darf.
Seit Kubernetes 1.8 ist RBAC standardmaessig aktiviert und der einzige empfohlene
Autorisierungsmechanismus.

Jede Anfrage an den API-Server durchlaeuft drei Stufen:

```
Anfrage --> Authentifizierung --> Autorisierung (RBAC) --> Admission Control --> etcd
```

---

## Die 4 RBAC-Objekte

| Objekt | Scope | Beschreibung |
|--------|-------|--------------|
| `Role` | Namespace | Definiert Rechte innerhalb eines Namespaces |
| `ClusterRole` | Cluster-weit | Definiert Rechte cluster-weit (oder als Vorlage fuer Namespaces) |
| `RoleBinding` | Namespace | Verknuepft eine Role oder ClusterRole mit einem Subjekt im Namespace |
| `ClusterRoleBinding` | Cluster-weit | Verknuepft eine ClusterRole cluster-weit mit einem Subjekt |

**Faustregel:** Roles definieren *was* erlaubt ist. Bindings legen fest *wer* es darf.

---

## Subjekte: Wer kann Rechte bekommen?

| Typ | Beispiel | Beschreibung |
|-----|---------|--------------|
| `User` | `alice` | Kubernetes-User (extern verwaltet, z.B. via Zertifikate) |
| `Group` | `system:masters` | Gruppe von Usern |
| `ServiceAccount` | `default` | In-Cluster Identitaet fuer Pods und Workloads |

Kubernetes verwaltet **keine** eigene User-Datenbank.  
User werden ueber externe Mechanismen identifiziert (Zertifikate, OIDC, Tokens).  
ServiceAccounts dagegen sind echte Kubernetes-Objekte und leben in einem Namespace.

---

## Aufbau einer Role

Eine Role besteht aus **Rules** — jede Rule hat drei Felder:

```
# vi role-example.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: mein-namespace
rules:
- apiGroups: [""]        # "" = core API group (pods, services, secrets, ...)
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### apiGroups

| apiGroup | Ressourcen |
|----------|-----------|
| `""` (leer) | pods, services, secrets, configmaps, namespaces |
| `apps` | deployments, replicasets, daemonsets, statefulsets |
| `batch` | jobs, cronjobs |
| `rbac.authorization.k8s.io` | roles, rolebindings, clusterroles |
| `networking.k8s.io` | ingresses, networkpolicies |

### Verbs

| Verb | Entspricht |
|------|-----------|
| `get` | Einzelne Ressource lesen |
| `list` | Alle Ressourcen auflisten |
| `watch` | Aenderungen beobachten |
| `create` | Neue Ressource erstellen |
| `update` | Ressource aendern |
| `patch` | Ressource teilweise aendern |
| `delete` | Ressource loeschen |
| `*` | Alles erlaubt (CIS-Warnung: vermeiden!) |

---

## RoleBinding: Role + Subjekt verbinden

```
# vi rolebinding-example.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-pod-reader
  namespace: mein-namespace
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Scope: Namespace vs. Cluster

```
Role + RoleBinding
  --> Zugriff NUR im angegebenen Namespace

ClusterRole + ClusterRoleBinding
  --> Zugriff auf alle Namespaces + nicht-namespaced Ressourcen (Nodes, PVs)

ClusterRole + RoleBinding  (Kombination moeglich!)
  --> ClusterRole als Vorlage, Zugriff nur im Namespace des RoleBindings
```

### Nicht-namespaced Ressourcen (benoetigen ClusterRole)

```
kubectl api-resources --namespaced=false
```

Beispiele: `nodes`, `persistentvolumes`, `clusterroles`, `namespaces`

---

## Berechtigungen pruefen

Eigene Rechte pruefen:

```
kubectl auth can-i get pods -n mein-namespace
```

Rechte eines anderen Subjekts pruefen (`--as`):

```
kubectl auth can-i get secrets \
  --as=system:serviceaccount:mein-namespace:mein-sa \
  -n mein-namespace
```

Vollstaendige Uebersicht:

```
kubectl auth can-i --list \
  --as=system:serviceaccount:mein-namespace:mein-sa \
  -n mein-namespace
```

---

## Beispiel: ServiceAccount fuer eine App

```
                +------------------+
                |  ServiceAccount  |   (Identitaet des Pods)
                |  app-reader      |
                +------------------+
                         |
                         | RoleBinding
                         |
                +------------------+
                |      Role        |   (Was ist erlaubt?)
                |  pod-reader      |
                |  verbs: get,list |
                |  resources: pods |
                +------------------+
```

Der Pod laeuft mit dem ServiceAccount `app-reader` und darf nur Pods lesen —
kein Zugriff auf Secrets, keine Schreibrechte, kein anderer Namespace.

---

## Haeufige Fallen

| Problem | Risiko | CIS |
|---------|--------|-----|
| `cluster-admin` fuer normale Apps | Kompletter Cluster-Zugriff | 5.1.1 |
| Wildcard-Verbs (`*`) | Unbeabsichtigte Rechte | 5.1.3 |
| `default` SA mit Rechten | Jeder Pod im Namespace hat Zugriff | 5.1.5 |
| `automountServiceAccountToken: true` | Token in Pod auch wenn nicht benoetigt | 5.1.6 |
| `system:masters` Gruppe | Umgeht RBAC vollstaendig | 5.1.7 |

---

## Referenzen

  * https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1)
  * https://github.com/FairwindsOps/rbac-lookup
