# kubectl installatieren und bash completion

## Client installieren

```
# Danach in PATH reinpacken. Am besten als root 
curl -LO https://dl.k8s.io/release/v1.36.0/bin/linux/amd64/kubectl
cp -a kubectl /usr/local/bin
```

## Bash completion  

```
# Prerequisites bash completion paket ist aktivieren
kubectl completion bash > /etc/bash_completion.d/kubect
su - 
```
