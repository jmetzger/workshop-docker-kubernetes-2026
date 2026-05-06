# Container-Runtime-Alternativen zu Docker

## Übersicht

| Tool | Typ | Build | Push | Run | Daemon | Privileged | Primärer Use Case |
|---|---|:---:|:---:|:---:|:---:|:---:|---|
| **Podman** | CLI Drop-in | ✅ | ✅ | ✅ | ❌ | ❌ | Docker-Ersatz, rootless Dev |
| **nerdctl** | CLI für containerd | ✅ | ✅ | ✅ | ❌ | teilweise | Dev-Workflow auf containerd |
| **Buildah** | Image-Build | ✅ | ✅ | ❌ | ❌ | ❌ | Scripted Builds |
| **kaniko** | CI/CD Build+Push | ✅ | ✅ (direkt) | ❌ | ❌ | ❌ | CI/CD in Kubernetes |
| **Skopeo** | Image-Transfer | ❌ | ✅ | ❌ | ❌ | ❌ | copy/inspect/sign |
| **containerd** | Low-level Runtime | ❌ | ❌ | ✅ | ✅ | — | Kubernetes CRI |
| **CRI-O** | Low-level Runtime | ❌ | ❌ | ✅ | ✅ | — | Kubernetes CRI (minimal) |
| **LXC/LXD** | System-Container | ❌ | ❌ | ✅ | ✅ | — | Leichte VMs |

---

## Wichtige Unterschiede zu Docker

### Podman
- Kein Daemon → kein `dockerd`-Prozess
- Rootless out-of-the-box → sicherer
- `alias docker=podman` funktioniert meist reibungslos
- `podman-compose` als Drop-in für `docker-compose`

### nerdctl
- Docker-kompatibles CLI direkt auf containerd
- Aktiv gepflegt (v2.2.2, März 2026), offizielles containerd-Subprojekt
- Nützlich zum Debuggen von Kubernetes-Nodes (`--namespace=k8s.io`)
- Lazy-pulling, Image-Encryption, IPFS-Distribution als Extras

### Buildah
- Nur Build, kein Runtime
- Gut für CI-Pipelines ohne laufenden Daemon
- Oft in Kombination mit Podman verwendet

### kaniko
- Ursprünglich von Google entwickelt
- Baut **und pusht** in einem Schritt direkt in die Registry (`--destination`)
- Kein privilegierter Container nötig → ideal für Kubernetes-Pods
- GitLab empfiehlt kaniko explizit für CI/CD ohne Docker-Socket

```yaml
# .gitlab-ci.yml
build:
  image:
    name: gcr.io/kaniko-project/executor:debug
  script:
    - /kaniko/executor
      --context $CI_PROJECT_DIR
      --dockerfile Dockerfile
      --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
      --cache=true
```

### Skopeo
- Kein Build, kein Run — nur Image-Operationen
- copy, inspect, sign zwischen Registries
- Ideal für Registry-Mirroring und Image-Promotion in CI/CD

---

## crictl — CRI-Debugging

`crictl` ist ein generischer CRI-Client für **containerd und CRI-O**:

```yaml
# /etc/crictl.yaml

# containerd
runtime-endpoint: unix:///run/containerd/containerd.sock

# CRI-O
runtime-endpoint: unix:///run/crio/crio.sock
```

Oder per Flag: `crictl --runtime-endpoint unix:///run/crio/crio.sock ps`

---

## Empfehlungen nach Use Case

| Use Case | Empfehlung |
|---|---|
| Lokale Entwicklung (Docker-Ersatz) | **Podman** |
| CI/CD in Kubernetes ohne Daemon | **kaniko** |
| Kubernetes-Node debuggen | **crictl** / **nerdctl** |
| Image-Build in Skripten | **Buildah** |
| Registry-Operationen | **Skopeo** |
| Containerd direkt nutzen | **nerdctl** |

