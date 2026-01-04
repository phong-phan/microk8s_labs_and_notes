High availability is automatically enabled on MicroK8s for clusters with three or more nodes.

There are three components necessary for a highly available Kubernetes cluster:

- There must be more than one node available at any time.
- The control plane must be running on more than one node so that losing a single node would not render the cluster inoperable.
- The cluster state must be in a datastore that is itself highly available.

Setup HA for Microk8s
1. Install version 1.19 or higher
2. At least 3 nodes.

Install the first node and join member nodes to the cluster
- Following the ./microk8s.md guide

Working with HA
- All nodes of the HA cluster run the master control plane. A subset of the cluster nodes (at least three) maintain a copy of the Kubernetes dqlite database
- Database maintenance involves a voting process through which a leader is elected. Apart from the voting nodes there are non-voting nodes silently keeping a copy of the database.These nodes are on standby to take over the position of a departing voter
- There are nodes that neither vote nor replicate the database. These nodes are called spare

There are three node roles
voters: replicating the database, participating in leader election
standby: replicating the database, not participating in leader election
spare: not replicating the database, not participating in leader election

- Checking the status of the cluster
microk8s status
microk8s is running
high-availability: yes
  datastore master nodes: 10.0.4.18:19001 10.0.4.19:19001 10.0.4.17:19001
  datastore standby nodes: none
addons:
  enabled:
    dashboard            # (core) The Kubernetes dashboard
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    ingress              # (core) Ingress controller for external access
    metallb              # (core) Loadbalancer for your Kubernetes cluster
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
    observability        # (core) A lightweight observability stack for logs, traces and metrics
    storage              # (core) Alias to hostpath-storage add-on, deprecated
  disabled:
    cert-manager         # (core) Cloud native certificate management
    cis-hardening        # (core) Apply CIS K8s hardening
    community            # (core) The community addons repository
    gpu                  # (core) Alias to nvidia add-on
    host-access          # (core) Allow Pods connecting to Host services smoothly
    kube-ovn             # (core) An advanced network fabric for Kubernetes
    mayastor             # (core) OpenEBS MayaStor
    minio                # (core) MinIO object storage
    nvidia               # (core) NVIDIA hardware (GPU and network) support
    prometheus           # (core) Prometheus operator for monitoring and logging
    rbac                 # (core) Role-Based Access Control for authorisation
    registry             # (core) Private image registry exposed on localhost:32000
    rook-ceph            # (core) Distributed Ceph storage using Rook

-  To ensure the health of the cluster the following timings should be taken into account
If the leader node gets “removed” ungracefully, e.g. it crashes and never comes back, it will take up to 5 seconds for the cluster to elect a new leader.
Promoting a non-voter to a voter takes up to 30 seconds. This promotion takes place when a new node enters the cluster or when a voter crashes.

- To remove a node gracefully, first run the leave command on the departing node
microk8s leave
- The node will be marked as ‘NotReady’ (unreachable) in Kubernetes. To complete the removal of the departing node, issue the following on any of the remaining nodes
microk8s remove-node <node_name>
- In the case we are not able to call microk8s leave from the departing node, e.g. due to a node crash, we need to call microk8s remove-node with the --force flag:
microk8s remove-node <node_name> --force
