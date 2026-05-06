# Example Service 

## Warum Services ? 

  * Wenn in einem Deployment bei einem Wechsel des images neue Pods erstellt werden, erhalten diese eine neue IP-Adresse
  * Nachteil: Man müsste diese dann in allen Applikation ständig ändern, die auf die Pods zugreifen.
  * Lösung: Wir schalten einen Service davor !

## Hintergrund IP-Wechsel 
 
 <img width="930" height="134" alt="image" src="https://github.com/user-attachments/assets/26c16134-1f2a-4b42-8cca-355099d08604" />

 * Image-Version wurde jetzt in Deployment geändert, Ergebnis:

<img width="939" height="137" alt="image" src="https://github.com/user-attachments/assets/fb5a665b-98a7-445b-8ec7-27f12c2267e1" />


## Example 1: ClusterIP

### Schritt 1: Deployment 

```
cd
mkdir -p manifests
cd manifests 
mkdir 04-service 
cd 04-service 
```

```
nano 01-deploy.yml
```

```
# 01-deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 3
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

```
kubectl apply -f .
```

### Schritt 2:

```
nano 02-svc.yml
```

```
# 02-svc.yml 
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
```

```
kubectl apply -f . 
```

```
# wie ist die ClusterIP ?  
kubectl get all
kubectl get svc my-nginx
# Find endpoints / did svc find pods ?
kubectl describe svc my-nginx 
```

### Schritt 3: Deployment löschen 

```
kubectl delete -f 01-deploy.yml
# Keine endpunkte mehr 
kubectl describe svc my-nginx
```

 ### Schritt 4: Deployment wieder erstellen 

```
kubectl apply -f .
# Endpunkte wieder da
kubectl describe svc my-nginx
```

## Example II : Short version (NodePort)

```
# Wo sind wir ?
# cd; cd manifests/04-service 
```

```
nano 02-svc.yml
# in Zeile type: 
# ClusterIP ersetzt durch NodePort 

kubectl apply -f .
# NodePort ab 30.000 ausfindig machen
kubectl get svc
```

<img width="793" height="44" alt="image" src="https://github.com/user-attachments/assets/16bf90d4-7c3f-4c8f-9846-2ff5d0e63fcf" />

```
kubectl get nodes -o wide
```

<img width="926" height="157" alt="image" src="https://github.com/user-attachments/assets/eb396f36-cff1-4b6d-b136-e110fff1c807" />

```
# im client Externe NodeIP und NodePort verwenden 
curl http://164.92.193.245:32708
```

## Example III: Service mit LoadBalancer (ExternalIP)

```
cd; cd manifests/04-service 
nano 02-svc.yml
# in Zeile type: 
# NodePort ersetzt durch LoadBalancer  

kubectl apply -f .
kubectl get svc svc-nginx

kubectl describe svc my-nginx
# hier heisst das nicht External-IP ->
# sondern
```

<img width="775" height="63" alt="image" src="https://github.com/user-attachments/assets/3f1db219-e5d8-4bbf-a001-17fc5eaae93f" />

```
kubectl get svc my-nginx -w 
# spätestens nach 5 Minuten bekommen wir eine externe ip
# z.B. 41.32.44.45

curl http://41.32.44.45 
```



## Ref.

  * https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
