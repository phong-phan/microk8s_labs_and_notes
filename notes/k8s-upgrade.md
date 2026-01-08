# Kubernetes Cluster Upgrade Notes (MicroK8s)

## Generic Advice

- For production, always make sure that a **working backup** of the Kubernetes cluster database is available.
- Upgrade **only one node at a time**.
- Start with the **control plane** first. After all control plane nodes are upgraded, proceed with the Kubernetes **worker nodes**, upgrading them one by one (for larger clusters, in small batches).
- Make sure you **upgrade by one minor version at a time**. Before upgrading, refer to the release notes to review:
  - Breaking changes
  - Removed APIs
  - Deprecated APIs  
  Ensure these changes do not affect your cluster.
- **Cordon and drain nodes** prior to upgrading, and restore them afterward to ensure application workloads are not affected.

---

## Example Upgrade Scenario

We will upgrade a cluster with **3 nodes**:

- `docker1`
- `docker2`
- `docker3`

---

## Drain Node Before Upgrade

```bash
kubectl drain docker3 --ignore-daemonsets
```

---

## Verify Pod Eviction and Node Status

Verify that all pods previously running on `docker3` have been removed and rescheduled on other nodes.  
Also ensure the node is marked as `SchedulingDisabled`.

```bash
microk8s kubectl get node
microk8s kubectl get pod -o wide
```

---

## Upgrade MicroK8s Version

- Current version on all nodes: **v1.32.9**
- Target version: **v1.34.3**

Check and upgrade MicroK8s:

```bash
snap refresh microk8s --channel 1.34.3/stable
```

---

## Verify Node Version After Upgrade

```bash
kubectl get nodes
```

Expected output example:

```text
NAME      STATUS   ROLES    AGE   VERSION
docker    Ready    <none>   9d    v1.32.9
docker2   Ready    <none>   44h   v1.32.9
docker3   Ready    <none>   31h   v1.34.3
```

---

## Uncordon the Node

```bash
microk8s kubectl uncordon docker3
microk8s kubectl get node
```

---

## Rollback Procedure (If Upgrade Fails)

Drain the node again:

```bash
microk8s kubectl drain docker3
```

Revert MicroK8s to the previous version:

```bash
snap revert microk8s
```

Verify node status:

```bash
microk8s kubectl get node
```

Example output:

```text
NAME      STATUS                     ROLES    AGE   VERSION
docker    Ready                      <none>   9d    v1.32.9
docker2   Ready                      <none>   44h   v1.32.9
docker3   Ready,SchedulingDisabled   <none>   31h   v1.32.9
```

Uncordon the node:

```bash
microk8s kubectl uncordon docker3
microk8s kubectl get node
```

---

## Upgrade Remaining Nodes

Repeat the same upgrade process for:

- `docker1`
- `docker2`

Verify that the entire cluster is running the expected version.

---

## Post-Upgrade Testing (Sample Workload)

Deploy a test workload:

```bash
microk8s kubectl create deploy microbot-2 --image dontrebootme/microbot:v1 -n demo
microk8s kubectl scale deploy microbot-2 --replicas 4 -n demo
microk8s kubectl expose deploy microbot-2 --port 80 --type NodePort -n demo
```

---

## Verify Pods Distribution

```bash
microk8s kubectl get pod -l app=microbot-2 -o wide -n demo
```

Example output:

```text
NAME                          READY   STATUS    RESTARTS   AGE   IP             NODE
microbot-2-85d8655fb9-2s8cq   1/1     Running   0          17m   10.1.239.44    docker
microbot-2-85d8655fb9-8v76n   1/1     Running   0          17m   10.1.227.134   docker3
microbot-2-85d8655fb9-klrtz   1/1     Running   0          17m   10.1.100.71    docker2
microbot-2-85d8655fb9-x8x7f   1/1     Running   0          17m   10.1.239.58    docker
```

---

## Verify Service

```bash
microk8s kubectl get svc microbot-2 -n demo
```

Example output:

```text
NAME         TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
microbot-2   NodePort   10.152.183.134   <none>        80:31648/TCP   18m
```

---

## Verify Load Balancing (NodePort)

Because a **NodePort** service is used, verify load balancing across all pods.

> Ensure each node can resolve other node hostnames.
> You can use:
> - A local DNS server, or
> - Temporary `/etc/hosts` entries

Run the following commands:

```bash
curl --silent docker1:31648 | grep hostname
curl --silent docker2:31648 | grep hostname
curl --silent docker2:31648 | grep hostname
curl --silent docker3:31648 | grep hostname
curl --silent docker3:31648 | grep hostname
```

### Sample Output

```text
<p class="centered">Container hostname: microbot-2-85d8655fb9-2s8cq</p>
<p class="centered">Container hostname: microbot-2-85d8655fb9-8v76n</p>
<p class="centered">Container hostname: microbot-2-85d8655fb9-klrtz</p>
<p class="centered">Container hostname: microbot-2-85d8655fb9-x8x7f</p>
```

Different pod hostnames confirm that traffic is being load-balanced correctly.
