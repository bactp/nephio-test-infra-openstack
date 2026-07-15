# Component Versions — Nephio Management Cluster on OpenStack

This document tracks the software versions used to bootstrap and run the Nephio
management cluster on OpenStack via this repository's Ansible playbooks
(`e2e/provision/`). Update it whenever a version is bumped in the playbooks.

Last verified: 2026-07-14

## Node / OS

| Component | Version / Value | Source |
|---|---|---|
| Base image | `Ubuntu22-CAPI` (Ubuntu 22.04, pre-baked CAPI image) | `roles/capo/vars/main.yml` (`openstack_image_name`) |
| Node flavor | `30a13991-3518-4c99-b2e2-059839d4af36` (flavor name `nephio`, ≥ 8 vCPU / ≥ 16 GB RAM) | `roles/openstack-main/vars/main.yml` (`flavor`) |
| Cluster topology | 1 control-plane (`sre-control`) + 2 workers (`sre-worker01`, `sre-worker02`) | `roles/openstack-main/vars/main.yml` (`instance_tag`) |
| Kubernetes | v1.29 (kubeadm, `pkgs.k8s.io/core:/stable:/v1.29`) | `roles/k8s_control/tasks/main.yml`, `mgmt.yml` (`k8s.version: v1.29.2`) |
| Container runtime | Docker CE + `containerd.io` (latest from Docker's official apt repo — **not version-pinned**) | `roles/k8s_control/tasks/main.yml` |
| CNI | Cilium (latest stable, resolved dynamically from `cilium-cli/main/stable.txt` — **not version-pinned**) | `roles/k8s_control/tasks/main.yml` |
| Docker config | `time-namespaces: false` (workaround for OpenStack VM kernels without `CONFIG_TIME_NS`) | `roles/k8s_control/tasks/main.yml` |

## Cluster API / OpenStack provisioning

| Component | Version | Source |
|---|---|---|
| clusterctl / Cluster API core | v1.13.3 | `roles/capo/vars/main.yml` (`clusterctl_version`) |
| CAPO (cluster-api-provider-openstack) | **unpinned** — resolved to whatever `clusterctl init --infrastructure openstack` considers latest at install time (currently `v0.14.6`) | `roles/capo/tasks/main.yml` |
| ORC (openstack-resource-controller) | v2.4.0 | `roles/capo/vars/main.yml` (`orc_version`) |
| cert-manager | Auto-installed by `clusterctl init` (version tied to the clusterctl release, not separately pinned) | `roles/capo/tasks/main.yml` |
| kustomize | latest (installed via upstream install script — **not version-pinned**) | `roles/capo/tasks/main.yml` |

> ⚠️ CAPO is intentionally left unpinned to track upstream, but this means the ORC version above must stay compatible with whatever CAPO version gets pulled (check CAPO's `go.mod` for its required `k-orc/openstack-resource-controller` version before bumping either one).

## Nephio core

| Component | Version | Source |
|---|---|---|
| kpt CLI | v1.0.0-beta.49 | `playbooks/cluster.yml` |
| porchctl | v3.0.0 | `playbooks/cluster.yml` |
| Python `kubernetes` client | 26.1.0 | `playbooks/cluster.yml` |
| Nephio catalog | [`github.com/bactp/catalog`](https://github.com/bactp/catalog) @ `main` | `roles/install/defaults/main.yml`, `roles/bootstrap/defaults/main.yml` (`nephio_catalog_repo_uri`) |
| porch | catalog `nephio/core/porch` @ `main` | `roles/install/defaults/main.yml` |
| nephio-operator | catalog `nephio/core/nephio-operator` @ `main` | `roles/install/defaults/main.yml` |
| GitOps engine | **ArgoCD** (catalog `nephio/core/argo-cd` @ `main`) — not Config Sync | `roles/install/defaults/main.yml` |
| network-config | catalog `nephio/optional/network-config` @ `main` | `roles/install/defaults/main.yml` |

## Nephio WebUI

| Component | Version | Source |
|---|---|---|
| Package | catalog `nephio/optional/webui` @ `main` | `roles/install/defaults/main.yml` |
| `kpt-backstage-plugins` image | `v5.0.0` | catalog `nephio/optional/webui/deployment.yaml` |
| `gen-configmap-fn` image | `v6.0.0` | catalog `nephio/optional/webui/Kptfile` |

## Supporting infrastructure (all via Nephio catalog @ `main` unless noted)

| Component | Catalog package |
|---|---|
| Gitea (in-cluster git server) | `distros/sandbox/gitea` |
| cert-manager (kpt copy, in addition to the clusterctl-installed one) | `distros/sandbox/cert-manager` |
| MetalLB | `distros/sandbox/metallb` + `distros/sandbox/metallb-sandbox-config` |
| Resource backend | `nephio/optional/resource-backend` |
| Cluster API (kpt-managed copy) | `infra/capi/cluster-capi` |
| Longhorn storage provisioner | `infra/capi/longhorn-storage-provisioner` |
| Flux Helm controllers (DR) | `nephio/optional/flux-helm-controllers` |
| MinIO (DR) | `nephio/optional/minio` |
| Velero (binary) | `v1.7.0` |
| gtp5g kernel module (free5gc E2E) | `v0.8.3` |

## Ansible Galaxy roles / collections (`galaxy-requirements.yml`)

| Name | Version |
|---|---|
| `andrewrothstein.docker_engine` | v0.3.0 |
| `andrewrothstein.podman` | v2.1.1 |
| `andrewrothstein.kind` | v1.2.10 |
| `andrewrothstein.kubectl` | v1.3.4 |
| `andrewrothstein.kpt` | 0.0.3 |
| `darkwizard242.cni` | 2.4.0 |
| `community.general` | 8.5.0 |
| `kubernetes.core` | 3.0.1 |
| `community.docker` | 3.8.1 |
| `ansible.posix` | 1.5.4 |

## Known unpinned components (track upstream "latest" — verify before every bootstrap)

- CAPO (`clusterctl init --infrastructure openstack`)
- Docker CE / `containerd.io` (plain `apt install`, no version pin)
- Cilium CLI (`cilium-cli/main/stable.txt`)
- kustomize (upstream install script)
- cert-manager (bundled with clusterctl init)
