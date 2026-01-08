# MicroK8s Service Ports

Services can be placed into two groups:

- **localhost:** Only available within the host.
- **public:** Services binding to the default host interface are available from outside the host and thus are subject to access restrictions.

---

## Services Binding to the Default Host Interface

| Port      | Service          | Access Restrictions                                                                 |
|-----------|------------------|-------------------------------------------------------------------------------------|
| 16443     | API server       | SSL encrypted. Clients need to present a valid password from a Static Password File. |
| 10250     | kubelet          | Anonymous authentication is disabled. X509 client certificate is required.           |
| 10255     | kubelet          | Read only port for the Kubelet.                                                     |
| 25000     | cluster-agent    | Proper token required to authorise actions.                                         |
| 12379     | etcd             | SSL encrypted. Client certificates required to connect.                             |
| 10257     | kube-controller  | Serve HTTPS with authentication and authorization.                                  |
| 10259     | kube-scheduler   | Serve HTTPS with authentication and authorization.                                  |
| 19001     | dqlite           | SSL encrypted. Client certificates required to connect.                             |
| 4789/udp  | calico           | Calico networking with VXLAN enabled.                                               |

---

## Services Binding to the Localhost Interface

| Port   | Service         | Description                                 |
|--------|----------------|---------------------------------------------|
| 10248  | kubelet        | Localhost healthz endpoint.                 |
| 10249  | kube-proxy     | Port for the metrics server to serve on.    |
| 10251  | kube-scheduler | Port on which to serve HTTP insecurely.     |
| 10252  | kube-controller| Port on which to serve HTTP insecurely.     |
| 10256  | kube-proxy     | Port to bind the health check server.       |
| 2380   | etcd           | Used for peer connections.                  |
| 1338   | containerd     | Metrics port.                               |

---

## Containerd and etcd

Both these services are exposed through Unix sockets.

| Service     | Socket                                      |
|-------------|---------------------------------------------|
| containerd  | `unix:///var/snap/microk8s/common/run/`     |
