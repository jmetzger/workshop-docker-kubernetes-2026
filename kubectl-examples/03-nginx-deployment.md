# Example Deployment nginx 

```
cd
cd manifests
mkdir 03-deploy
cd 03-deploy 
nano deploy.yml 
```

```

# vi deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 8 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        
```

```
kubectl apply -f deploy.yml 
```

## Explore 

```
kubectl get all
```

## Optional: Change image - Version 

```
nano nginx-deployment.yml 
```


### Version 1: (optical nicer)

```
# Ändern des images von nginx:1.21 -> auf 1.22
# danach 
kubectl apply -f . && watch kubectl get pods 
```

### Version 2: 

```
# Ändern des images von nginxinc/nginx-unprivileged:1.28 -> auf 1.29
# danach 
kubectl apply -f . && kubectl get all && kubectl get pods -w
```
