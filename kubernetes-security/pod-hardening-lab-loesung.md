# Lab: Pod Hardening - Loesungen

## Loesung Aufgabe 1: `01-pods-hardened.yml`

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

## Loesung Aufgabe 2: `02-networkpolicies.yml`

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

## Loesung Aufgabe 3: `03-rbac.yml`

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

## Vollstaendige getestete Loesung (mit korrekter Reihenfolge)

> Getestet auf kubeadm-Cluster v1.35.4 mit Calico CNI.
>
> **Wichtige Korrekturen gegenueber dem urspruenglichen Loesungsvorschlag:**
> - `kennethreitz/httpbin` laeuft als root und kann Port 80 nicht mit UID 1000 binden
>   → ersetzt durch `mccutchen/go-httpbin` (Port 8080, non-root-tauglich)
> - Service: port 80 → targetPort **8080** (curl http://backend/get funktioniert weiterhin)
> - NetworkPolicy-Ports: **8080** (Pod-Port nach DNAT, nicht Service-Port)
> - **Reihenfolge**: RBAC zuerst, dann Pods (frontend-sa muss vor dem Pod existieren)

### Schritt 0: Namespace und Verzeichnis vorbereiten

```
export NS=tln<nr>
kubectl config set-context --current --namespace=$NS
```

```
cd
mkdir -p manifests/pod-hardening
cd manifests/pod-hardening
```

### Schritt 1: RBAC anlegen (muss vor den Pods existieren!)

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

### Schritt 2: Gehaertete Pods anlegen

> Backend-Image: `mccutchen/go-httpbin` statt `kennethreitz/httpbin`
> (non-root, Port 8080, read-only Filesystem moeglich)
> Service mappt Port 80 → targetPort 8080, daher bleibt `curl http://backend/get` unveraendert.

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
    image: mccutchen/go-httpbin
    ports:
    - containerPort: 8080
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
    targetPort: 8080
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
kubectl wait pod frontend backend --for=condition=Ready --timeout=90s
kubectl get pod
```

### Schritt 3: NetworkPolicy anlegen

> Achtung: NetworkPolicies werden gegen den **Pod-Port** ausgewertet (nach DNAT).
> Der Service mappt 80→8080 — die Policies muessen Port **8080** erlauben.

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
      port: 8080
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
      port: 8080
```

```
kubectl apply -f 02-networkpolicies.yml
```

### Schritt 4: Default-ServiceAccount patchen

```
kubectl patch serviceaccount default -p '{"automountServiceAccountToken": false}'
```

### Schritt 5: Validierung

```
# 1. Beide Pods laufen
kubectl get pod
```

```
# 2. Frontend erreicht Backend (Service 80 -> Pod 8080)
kubectl exec frontend -- curl -s http://backend/get | head -5
```

Erwartete Ausgabe: JSON mit `"args": {}` von go-httpbin.

```
# 3. Externer Traffic geblockt
kubectl exec frontend -- curl -s --max-time 5 http://example.com
```

Erwartete Ausgabe: `curl: (28) Connection timed out`

```
# 4. Backend kann Frontend nicht erreichen
#    Hinweis: go-httpbin hat kein curl/wget im Image (minimales Go-Binary).
#    Test mit temporaerem Pod:
kubectl run nettest --image=busybox --restart=Never --rm -it -- \
  wget -qO- --timeout=5 http://frontend 2>&1 || echo "BLOCKED"
```

Erwartete Ausgabe: `wget: bad address 'frontend'` (DNS geblockt durch default-deny) + BLOCKED

```
# 5. RBAC: frontend-sa darf Pods lesen
kubectl auth can-i get pods --as=system:serviceaccount:$NS:frontend-sa
```

Erwartete Ausgabe: `yes`

```
# 6. RBAC: frontend-sa darf keine Pods loeschen
kubectl auth can-i delete pods --as=system:serviceaccount:$NS:frontend-sa
```

Erwartete Ausgabe: `no`

### Schritt 6: Aufraeumen

```
kubectl delete -f 03-rbac.yml -f 02-networkpolicies.yml \
               -f 01-pods-hardened.yml -f 00-baseline.yml \
               --ignore-not-found
kubectl patch serviceaccount default -p '{"automountServiceAccountToken": true}'
```
