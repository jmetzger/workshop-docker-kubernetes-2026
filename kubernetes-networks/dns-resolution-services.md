# DNS Resolution of services 

## Exercise 

```
kubectl run podtest --rm -ti --image busybox
```

## Example with my-nginx 

```
# in sh
wget -O - http://my-nginx
wget -O - http://my-nginx.jochen
wget -O - http://my-nginx.jochen.svc
wget -O - http://my-nginx.jochen.svc.cluster.local
```

## How to find the FQDN (Full qualified domain name) 

```
# in busybox (clusterIP)
### Schritt 1: Service-IP ausfindig machen
wget -O - http://my-nginx
# z.B. 10.109.24.227 

### Schritt 2: nslookup mit dieser Service-IP
nslookup 10.109.24.227
# Ausgabe 
# name = my-nginx.jochen.svc.cluster.local
```
