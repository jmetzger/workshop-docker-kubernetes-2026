# API Server Profiling - Warum nicht in Produktion?

## Was ist Profiling?

Der Kubernetes API-Server ist in Go geschrieben. Go hat ein eingebautes
Profiling-Framework (`pprof`), das Laufzeitdaten des Prozesses über HTTP
bereitstellt. Profiling ist ein **Entwicklungs- und Debugging-Tool** — es
hilft Entwicklern, Speicherlecks, CPU-Hotspots und Deadlocks zu finden.

**Default:** `--profiling=true`

Das bedeutet: in jedem frisch installierten Kubernetes-Cluster ist Profiling
aktiv, ohne dass man etwas konfigurieren muss.

## Wo ist der Endpunkt erreichbar?

```
https://<api-server>:6443/debug/pprof/
```

Mit kubectl direkt abrufbar (kein Extra-Token noetig — der normale kubeconfig reicht):

```
kubectl get --raw /debug/pprof/
```

## Was gibt der Endpunkt preis?

| Endpunkt | Inhalt |
|----------|--------|
| `/debug/pprof/heap` | Speicherbelegung + interne Stack Traces mit Bibliothekspfaden und Versionsnummern |
| `/debug/pprof/goroutine` | Alle laufenden Go-Threads des API-Servers mit Callstacks |
| `/debug/pprof/allocs` | Alle Speicherallokationen seit Prozessstart |
| `/debug/pprof/profile` | 30-Sekunden CPU-Profil (blockiert den Request) |
| `/debug/pprof/trace` | Kompletter Laufzeit-Trace des Prozesses |
| `/debug/pprof/cmdline` | Startparameter des API-Server-Prozesses |

Beispielausgabe von `/debug/pprof/heap`:

```
heap profile: 262: 18177752 [2631: 72114632] @ heap/1048576
# sigs.k8s.io/json@v0.0.0-20250730193827-2d320260d730/...
# k8s.io/apiextensions-apiserver/pkg/apis/apiextensions/v1beta1/marshal.go:204
# k8s.io/apimachinery/pkg/util/json/json.go:46
```

Sichtbar: **exakte Versionsnummern interner Bibliotheken** und **interne
Codepfade** des API-Servers.

## Warum gehoert das nicht in Produktion?

Ein Angreifer mit gueltigem Kubernetes-Token (z.B. aus einem kompromittierten
Pod) kann den Endpunkt abfragen und bekommt:

1. **Versions-Fingerprinting** — exakte Softwareversionen fuer gezielte
   Exploit-Suche (z.B. bekannte CVEs in spezifischen Library-Versionen)

2. **Interne Architektur** — welche Komponenten wie zusammenhaengen,
   Codepfade und interne Ablaeufe

3. **Laufzeit-Zustand** — was der API-Server gerade verarbeitet,
   Speichermuster, Thread-Aktivitaet

4. **CPU-Profil** — Verarbeitungsmuster, die Rueckschluesse auf
   Clusteraktivitaeten erlauben

Das **Prinzip der minimalen Angriffsfläche** (Principle of Least Exposure)
verlangt: was nicht benoetigt wird, wird deaktiviert.

## CIS Benchmark: 1.2.15

```
[FAIL] 1.2.15 Ensure that the --profiling argument is set to false (Automated)

Remediation:
Edit the API server pod specification file
/etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameter:
--profiling=false
```

## Fix

```
# Auf dem Control Plane Node
nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Folgende Zeile in der `command`-Liste ergaenzen:

```
- --profiling=false
```

Der API-Server startet als Static Pod automatisch neu — kein manueller
Restart noetig. Nach ca. 30 Sekunden:

```
kubectl cluster-info
# Verbindung wieder hergestellt -> Neustart erfolgreich
```

Endpunkt ist danach nicht mehr erreichbar:

```
kubectl get --raw /debug/pprof/
# Error from server (NotFound): ...
```

## Referenzen

  * https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 1.2.15)
  * https://pkg.go.dev/net/http/pprof
