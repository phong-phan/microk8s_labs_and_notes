# MicroK8s Quorum Recovery (dqlite)

In HA mode, the default datastore (dqlite) implements a Raft-based protocol where an elected leader holds the definitive copy of the database.  
If you permanently lose the majority of the cluster members that serve as database nodes (e.g., in a three-node cluster, you lose two), the cluster becomes unavailable.  
However, if at least one database node survives, you can recover the cluster with the following manual steps.

> **Note:**  
> This recovery process applies only to clusters using the default (dqlite) datastore of MicroK8s.  
> This process does **not** recover any data you have in PVs on the lost nodes.

---

## 1. Stop dqlite on All Nodes

```bash
microk8s stop
```

---

## 2. Backup the Database

Dqlite stores data and configuration files under `/var/snap/microk8s/current/var/kubernetes/backend/`.  
To make a safe copy of the current state, log into a surviving node and create a tarball of the dqlite directory:

```bash
tar -cvf backup.tar /var/snap/microk8s/current/var/kubernetes/backend
```

---

## 3. Set the New State of the Database Cluster

Locate the file `cluster.yaml` under `/var/snap/microk8s/current/var/kubernetes/backend/` and reflect the state of the cluster as dqlite sees it.  
Edit this file and remove settings for lost nodes.

**Example:**  
If `10.0.4.19` and `10.0.4.17` are down, remove their settings:

```yaml
- Address: 10.0.4.18:19001
  Role: 0
# Remove the following:
# - ID: 6530853229361614930
#   Address: 10.0.4.19:19001
#   Role: 0
# - ID: 9268895754662775525
#   Address: 10.0.4.17:19001
#   Role: 0
```

After editing, the file should only contain the running node (e.g., `10.0.4.18`).  
This converts the cluster into a single-node cluster. You may add more nodes later (see `./microk8s.md` for details).

---

## 4. Reconfigure the Node Using dqlite Utility

On the running node, run:

```bash
sudo /snap/microk8s/current/bin/dqlite \
  -s 127.0.0.1:19001 \
  -c /var/snap/microk8s/current/var/kubernetes/backend/cluster.crt \
  -k /var/snap/microk8s/current/var/kubernetes/backend/cluster.key \
  k8s ".reconfigure /var/snap/microk8s/current/var/kubernetes/backend/ /var/snap/microk8s/current/var/kubernetes/backend/cluster.yaml"
```

**Arguments explained:**

- `sudo`: Needed to run the command with administrative privileges.
- `-s 127.0.0.1:19001`: Endpoint of the dqlite service.
- `-c /var/snap/microk8s/current/var/kubernetes/backend/cluster.crt`: Path to the cluster's public certificate.
- `-k /var/snap/microk8s/current/var/kubernetes/backend/cluster.key`: Path to the cluster's private key.
- `k8s ".reconfigure ..."`: The command to reconfigure the database with the new cluster state.

---

## 5. Update the Rest of the Cluster Nodes

On all other nodes:

- Copy the `cluster.yaml`, `snapshot-abc-abc-abc`, `snapshot-abc-abc-abc.meta`, and segment files (`00000abcxx-00000abcxx`) from the node where you ran the reconfigure command.
- Create an `info.yaml` file that matches the one you created previously.

> **WARNING:**  
> Ensure you delete any leftover `snapshot-abc-abc-abc`, `snapshot-abc-abc-abc.meta`, `segment` (`00000abcxx-000000abcxx`, `open-abc`), and `metadata{1,2}` files.  
> Failing to do so may cause the nodes to fail to rejoin the cluster cleanly.

---

## 6. Restart the MicroK8s Services

On all nodes, run:

```bash
microk8s start
```

---

## 7. Cleanup the Lost Nodes from k8s

The lost nodes are registered in Kubernetes but should be in a NotReady state.  
To remove the lost node, run:

```bash
microk8s remove-node <node_name>
```

---

## 8. Restore HA

High Availability (HA) will be reattained when there are three or more nodes in the MicroK8s cluster.
