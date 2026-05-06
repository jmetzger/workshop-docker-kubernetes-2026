# SecurityContext - Pods sicher haerten (CIS 5.2.2 / 5.2.4 / 5.2.6 / 5.2.9)

## Hintergrund

Ein `securityContext` legt fest, mit welchen Rechten ein Pod oder Container laeuft.
Ohne expliziten SecurityContext laeuft jeder Container standardmaessig als **root** —
ein Angreifer, der aus dem Container ausbricht, hat damit direkt root-Zugriff auf den Node.

| Einstellung | Beschreibung | CIS |
|-------------|-------------|-----|
| `runAsNonRoot: true` | Container darf nicht als root starten | 5.2.6 |
| `runAsUser` | Konkrete UID setzen | 5.2.6 |
| `readOnlyRootFilesystem` | Filesystem ist schreibgeschuetzt | 5.2.4 |
| `allowPrivilegeEscalation: false` | Kein sudo/setuid moeglich | 5.2.5 |
| `capabilities.drop: ["ALL"]` | Alle Linux-Capabilities entfernen | 5.2.7 |
| `seccompProfile: RuntimeDefault` | Syscall-Filter aktivieren | 5.2.2 |

---

## Schritt 1: Problem zeigen - Pod laeuft als root

```
cd
mkdir -p manifests/securitycontext
cd manifests/securitycontext
```

```
kubectl create namespace securitycontext
```

```
# vi 01-pod-root.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-root
spec:
  containers:
  - name: app
    image: nginx:1.27
```

```
kubectl apply -f 01-pod-root.yml -n securitycontext
```

Als welcher User laeuft der Container?

```
kubectl exec -n securitycontext pod-root -- id
```

**Erwartete Ausgabe:**
```
uid=0(root) gid=0(root) groups=0(root)
```

Kann der Container ins Filesystem schreiben?

```
kubectl exec -n securitycontext pod-root -- touch /test-file && echo "Schreiben moeglich!"
```

Das ist das Problem: root im Container, volles Schreibrecht.

---

## Schritt 2: runAsNonRoot und runAsUser

```
# vi 02-pod-nonroot.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nonroot
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - name: app
    image: nginxinc/nginx-unprivileged:1.27
```

```
kubectl apply -f 02-pod-nonroot.yml -n securitycontext
kubectl get pod pod-nonroot -n securitycontext
```

User pruefen:

```
kubectl exec -n securitycontext pod-nonroot -- id
```

**Erwartete Ausgabe:**
```
uid=1000 gid=1000 groups=1000
```

> Hinweis: Wir verwenden `nginxinc/nginx-unprivileged` statt `nginx:1.27`.
> Das Standard-nginx-Image besteht darauf, als root zu starten — und wuerde
> mit `runAsNonRoot: true` sofort crashen. nginx-unprivileged ist fuer
> non-root gebaut und lauscht auf Port **8080** statt 80.

---

## Schritt 3: readOnlyRootFilesystem + emptyDir

Ein read-only Filesystem verhindert, dass Malware sich ins Container-Filesystem schreibt.
Fuer Verzeichnisse, die nginx zum Schreiben braucht, legen wir `emptyDir`-Volumes an.

```
# vi 03-pod-readonly.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-readonly
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - name: app
    image: nginxinc/nginx-unprivileged:1.27
    securityContext:
      readOnlyRootFilesystem: true
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
kubectl apply -f 03-pod-readonly.yml -n securitycontext
kubectl get pod pod-readonly -n securitycontext
```

Schreibversuch ins Root-Filesystem:

```
kubectl exec -n securitycontext pod-readonly -- touch /test-file
```

**Erwarteter Fehler:**
```
touch: /test-file: Read-only file system
```

Schreiben in erlaubtes Verzeichnis funktioniert noch:

```
kubectl exec -n securitycontext pod-readonly -- touch /tmp/test-file && echo "tmp ist schreibbar"
```

---

## Schritt 4: Capabilities droppen + Privilege Escalation verbieten

Linux-Capabilities geben Prozessen einzelne root-Privilegien (z.B. `NET_RAW` fuer
raw Sockets, `SYS_ADMIN` fuer Kernel-Operationen). Standardmaessig erbt ein Container
eine Reihe davon — wir droppen alle.

