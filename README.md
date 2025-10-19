# Kubernetes Cluster Configuration

This repository contains the Kubernetes manifests and configurations for my
personal lab cluster managed with [Flux](https://fluxcd.io/).

## Cluster Overview

The cluster is deployed using [k3d](https://k3d.io/) (lightweight Kubernetes in
Docker) and includes GPU passthrough for media applications and host path
volumes for data persistence.

## Applications

This cluster runs several self-hosted applications:

- **Jellyfin** - Media streaming server
- **Deluge** - Torrent client
- **Sabnzbd** - Usenet client
- **Prowlarr** - Indexer manager
- **Radarr** - Movie collection manager
- **Nextcloud** - Personal cloud storage
- **Intel** - GPU device plugin for Intel graphics
- **Istio** - Service mesh
- **Kiali** - Istio observability
- **Cert-Manager** - Certificate management
- **Tailscale** - Ingress controller
- **Metrics Server** - Resource metrics
- **Node Feature Discovery** - Hardware detection
- **LTGM** - Loki, Tempo, Grafana, and Mimir stack for logging, tracing, and
  metrics

## Prerequisites

- [k3d](https://k3d.io/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Deployment

1. Create the k3d cluster:
   ```bash
   k3d cluster create --config k3d-configs/lab.yaml
   ```

2. Install the gotk-components into the cluster with kustomize:
   ```bash
   kubectl apply -k https://github.com/theZMC/k8s.git/clusters/lab?ref=main
   ```

3. Create the secret for SOPS decryption:
   ```bash
   kubectl create secret generic sops-gpg --from-file=path/to/the/private.key -n flux-system
   ```
