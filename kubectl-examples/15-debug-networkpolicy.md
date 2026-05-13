# Debugging: FE zu Backend Verbindungen mit kubectl debug und NetworkPolicy

## Hintergrund

In produktiven Umgebungen laufen Container oft als minimale Images ohne Debug-Tools
(curl, wget, nc, nslookup). Wenn eine Verbindung zwischen Frontend und Backend
nicht funktioniert, helfen zwei Strategien:

| Strategie | Beschreibung |
|-----------|-------------|
| `kubectl debug` | Fuegt einen ephemeral Container mit Debug-Tools in den laufenden Pod ein |
| NetworkPolicy pruefen | Zeigt welche Pods miteinander kommunizieren duerfen |

Ein haeufiger Fehler: Das falsche Label am Source-Pod - die NetworkPolicy
erlaubt den Zugriff nur fuer Pods mit einem bestimmten Label.

## Schritt 1: Vorbereitung

```
cd
mkdir -p manifests
cd manifests
mkdir 15-debug-networkpolicy
cd 15-debug-networkpolicy
```

## Schritt 2: Backend Deployment und Service anlegen

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
    app: backend
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

## Schritt 4: NetworkPolicy anwenden

Die NetworkPolicy erlaubt Zugriff auf das Backend **nur** von Pods mit dem Label
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

## Schritt 5: Problem entdecken - keine Debug-Tools im Frontend

Versuche in den Frontend-Pod zu verbinden und curl/wget zu nutzen:

```
FE_POD=$(kubectl get pod -n debug-<dein-name> -l app=frontend -o jsonpath='{.items[0].metadata.name}')
echo $FE_POD
```

```
kubectl exec -it $FE_POD -n debug-<dein-name> -- sh
```

Im Pod - pruefen welche Tools vorhanden sind:

```
which curl
which wget
which nc
```

**Erwartete Ausgabe:**
```
no curl
no wget
no nc
```

Direkt ausfuehren schlaegt fehl:

```
curl http://backend-svc
```

**Erwarteter Fehler:**
```
sh: curl: not found
```

Verlasse den Pod:

```
exit
```

## Schritt 6: kubectl debug - Ephemeral Container mit Tools hinzufuegen

`kubectl debug` schleust einen Debug-Container in den laufenden Pod ein.
Der Debug-Container teilt das Netzwerk des Frontend-Pods - und damit
auch dessen NetworkPolicy-Regeln.

```
kubectl debug -it $FE_POD -n debug-<dein-name> \
  --image=busybox:1.36 \
  --target=frontend \
  --profile=general \
  -- sh
```

Im Debug-Container die DNS-Aufloessung und Verbindung testen:

```
nslookup backend-svc
```

**Erwartete Ausgabe (DNS funktioniert):**
```
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   backend-svc.debug-<dein-name>.svc.cluster.local
Address: 10.x.x.x
```

```
wget -qO- http://backend-svc --timeout=5
```

**Erwartete Ausgabe (Verbindung schlaegt fehl):**
```
wget: download timed out
```

Die DNS-Aufloessung funktioniert - der Service ist erreichbar.
Aber die TCP-Verbindung wird blockiert. Das ist ein NetworkPolicy-Problem.

Verlasse den Debug-Container:

```
exit
```

## Schritt 7: NetworkPolicy untersuchen

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

Jetzt die Labels des Frontend-Pods pruefen:

```
kubectl get pods -n debug-<dein-name> --show-labels
```

**Ausgabe:**
```
NAME           READY   LABELS
backend-xxx    1/1     app=backend,tier=backend,...
frontend-xxx   1/1     app=frontend,tier=frontend,...
```

**Diagnose:** Das Frontend-Pod hat NICHT das Label `role: api-consumer`.
Die NetworkPolicy blockiert deshalb den Zugriff.

## Schritt 8: Fix - Label zum Frontend Deployment hinzufuegen

Das Label muss im Pod-Template des Deployments gesetzt werden:

```
kubectl patch deployment frontend -n debug-<dein-name> \
  -p '{"spec":{"template":{"metadata":{"labels":{"role":"api-consumer"}}}}}'
```

Warten bis der neue Pod laeuft:

```
kubectl get pods -n debug-<dein-name> --show-labels
```

**Erwartete Ausgabe:**
```
NAME           READY   LABELS
frontend-yyy   1/1     app=frontend,role=api-consumer,tier=frontend,...
```

## Schritt 9: Verbindung erneut testen

Neuen Frontend-Pod-Namen holen und Debug-Container starten:

```
FE_POD=$(kubectl get pod -n debug-<dein-name> -l app=frontend,role=api-consumer \
  -o jsonpath='{.items[0].metadata.name}')

kubectl debug -it $FE_POD -n debug-<dein-name> \
  --image=busybox:1.36 \
  --target=frontend \
  --profile=general \
  -- sh
```

Im Debug-Container:

```
wget -qO- http://backend-svc --timeout=5
```

**Erwartete Ausgabe (Verbindung erfolgreich):**
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

Verbindung funktioniert! Verlasse den Debug-Container:

```
exit
```

## Aufraeumen

```
kubectl delete namespace debug-<dein-name>
```

## Zusammenfassung

| Schritt | Ergebnis |
|---------|----------|
| exec ohne Tools | Kein curl/wget/nc im Container |
| kubectl debug | Ephemeral Container mit busybox eingefuegt |
| wget ohne Label | Timeout - NetworkPolicy blockiert |
| kubectl describe netpol | Nur `role=api-consumer` erlaubt |
| Label hinzugefuegt | Neuer Pod mit korrektem Label |
| wget mit Label | Verbindung erfolgreich |
