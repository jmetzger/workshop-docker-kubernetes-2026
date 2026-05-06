# Veraltete Images finden und via private Registry aktualisieren

## Hintergrund

In Produktionsclustern laeuft typischerweise kein direkter Internetzugang von
den Nodes — alle Images kommen aus einer **privaten Registry** (Harbor, Nexus,
Artifactory). Das stellt ein besonderes Problem fuer Image-Updates dar:

```
Internet          Jump-Host           Private Registry       Cluster
(Docker Hub)  →   (hat Zugang)    →   (intern erreichbar) →  (Nodes)
nginx:1.28        pull + scan          push                    pull
```

Ohne diesen Prozess bleiben Images im Cluster veraltet — mit bekannten CVEs
die laengst gefixt waeren.

---

## Schritt 1: Inventarisierung — Was laeuft ueberhaupt?

### Alle Images cluster-weit auflisten

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u
```

### Kompakter: nur die Image-Namen ohne Namespace

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u
```

### Init-Container nicht vergessen

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.spec.initContainers[*].image}{"\n"}{end}' \
  | sort -u | grep -v "^$"
```

---

## Schritt 2: Anti-Pattern finden — latest-Tags und fehlende Tags

`latest` ist gefaehrlich: man weiss nicht welche Version genau laeuft,
Updates passieren unkontrolliert beim naechsten Pull.

### latest-Tags im Cluster finden

```
kubectl get pods -A -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data['items']:
    ns = item['metadata']['namespace']
    name = item['metadata']['name']
    for c in item['spec'].get('containers', []):
        img = c['image']
        if img.endswith(':latest') or ':' not in img:
            print(f'LATEST TAG: {ns}/{name} -> {img}')
"
```

**Erwartetes Ergebnis** (Beispiel aus unserem Cluster):
```
LATEST TAG: default/nginx -> nginx
```

`nginx` ohne Tag ist equivalent zu `nginx:latest` — man weiss nicht
welche Paketversionen darin sind.

### Images mit SHA-Digest pruefen (sichere Alternative zu Tags)

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u | grep "@sha256" | head -5
```

SHA-Digests sind unveraenderlich — sie zeigen ob ein Image wirklich
unveraendert geblieben ist, auch wenn der Tag ueberschrieben wurde.
Wenn keine SHA-Digest-Images laufen, gibt der Befehl nichts aus.

---

## Schritt 3: Test-Deployment mit veralteter Version

Namespace erstellen und ein Deployment mit einer aelteren nginx-Version starten:

```
kubectl create namespace img-<dein-name>
```

```
cd
mkdir -p manifests/img
cd manifests/img
```

```
# vi 01-deployment-alt.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-alt
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-alt
  template:
    metadata:
      labels:
        app: nginx-alt
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

```
kubectl apply -f . -n img-<dein-name>
kubectl get pods -n img-<dein-name>
```

Welches Image laeuft genau?

```
kubectl get pods -n img-<dein-name> \
  -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}'
```

---

## Schritt 4: Update-Workflow fuer private Registry (Theorie)

Dieser Schritt laeuft in Produktion so ab — im Workshop fuehren wir
ihn in Schritt 5 vereinfacht durch.

### Der Prozess (drei Stationen)

**Station 1: Jump-Host (hat Internetzugang)**

```
# Neue Version pullen
docker pull nginx:1.27

# Scannen BEVOR es in die Registry kommt
trivy image --severity CRITICAL,HIGH nginx:1.27

# Fuer private Registry taggen
docker tag nginx:1.27 registry.intern:5000/nginx:1.27
```

**Station 2: Private Registry**

```
# Nur bei gruener Scan-Ampel pushen
docker push registry.intern:5000/nginx:1.27
```

**Station 3: Cluster — Deployment aktualisieren**

```
kubectl set image deployment/nginx-alt \
  nginx=registry.intern:5000/nginx:1.27 \
  -n img-<dein-name>
