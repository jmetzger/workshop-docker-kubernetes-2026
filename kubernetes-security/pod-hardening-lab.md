# Lab: Pod Hardening & Kubernetes Security

**Schwierigkeit:** Mittel
**Dauer:** ca. 90-120 Minuten
**Voraussetzungen:** SecurityContext-Uebung, NetworkPolicy-Uebung, RBAC-Uebung absolviert

---

## Lernziele

Nach dieser Uebung kannst du:

- Pods mit sicherem `securityContext` ausstatten (non-root, read-only, Capabilities, Seccomp)
- Pod-zu-Pod-Traffic mit `NetworkPolicy` einschraenken
- Einen dedizierten `ServiceAccount` mit minimalen RBAC-Rechten betreiben

---

## Szenario

Ein Entwicklerteam hat eine kleine Web-App deployed: ein `frontend`-Pod (nginx) erreicht
ueber einen Service einen `backend`-Pod (httpbin). Beides laeuft als root, ohne
NetworkPolicy, mit dem Default-ServiceAccount.

Du sollst das Setup haerten.

---

## Vorbereitung

```
export NS=tln<nr>
kubectl config set-context --current --namespace=$NS
```

```
cd
mkdir -p manifests/pod-hardening
cd manifests/pod-hardening
```

---

## Ausgangslage (unsicher)

```
# vi 00-baseline.yml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
  - name: app
    image: kennethreitz/httpbin
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
  - name: web
    image: nginx:1.27
    ports:
    - containerPort: 80
```

```
kubectl apply -f 00-baseline.yml
kubectl get pod,svc
```

Warten bis beide Pods laufen:

```
kubectl wait pod frontend backend --for=condition=Ready --timeout=90s
```

Verifizieren, dass Frontend Backend erreicht:

```
kubectl exec frontend -- curl -s http://backend/get | head -20
```

---

## Aufgabe 1: SecurityContext haerten

Beide Pods sollen:

- als **non-root** laufen (UID 1000)
- ein **read-only** Root-Filesystem haben
- **alle Capabilities** droppen
- **Privilege-Escalation** unterbinden
- das `RuntimeDefault` seccomp-Profil nutzen

> Hinweis: Fuer nginx verwende `nginxinc/nginx-unprivileged:1.27` (Port 8080).
> Fuer httpbin braucht `/tmp` ein `emptyDir`-Volume.

---

## Aufgabe 2: NetworkPolicy

- **Default-Deny** fuer allen Ingress- und Egress-Traffic im Namespace
- Frontend darf **DNS** (kube-system, UDP/TCP 53) nutzen
- Frontend darf das Backend auf **Port 80** erreichen
- Backend darf **DNS** nutzen, nimmt sonst keinen Egress an
- Backend nimmt **nur Ingress vom Frontend** an

---

## Aufgabe 3: RBAC & ServiceAccount

- Der `default`-ServiceAccount soll **kein** Token mehr automounten
- Lege einen `ServiceAccount` `frontend-sa` an (kein Token-Automount)
- Lege eine `Role` `pod-reader` an: nur `get`/`list` auf `pods`
- Binde die Rolle an `frontend-sa`
- Frontend-Pod nutzt `frontend-sa`

---

## Aufgabe 4: Validierung

Pruefe nach dem Hardening:

```
# 1. Beide Pods laufen
kubectl get pod

# 2. Frontend erreicht Backend weiterhin
kubectl exec frontend -- curl -s http://backend/get | head -5

# 3. Externer Traffic geblockt
kubectl exec frontend -- curl -s --max-time 5 http://example.com

# 4. Backend kann Frontend nicht erreichen
kubectl exec backend -- curl -s --max-time 5 http://frontend

# 5. RBAC: frontend-sa darf Pods lesen
kubectl auth can-i get pods --as=system:serviceaccount:$NS:frontend-sa

# 6. RBAC: frontend-sa darf keine Pods loeschen
kubectl auth can-i delete pods --as=system:serviceaccount:$NS:frontend-sa
```

**Erwartete Ergebnisse:**
- Aufgabe 2: Timeout (Egress geblockt)
- Aufgabe 3: Timeout (kein Ingress zum Frontend)
- Aufgabe 5: `yes`
- Aufgabe 6: `no`

---

## Loesung

### Loesung Aufgabe 1: `01-pods-hardened.yml`

```
# vi 01-pods-hardened.yml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  automountServiceAccountToken: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: kennethreitz/httpbin
    ports:
    - containerPort: 80
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  serviceAccountName: frontend-sa
  automountServiceAccountToken: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: web
    image: nginxinc/nginx-unprivileged:1.27
    ports:
    - containerPort: 8080
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
  - name: tmp
    emptyDir: {}
```

```
kubectl apply -f 01-pods-hardened.yml
kubectl get pod
```

### Loesung Aufgabe 2: `02-networkpolicies.yml`

```
# vi 02-networkpolicies.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-egress-to-backend
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-ingress-from-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

```
kubectl apply -f 02-networkpolicies.yml
```

### Loesung Aufgabe 3: `03-rbac.yml`

```
# vi 03-rbac.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: frontend-sa
automountServiceAccountToken: false
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: frontend-sa-pod-reader
subjects:
- kind: ServiceAccount
  name: frontend-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f 03-rbac.yml
```

Default-ServiceAccount patchen:

```
kubectl patch serviceaccount default -p '{"automountServiceAccountToken": false}'
```

---

## Aufraeumen

```
kubectl delete -f 03-rbac.yml -f 02-networkpolicies.yml \
               -f 01-pods-hardened.yml -f 00-baseline.yml \
               --ignore-not-found
kubectl patch serviceaccount default -p '{"automountServiceAccountToken": true}'
```

---

## Referenzen

  * https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  * https://kubernetes.io/docs/concepts/services-networking/network-policies/
  * https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1, 5.2, 5.3)
