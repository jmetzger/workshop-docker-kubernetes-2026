# Veraltete Images erkennen mit version-checker

## Hintergrund

**version-checker** (jetstack, Apache 2.0) laeuft als Pod im Cluster, beobachtet
alle laufenden Container und vergleicht deren Image-Versionen mit der jeweiligen
Registry. Das Ergebnis ist eine Prometheus-Metrik die man direkt mit kubectl
abfragen kann — kein Prometheus, kein Grafana noetig.

---

## Schritt 1: version-checker deployen

```
kubectl apply -k https://github.com/jetstack/version-checker/deploy/yaml
```

```
namespace/version-checker created
serviceaccount/version-checker created
clusterrole.rbac.authorization.k8s.io/version-checker created
clusterrolebinding.rbac.authorization.k8s.io/version-checker created
service/version-checker created
deployment.apps/version-checker created
```

Standardmaessig prueft version-checker nur Pods mit einer opt-in Annotation.
Mit `--test-all-containers` prueft er alle Pods im Cluster automatisch:

```
kubectl -n version-checker patch deployment version-checker \
  --type=json \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args","value":["--test-all-containers"]}]'
```

Warten bis der Pod bereit ist:

```
kubectl -n version-checker rollout status deployment/version-checker
```

```
deployment "version-checker" successfully rolled out
```

---

## Schritt 2: Ergebnisse abfragen

kubectl kann den Metrics-Endpunkt direkt ueber den API-Server proxyen —
kein separater Prozess noetig:

```
kubectl get --raw \
  /api/v1/namespaces/version-checker/services/version-checker:8080/proxy/metrics \
  | grep version_checker_is_latest_version
```

Nur veraltete Images (`value = 0` bedeutet: nicht aktuell):

```
kubectl get --raw \
  /api/v1/namespaces/version-checker/services/version-checker:8080/proxy/metrics \
  | grep "version_checker_is_latest_version{" | grep " 0$"
```

**Beispielausgabe aus unserem Cluster:**
```
...{container="calico-apiserver",current_version="v3.31.2",latest_version="v3.32.0",...} 0
...{container="coredns",current_version="v1.13.1",latest_version="v1.14.3",...} 0
...{container="nginx",current_version="sha256:6e234...",latest_version="trixie-perl@sha256:3d55...",image="nginx",...} 0
```

Jede Zeile enthaelt `current_version` (was laeuft) und `latest_version` (was
in der Registry aktuell ist).

---

## Schritt 3: Abfragefehler anzeigen

Nicht jede Registry kann version-checker automatisch abfragen — `registry.k8s.io`
zum Beispiel erlaubt keine oeffentliche Tag-Suche:

```
kubectl get --raw \
  /api/v1/namespaces/version-checker/services/version-checker:8080/proxy/metrics \
  | grep version_checker_image_failures_total | grep -v "^#"
```

**Beispiel:**
```
version_checker_image_failures_total{container="kube-apiserver",image="registry.k8s.io/kube-apiserver:v1.35.2",...} 1
version_checker_image_failures_total{container="etcd",image="registry.k8s.io/etcd:3.6.6-0",...} 1
```

Das sind bekannte Grenzen — nicht jede Registry unterstuetzt die Docker
Registry API v2 fuer Tag-Listings.

---

## Private Registry

version-checker unterstuetzt beliebige private Registries (Harbor, Nexus,
Artifactory) ueber Umgebungsvariablen im Deployment. Der `<NAME>`-Suffix
erlaubt mehrere Registries gleichzeitig:

```
VERSION_CHECKER_SELFHOSTED_HOST_INTERN=registry.intern:5000
VERSION_CHECKER_SELFHOSTED_USERNAME_INTERN=nutzer
VERSION_CHECKER_SELFHOSTED_PASSWORD_INTERN=passwort
```

Danach prueft version-checker Images von `registry.intern:5000/...` genauso
wie oeffentliche Images — `latest_version` kommt dann aus der internen Registry.
Das ist der Standardweg in Air-Gapped Umgebungen.

---

## Aufraeumen

```
kubectl delete -k https://github.com/jetstack/version-checker/deploy/yaml
```

---

## Zusammenfassung

| Was | Befehl |
|-----|--------|
| Alle Images mit Status | `kubectl get --raw .../proxy/metrics \| grep is_latest` |
| Nur veraltete Images | `... \| grep is_latest \| grep ' 0$'` |
| Abfragefehler | `... \| grep failures_total \| grep -v '^#'` |
| Private Registry | `VERSION_CHECKER_SELFHOSTED_*` ENV im Deployment |
