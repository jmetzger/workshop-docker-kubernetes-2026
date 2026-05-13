# Debugging: FE zu Backend Verbindungen mit kubectl debug und NetworkPolicy

## Hintergrund

In produktiven Umgebungen laufen Container oft als minimale Images ohne Debug-Tools
(curl, wget, nc). Wenn eine Verbindung zwischen Frontend und Backend nicht funktioniert,
gibt es zwei haeufige Ursachen:

| Fehlerbild | Ursache | Diagnose |
|-----------|---------|----------|
| `Connection refused` | Service hat falsche Selector-Labels - keine Endpoints | `kubectl get endpoints` |
| `Connection timed out` | NetworkPolicy blockiert den Traffic | `kubectl describe networkpolicy` |

`kubectl debug` schleust einen ephemeral Container mit Debug-Tools in einen laufenden
Pod ein - ohne den Pod neu starten zu muessen.

## Schritt 1: Vorbereitung

```
cd
mkdir -p manifests
cd manifests
mkdir 15-debug-networkpolicy
cd 15-debug-networkpolicy
```

## Schritt 2: Backend Deployment und Service anlegen

Achtung: Im Service steckt ein Fehler - den sollt ihr selbst finden.

```
nano 01-backend.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      containers:
      - name: backend
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend-api
  ports:
  - port: 80
    targetPort: 80
```

```
kubectl apply -f 01-backend.yml -n debug-<dein-name>
```

## Schritt 3: Frontend Deployment anlegen (ohne Debug-Tools)

Das Frontend laeuft als minimales Python-Image - kein curl, kein wget, kein nc.

```
nano 02-frontend.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: python:3.12-slim
        command: ["python", "-c", "import time; time.sleep(86400)"]
```

```
kubectl apply -f 02-frontend.yml -n debug-<dein-name>
```

Warten bis beide Pods laufen:

```
kubectl get pods -n debug-<dein-name>
```

**Erwartete Ausgabe:**
```
NAME                        READY   STATUS    RESTARTS   AGE
backend-xxx                 1/1     Running   0          30s
frontend-xxx                1/1     Running   0          20s
```

---

## Problem 1: Connection refused - Service-Fehlkonfiguration

## Schritt 4: Verbindung testen - keine Tools im Frontend

```
FE_POD=$(kubectl get pod -n debug-<dein-name> -l app=frontend -o jsonpath='{.items[0].metadata.name}')
echo $FE_POD
```

Versuch direkt Tools zu nutzen:

```
kubectl exec -it $FE_POD -n debug-<dein-name> -- sh -c 'which curl; which wget; which nc'
```

**Erwartete Ausgabe:**
```
no curl
no wget
no nc
```

## Schritt 5: kubectl debug - Ephemeral Container hinzufuegen

```
kubectl debug -it $FE_POD -n debug-<dein-name> \
  --image=busybox:1.36 \
  --target=frontend \
  --profile=general \
  -- sh
```

Im Debug-Container testen:

```
nslookup backend-svc
```

**Erwartete Ausgabe (DNS funktioniert):**
```
Name:   backend-svc.debug-<dein-name>.svc.cluster.local
Address: 10.x.x.x
```

```
wget -qO- http://backend-svc --timeout=5
```

**Erwartete Ausgabe:**
```
wget: can't connect to remote host (10.x.x.x): Connection refused
```

DNS loest auf - aber TCP schlaegt sofort fehl (kein Timeout, sondern `Connection refused`).
Das ist kein NetworkPolicy-Problem. Der Service hat keine Endpoints.

Verlasse den Debug-Container:

```
exit
```

## Schritt 6: Endpoints pruefen

```
kubectl get endpoints backend-svc -n debug-<dein-name>
```

**Erwartete Ausgabe:**
```
NAME          ENDPOINTS   AGE
backend-svc   <none>      2m
```

Keine Endpoints! Der Service findet keine passenden Pods. Vergleiche den Service-Selector
mit den Pod-Labels:

```
kubectl get service backend-svc -n debug-<dein-name> -o jsonpath='{.spec.selector}'
kubectl get pods -n debug-<dein-name> -l app=backend --show-labels
```

