# Exercise kube-bench (for control plane) - Scanning Cluster based on CIS Benchmark Kubernetes

## PreChecks / Cleanup 

```
# if you have a job kube-bench from the last exercise -> delete it
kubectl get jobs | grep kube-bench
# Only if it was there 
kubectl delete jobs kube-bench 
```

## Walkthrough

```
# Node-Namen anzeigen
kubectl get nodes

# Taint des Control-Plane-Nodes pruefen
# Hinweis: Node-Name hat das Format k8s-tln<nr>-cp (z.B. k8s-tln1-cp)
kubectl describe node k8s-tln1-cp | grep -i taints
```

```
# Control-Plane schedulable machen
kubectl taint nodes k8s-tln1-cp node-role.kubernetes.io/control-plane:NoSchedule-
kubectl describe node k8s-tln1-cp | grep -i taints
```

```
cd
mkdir -p manifests/kube-bench-cis-cp
cd manifests
cd kube-bench-cis-cp
```

```
wget https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
```

```
# nodeName in template/spec: ergänzen wie folgt
# Hinweis: k8s-tln1-cp durch deinen Node-Namen ersetzen (siehe kubectl get nodes)
spec:
  template:
    spec:
      nodeName: k8s-tln1-cp
      containers:
        - command: ["kube-bench"]
 
```

```
kubectl apply -f job.yaml
kubectl get pods -o wide 
# Durch Euren Pod ersetzen
# kubectl logs kube-bench-j76s9
# Oder: einfacher -> zeigt logs des 1. Pods des jobs
kubectl logs job/kube-bench
```

```
# Ausgabe
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 API Server
```

```
kubectl logs job/kube-bench > report.txt
```

## Beispiel für Fehlerbehebung auf Node (anonymous auth)

### FAIL: 

```
[FAIL] 1.2.15 Ensure that the --profiling argument is set to false (Automated)
```

```
# Remediations
1.2.15 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the below parameter.
--profiling=false
```

### Fix: Walkthrough 

```
# Auf den Control-Plane-Node wechseln
ssh cp
```

```
cd /etc/kubernetes/manifests/
nano kube-apiserver.yaml
```

```
# changes from cis-scan 
- --profiling=false  # ergänzen
```

```
kubectl cluster-info
kubectl -n kube-system get pods
kubectl -n kube-system describe pods kube-apiserver
```

```
exit
```


### After Fix: Walkhrough 

```
kubectl delete -f job.yaml
kubectl apply -f job.yaml
kubectl logs job/kube-bench > report-afterfix.txt
diff report.txt report-afterfix.txt
```

```
[PASS] 1.2.15 Ensure that the --profiling argument is set to false (Automated)
```

## Make it non-scheduable again 

```
kubectl taint nodes k8s-tln1-cp node-role.kubernetes.io/control-plane:NoSchedule
```


## Reference:

  * https://hub.docker.com/r/aquasec/kube-bench
  * https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
