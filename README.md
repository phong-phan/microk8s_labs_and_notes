# MicroK8s Labs and Research

This repository documents my research, testing, and practical notes on [MicroK8s](https://microk8s.io/) — a lightweight, production-grade Kubernetes distribution from Canonical.

---

## Lab Topology

- **Cluster Nodes:**  
  3 nodes running AlmaLinux 9 (minimum required for HA)
- **NFS Server:**  
  1 node providing NFS storage for the cluster

### IP Address Allocation

| Hostname   | Role        | IP Address   | OS Image                          | Kernel Version                   | Container Runtime      |
|------------|-------------|--------------|-----------------------------------|----------------------------------|-----------------------|
| node1      | Cluster     | 10.0.4.18    | AlmaLinux 9.5 (Teal Serval)       | 5.14.0-503.35.1.el9_5.x86_64     | containerd://1.6.36   |
| node2      | Cluster     | 10.0.4.19    | AlmaLinux 9.7 (Moss Jungle Cat)   | 5.14.0-611.9.1.el9_7.x86_64      | containerd://1.6.36   |
| node3      | Cluster     | 10.0.4.17    | AlmaLinux 9.7 (Moss Jungle Cat)   | 5.14.0-611.9.1.el9_7.x86_64      | containerd://1.6.36   |
| nfs-server | NFS Server  | 10.0.4.20    |                                   |                                  |                       |

---

## MicroK8s Cluster Status

- **MicroK8s is running**
- **High Availability:** Enabled
- **Datastore master nodes:**  
  - 10.0.4.18:19001  
  - 10.0.4.19:19001  
  - 10.0.4.17:19001  
- **Datastore standby nodes:** none

---
## Enabled Addons

- `trivy`                — Kubernetes-native security scanner
- `community`            — The community addons repository
- `dashboard`            — The Kubernetes dashboard
- `dns`                  — CoreDNS
- `ha-cluster`           — High availability configuration
- `helm`                 — Helm package manager
- `helm3`                — Helm 3 package manager
- `hostpath-storage`     — Storage class from host directory
- `ingress`              — Ingress controller for external access
- `metallb`              — Load balancer for the Kubernetes cluster
- `metrics-server`       — Metrics Server for API access to service metrics
- `observability`        — Lightweight observability stack (logs, traces, metrics)
- `rbac`                 — Role-Based Access Control
- `registry`             — Private image registry (localhost:32000)
- `rook-ceph`            — Distributed Ceph storage using Rook
- `storage`              — Alias to hostpath-storage (deprecated)

---

## Notes

- This project is a living knowledge base and will be updated as I continue to explore MicroK8s features, storage, security, and HA.
- See individual markdown files for detailed guides and troubleshooting steps.

---