```
# vi 04-pod-nocaps.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nocaps
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - name: app
    image: nginxinc/nginx-unprivileged:1.27
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
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
kubectl apply -f 04-pod-nocaps.yml -n securitycontext
kubectl get pod pod-nocaps -n securitycontext
```

Capabilities pruefen:

```
kubectl exec -n securitycontext pod-nocaps -- cat /proc/1/status | grep Cap
```

**Erwartete Ausgabe:** Alle Capability-Werte sind `0000000000000000` — keine Capabilities aktiv.

---

## Schritt 5: Seccomp-Profil aktivieren

Seccomp (Secure Computing Mode) filtert erlaubte Linux-Syscalls.
`RuntimeDefault` verwendet das vom Container-Runtime vorgegebene Profil
und blockiert gefaehrliche Syscalls wie `ptrace` oder `mount`.

```
# vi 05-pod-hardened.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hardened
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginxinc/nginx-unprivileged:1.27
    ports:
    - containerPort: 8080
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
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
kubectl apply -f 05-pod-hardened.yml -n securitycontext
kubectl get pod pod-hardened -n securitycontext
```

Pod laeuft und ist voll gehaertet:

```
kubectl exec -n securitycontext pod-hardened -- id
kubectl exec -n securitycontext pod-hardened -- curl -s http://localhost:8080 | head -5
```

---

## Schritt 6: Vergleich mit kube-bench / Scan

Was wuerde kube-bench zu unserem urspruenglichen Pod sagen?

```
kubectl get pod pod-root -n securitycontext -o json | python3 -c "
import sys, json
p = json.load(sys.stdin)
c = p['spec']['containers'][0]
sc = c.get('securityContext', {})
psc = p['spec'].get('securityContext', {})
checks = {
  'runAsNonRoot':             psc.get('runAsNonRoot', False),
  'readOnlyRootFilesystem':   sc.get('readOnlyRootFilesystem', False),
  'allowPrivilegeEscalation': sc.get('allowPrivilegeEscalation', True) == False,
  'capabilities dropped':     sc.get('capabilities', {}).get('drop', []) == ['ALL'],
  'seccompProfile':           psc.get('seccompProfile', {}).get('type') == 'RuntimeDefault',
}
for k, v in checks.items():
    status = 'PASS' if v else 'FAIL'
    print(f'  {status}  {k}')
"
```

Dasselbe fuer den gehaerteten Pod:

```
kubectl get pod pod-hardened -n securitycontext -o json | python3 -c "
import sys, json
p = json.load(sys.stdin)
c = p['spec']['containers'][0]
sc = c.get('securityContext', {})
psc = p['spec'].get('securityContext', {})
checks = {
  'runAsNonRoot':             psc.get('runAsNonRoot', False),
  'readOnlyRootFilesystem':   sc.get('readOnlyRootFilesystem', False),
  'allowPrivilegeEscalation': sc.get('allowPrivilegeEscalation', True) == False,
  'capabilities dropped':     sc.get('capabilities', {}).get('drop', []) == ['ALL'],
  'seccompProfile':           psc.get('seccompProfile', {}).get('type') == 'RuntimeDefault',
}
for k, v in checks.items():
    status = 'PASS' if v else 'FAIL'
    print(f'  {status}  {k}')
"
```

---

## Aufraeumen

```
kubectl delete pod pod-root pod-nonroot pod-readonly pod-nocaps pod-hardened -n securitycontext
```

---

## Zusammenfassung

| Was | Wo konfiguriert | Warum |
|-----|----------------|-------|
| `runAsNonRoot` / `runAsUser` | `pod.spec.securityContext` | Kein root im Container |
| `readOnlyRootFilesystem` | `container.securityContext` | Kein Schreiben ins Filesystem |
| `allowPrivilegeEscalation: false` | `container.securityContext` | Kein sudo/setuid |
| `capabilities.drop: ["ALL"]` | `container.securityContext` | Keine Kernel-Privilegien |
| `seccompProfile: RuntimeDefault` | `pod.spec.securityContext` | Syscall-Filter |
| `emptyDir` Volumes | `volumes` + `volumeMounts` | Schreibbare Verzeichnisse fuer die App |
| `nginxinc/nginx-unprivileged` | Image | nginx ohne root-Anforderung |

## Referenzen

  * https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  * https://kubernetes.io/docs/concepts/security/pod-security-standards/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.2)
