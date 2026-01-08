# Managing DockerHub Rate Limits in MicroK8s

By default, MicroK8s uses DockerHub as the source for pulling images required for the Kubernetes cluster. DockerHub enforces a rate limit on image downloads. When this limit is reached, pods are blocked in the `ImagePullBackOff` state (as shown in `microk8s kubectl get po`), and describing the blocked pods reports a "Too Many Requests" error.

To avoid hitting DockerHub rate limits, consider the following workarounds:

---

## 1. Login to DockerHub

DockerHub rate limits are less strict for authenticated users, though they may still cause problems depending on your usage. Providing a username and password can be achieved by editing the MicroK8s configuration or by setting up a secret and configuring your pods to use it.

**DockerHub Rate Limits:**

| User Type                        | Rate Limit                                      |
|-----------------------------------|-------------------------------------------------|
| Unauthenticated (Anonymous)       | 100 pulls per 6 hours per IPv4 address or IPv6 /64 subnet |
| Authenticated (Free Personal)     | 200 pulls per 6 hours                           |

### 1.1 Configure containerd

> **Note:** For MicroK8s clusters, this action needs to be repeated on all nodes.

You can configure your DockerHub credentials in the containerd configuration so they are used automatically when pulling images from DockerHub, without specifying an image pull secret manually for each container.

Edit `/var/snap/microk8s/current/args/containerd-template.toml` and add the following section (TOML format, indentation does not matter):

```toml
[plugins."io.containerd.grpc.v1.cri".registry.configs."registry-1.docker.io".auth]
username = "DOCKERHUB_USERNAME"
password = "DOCKERHUB_PASSWORD"
```

Afterwards, restart MicroK8s:

```bash
microk8s stop
microk8s start
```

### 1.2 Using Image Pull Secret

Kubernetes allows you to create a secret containing DockerHub credentials and then use it for pulling images.

---

## 2. Use Other Container Registry

You can edit deployments to source images from other public registries, such as `rocks.canonical.com`, `gcr.io`, or `quay.io`. This involves manually editing the spec of the pods running in your cluster.

To change the image used by Calico CNI, use this command:

```bash
kubectl edit deploy/calico-kube-controllers
```

Change the image:

- **From:**
  ```
  Image:      docker.io/calico/kube-controllers:v3.28.1
  ```
- **To:**
  ```
  Image:      quay.io/calico/kube-controllers:v3.28.1
  ```

---

## 3. Use Private Image Registry to Mirror DockerHub

For production environments, it is highly recommended to use a private image registry to mirror DockerHub. In this setup, when you specify an image from `docker.io` (e.g., `nginx` or `docker.io/nginx`), MicroK8s will retrieve the image from your private registry instead of DockerHub.

> **Note:** Your private registry should contain at least the images you need (e.g., `nginx`).

Assuming your private registry is available at `https://my.registry.internal:5000`, the required configuration depends on your MicroK8s version.

### For MicroK8s Version 1.23 or Newer

MicroK8s 1.23 and newer use separate `hosts.toml` files for each image registry. For `docker.io`, this can be found at `/var/snap/microk8s/current/args/certs.d/docker.io/hosts.toml`.

Edit the file so that the contents look like this:

```toml
# /var/snap/microk8s/current/args/certs.d/docker.io/hosts.toml
server = "https://my.registry.internal:5000"
[host."my.registry.internal:5000"]
capabilities = ["pull", "resolve"]
```

Then, restart MicroK8s:

```bash
microk8s stop
microk8s start
```