```

Rollout-Status pruefen:

```
kubectl rollout status deployment/nginx-alt -n img-<dein-name>
```

Image im laufenden Pod verifizieren:

```
kubectl get pods -n img-<dein-name> \
  -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}'
```

---

## Schritt 5: Praxis — Image im Deployment aktualisieren

Da im Workshop keine private Registry vorhanden ist, verwenden wir
`kubectl set image` direkt mit einem neueren Docker-Hub-Tag.
In Produktion wuerde hier das private-Registry-Tag stehen.

```
kubectl set image deployment/nginx-alt nginx=nginx:1.27 -n img-<dein-name>
```

Rollout beobachten (warten bis fertig):

```
kubectl rollout status deployment/nginx-alt -n img-<dein-name>
```

Image im laufenden Pod verifizieren (erst NACH rollout status ausfuehren —
waehrend des Rollouts laufen kurz beide Versionen parallel):

```
kubectl get pods -n img-<dein-name> \
  -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}'
```

**Erwartete Ausgabe:**
```
nginx:1.27
```

Rollout-History — zeigt beide Revisionen:

```
kubectl rollout history deployment/nginx-alt -n img-<dein-name>
```

```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

---

## Schritt 6: Was tun wenn das Image kritische CVEs hat?

Wenn trivy CRITICAL-CVEs findet und kein Fix verfuegbar ist:

| Option | Beschreibung |
|--------|-------------|
| **Warten auf Upstream** | Hersteller muss Basis-Image aktualisieren |
| **Eigenes Image bauen** | FROM nginx:1.27 mit eigenen Patches obendrauf |
| **Schicht-Update** | `apt-get upgrade` im Dockerfile direkt (kurzfristig) |
| **Deployment stoppen** | Bei kritischem Risiko — lieber kein Service als ein verwundbarer |

### Schicht-Update als Ueberbrueckung

```
# vi Dockerfile
FROM nginx:1.24
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*
```

```
docker build -t registry.intern:5000/nginx-patched:1.24-fixed .
trivy image registry.intern:5000/nginx-patched:1.24-fixed
docker push registry.intern:5000/nginx-patched:1.24-fixed
```

---

## Schritt 7: Automatisierung — Renovate in Air-Gapped Umgebungen

**Renovate** (Open Source, Apache 2.0) kann auch in geschlossenen Umgebungen
eingesetzt werden:

```
Renovate laeuft als Job im Cluster
         ↓
prueft Kubernetes-Manifeste auf Image-Tags
         ↓
fragt interne Registry ab (Harbor/Nexus API)
         ↓
oeffnet automatisch Pull Requests bei neuen Tags
```

Beispiel-Konfiguration `renovate.json` fuer eine interne Registry:

```
{
  "registryAliases": {
    "docker.io": "registry.intern:5000"
  },
  "hostRules": [
    {
      "matchHost": "registry.intern:5000",
      "username": "renovate-bot",
      "password": "{{ secrets.REGISTRY_PASSWORD }}"
    }
  ]
}
```

---

## Aufraeumen

```
kubectl delete namespace img-<dein-name>
```

---

## Zusammenfassung: Checkliste fuer Image-Updates im isolierten Cluster

| Schritt | Aktion | Wer |
|---------|--------|-----|
| 1. Inventarisierung | `kubectl get pods -A -o jsonpath=...` | Ops |
| 2. latest-Tags finden | kubectl + python3 Query | Ops |
| 3. Neue Version pruefen | `docker pull` + `trivy image` | Ops/Security |
| 4. In private Registry pushen | `docker tag` + `docker push` | Ops |
| 5. Deployment aktualisieren | `kubectl set image` oder Manifest-Update | DevOps |
| 6. Rollout verifizieren | `kubectl rollout status` | Ops |
| 7. Automatisieren | Renovate selbst-hostbar im Cluster | Platform-Team |

**Kernregel:** Nie direkt aus Docker Hub in den Cluster — immer ueber eine
private Registry als Schleuse, mit Scan-Gate dazwischen.
