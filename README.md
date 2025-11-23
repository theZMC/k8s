# Kubernetes Cluster Configuration

This repository contains the Kubernetes manifests and configurations for my
personal lab cluster managed with [Flux](https://fluxcd.io/).

## Cluster Overview

The cluster is deployed using [k3d](https://k3d.io/) (lightweight Kubernetes in
Docker) and includes GPU passthrough for media applications and host path
volumes for data persistence.

## Applications

This cluster runs several self-hosted applications:

| Application                                                                          | Description                                                          |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| [jellyfin](https://jellyfin.org/)                                                    | Media streaming server                                               |
| [deluge](https://deluge-torrent.org/)                                                | Torrent client                                                       |
| [sabnzbd](https://sabnzbd.org/)                                                      | Usenet client                                                        |
| [prowlarr](https://prowlarr.com/)                                                    | Indexer manager                                                      |
| [radarr](https://radarr.video/)                                                      | Movie collection manager                                             |
| [nextcloud](https://nextcloud.com/)                                                  | Personal cloud storage                                               |
| [intel-device-plugins](https://github.com/intel/intel-device-plugins-for-kubernetes) | GPU device plugins for Intel graphics                                |
| [istio](https://istio.io/)                                                           | Service mesh                                                         |
| [kiali](https://kiali.org/)                                                          | Istio observability                                                  |
| [cert-manager](https://cert-manager.io/)                                             | TLS Certificate management                                           |
| [tailscale-operator](https://github.com/tailscale/tailscale)                         | Tailscale Kubernetes operator for secure networking                  |
| [metrics-server](https://github.com/kubernetes-sigs/metrics-server)                  | Resource metrics                                                     |
| [node-feature-discovery](https://github.com/kubernetes-sigs/node-feature-discovery)  | Hardware detection                                                   |
| [lgtm](https://grafana.com/)                                                         | Logging, tracing, and monitoring stack (Loki, Grafana, Tempo, Mimir) |
| [local-path-provisioner](https://github.com/rancher/local-path-provisioner)          | Local storage provisioner                                            |

## Prerequisites

- [k3d](https://k3d.io/) or
  [canonical k8s](https://github.com/canonical/k8s-snap)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Deployment

1. Create the k3d cluster:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/theZMC/k8s/refs/heads/main/k3d-configs/homeserv.yaml | k3d cluster create --config /dev/stdin
   ```
   Alternatively, if using canonical k8s:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/theZMC/k8s/refs/heads/main/k8s-configs/homeserv.yaml | sudo k8s bootstrap --config -
   ```

1. Install the gotk-components into the cluster with kustomize:
   ```bash
   kubectl apply -k https://github.com/theZMC/k8s.git/clusters/homeserv?ref=main
   ```

1. Create the secret for SOPS decryption:
   ```bash
   kubectl create secret generic sops-gpg --from-file=path/to/the/private.key -n flux-system
   ```