**Ausgabe:**
```
{"app":"backend-api"}
NAME          LABELS
backend-xxx   app=backend,tier=backend,...
```

**Diagnose:** Service sucht `app=backend-api`, Pods haben `app=backend`.

## Schritt 7: Fix - Service-Selector korrigieren

```
kubectl patch service backend-svc -n debug-<dein-name> \
  -p '{"spec":{"selector":{"app":"backend"}}}'
```

```
kubectl get endpoints backend-svc -n debug-<dein-name>
```

**Erwartete Ausgabe (jetzt mit Endpoint):**
```
NAME          ENDPOINTS         AGE
backend-svc   10.x.x.x:80      3m
```

Erneut testen mit kubectl debug:

```
kubectl debug -it $FE_POD -n debug-<dein-name> \
  --image=busybox:1.36 \
  --target=frontend \
  --profile=general \
  -- sh
```

```
wget -qO- http://backend-svc --timeout=5
```

**Erwartete Ausgabe (Verbindung OK):**
```
<h1>Welcome to nginx!</h1>
```

```
exit
```

---

## Problem 2: Connection timed out - NetworkPolicy

## Schritt 8: NetworkPolicy anwenden

Die NetworkPolicy erlaubt Zugriff auf das Backend nur von Pods mit dem Label
`role: api-consumer`.

```
nano 03-networkpolicy.yml
```

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
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
          role: api-consumer
    ports:
    - protocol: TCP
      port: 80
```

```
kubectl apply -f 03-networkpolicy.yml -n debug-<dein-name>
```

## Schritt 9: Verbindung testen - anderer Fehler

```
kubectl debug -it $FE_POD -n debug-<dein-name> \
  --image=busybox:1.36 \
  --target=frontend \
  --profile=general \
  -- sh
```

```
wget -qO- http://backend-svc --timeout=5
```

**Erwartete Ausgabe:**
```
wget: download timed out
```

Diesmal ein **Timeout** - nicht `Connection refused`. Der Service hat Endpoints,
aber die NetworkPolicy blockiert den Traffic.

```
exit
```

## Schritt 10: NetworkPolicy und Labels pruefen

```
kubectl describe networkpolicy backend-policy -n debug-<dein-name>
```

**Ausgabe:**
```
Spec:
  PodSelector:     app=backend
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      PodSelector: role=api-consumer
```

```
kubectl get pods -n debug-<dein-name> --show-labels
```

**Ausgabe:**
```
NAME           LABELS
backend-xxx    app=backend,tier=backend,...
frontend-xxx   app=frontend,tier=frontend,...
```

**Diagnose:** Frontend-Pod hat kein Label `role=api-consumer`.

## Schritt 11: Fix - Label zum Frontend Deployment hinzufuegen

```
kubectl patch deployment frontend -n debug-<dein-name> \
  -p '{"spec":{"template":{"metadata":{"labels":{"role":"api-consumer"}}}}}'
```

Warten bis der neue Pod laeuft:

```
kubectl get pods -n debug-<dein-name> -l role=api-consumer
```

**Erwartete Ausgabe:**
```
NAME           READY   STATUS    RESTARTS   AGE
frontend-yyy   1/1     Running   0          10s
```

## Schritt 12: Finale Verbindung pruefen

```
FE_POD=$(kubectl get pod -n debug-<dein-name> -l app=frontend,role=api-consumer \
  -o jsonpath='{.items[0].metadata.name}')

kubectl debug -it $FE_POD -n debug-<dein-name> \
  --image=busybox:1.36 \
  --target=frontend \
  --profile=general \
  -- sh
```

```
wget -qO- http://backend-svc --timeout=5
```

**Erwartete Ausgabe:**
```
<h1>Welcome to nginx!</h1>
```

Verbindung funktioniert. Verlasse den Debug-Container:

```
exit
```

## Aufraeumen

```
kubectl delete namespace debug-<dein-name>
```

## Zusammenfassung

| Problem | Fehlermeldung | Diagnose-Befehl | Fix |
|---------|--------------|-----------------|-----|
| Falscher Service-Selector | `Connection refused` | `kubectl get endpoints` | Selector anpassen |
| NetworkPolicy blockiert | `download timed out` | `kubectl describe networkpolicy` | Label am Pod-Template erganzen |
