# Kubernetes Cluster Configuration

This repository contains the Kubernetes manifests and configurations for my
personal lab cluster managed with [Flux](https://fluxcd.io/).

## Cluster Overview

The cluster can be deployed using either [k3d](https://k3d.io/) (lightweight
Kubernetes in Docker) or [Canonical K8s](https://github.com/canonical/k8s-snap).
It includes Intel GPU passthrough for media applications and host path volumes
for data persistence.

## Repository Structure

```
flux/
├── base/
│   ├── charts/          # Helm chart releases
│   ├── kustomizations/  # External Kustomizations (Gateway API, Calico)
│   └── manifests/       # Raw Kubernetes manifests
└── clusters/
    └── homeserv/        # Cluster-specific configurations
configs/
├── k3d/                 # k3d cluster config
└── canonical-k8s/       # Canonical K8s cluster config
```

## Applications

### Media Stack

| Application                           | Description              |
| ------------------------------------- | ------------------------ |
| [Jellyfin](https://jellyfin.org/)     | Media streaming server   |
| [Deluge](https://deluge-torrent.org/) | Torrent client           |
| [SABnzbd](https://sabnzbd.org/)       | Usenet client            |
| [Prowlarr](https://prowlarr.com/)     | Indexer manager          |
| [Radarr](https://radarr.video/)       | Movie collection manager |

### Infrastructure

| Application                                                                          | Description                  |
| ------------------------------------------------------------------------------------ | ---------------------------- |
| [Istio](https://istio.io/) (on K3d; canonical K8s ships with cilium)                 | Service mesh                 |
| [Gateway API](https://gateway-api.sigs.k8s.io/)                                      | Kubernetes ingress evolution |
| [Cert-Manager](https://cert-manager.io/)                                             | TLS certificate management   |
| [Tailscale Operator](https://tailscale.com/kb/1236/kubernetes-operator)              | Secure networking            |
| [Local Path Provisioner](https://github.com/rancher/local-path-provisioner)          | Local storage provisioner    |
| [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)                  | Resource metrics             |
| [Node Feature Discovery](https://github.com/kubernetes-sigs/node-feature-discovery)  | Hardware detection           |
| [Intel Device Plugins](https://github.com/intel/intel-device-plugins-for-kubernetes) | Intel GPU device plugins     |
| [Calico](https://www.tigera.io/project-calico/) (optional)                           | CNI for advanced networking  |

### Observability

| Application                                                                                            | Description                      |
| ------------------------------------------------------------------------------------------------------ | -------------------------------- |
| [Grafana](https://grafana.com/)                                                                        | Metrics visualization            |
| [Loki](https://grafana.com/oss/loki/)                                                                  | Log aggregation                  |
| [Tempo](https://grafana.com/oss/tempo/)                                                                | Distributed tracing              |
| [Mimir](https://grafana.com/oss/mimir/)                                                                | Long-term metrics storage        |
| [Kiali](https://kiali.io/)                                                                             | Istio service mesh observability |
| [K8s Monitoring](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/) | Kubernetes monitoring stack      |
| [Prometheus Operator CRDs](https://prometheus-operator.dev/)                                           | Prometheus custom resources      |

## Prerequisites

- [k3d](https://k3d.io/) or
  [Canonical K8s](https://github.com/canonical/k8s-snap)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- A GPG key for SOPS secret encryption and decryption

## Deployment

### Option 1: k3d Cluster

1. Create the cluster:
   ```bash
   k3d cluster create --config configs/k3d/homeserv.yaml
   ```
   Or from GitHub directly:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/theZMC/k8s/refs/heads/main/configs/k3d/homeserv.yaml | k3d cluster create --config /dev/stdin
   ```

### Option 2: Canonical K8s

1. Create the cluster:
   ```bash
   sudo k8s bootstrap --config configs/canonical-k8s/homeserv.yaml
   cilium config set devices "enp+"  # Prevent Cilium from interfering with Tailscale
   ```
   Or from GitHub directly:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/theZMC/k8s/refs/heads/main/configs/canonical-k8s/homeserv.yaml | sudo k8s bootstrap --config -
   cilium config set devices "enp+"
   ```

### Bootstrap Flux

1. Install Flux components:
   ```bash
   kubectl apply -k flux/clusters/homeserv/flux-system
   ```
   Or from GitHub directly:
   ```bash
   kubectl apply -k https://github.com/theZMC/k8s.git/flux/clusters/homeserv/flux-system?ref=main
   ```

2. Create the SOPS decryption secret (must match the public key in
   `flux/clusters/homeserv/secrets/.sops.pub.asc`):
   ```bash
   echo "Paste GPG key (Ctrl+D when done):" && \
      kubectl create secret generic sops-gpg --from-file=sops.asc=/dev/stdin -n flux-system
   ```

3. Flux will automatically reconcile and deploy all applications.

## Management

### Secrets

Secrets are encrypted with [SOPS](https://github.com/getsops/sops) and stored in
`flux/clusters/homeserv/secrets/`. The public key is available at
`flux/clusters/homeserv/secrets/.sops.pub.asc`.

To encrypt a new secret:

```bash
sops --encrypt --in-place flux/clusters/homeserv/secrets/your-secret.yaml
```

### Updating Applications

This repository uses [Renovate](https://docs.renovatebot.com/) for automated
dependency updates. Manual updates can be made by modifying the respective
HelmRelease or Kustomization files in `flux/clusters/homeserv/`.

## Features

- **GitOps**: All cluster state is declaratively managed via Git
- **Automatic reconciliation**: Flux continuously monitors the repository
- **Secret management**: Encrypted secrets with SOPS
- **GPU support**: Intel GPU passthrough for Jellyfin hardware transcoding
- **Observability**: Complete LGTM stack (Loki, Grafana, Tempo, Mimir)
- **Secure networking**: Tailscale integration for secure remote access
