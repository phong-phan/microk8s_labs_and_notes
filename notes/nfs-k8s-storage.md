# Install NFS Server as Cluster Storage for Kubernetes

> **Note**
> SELinux is disabled for testing purposes.

---

## Overview

This guide describes how to install and configure an **NFS server** to be used as shared storage for a Kubernetes cluster.

A **dedicated disk** is recommended for storing NFS data.

- In **VMware**, attaching an additional disk is straightforward.
- On **on-premise servers**, the procedure is mostly the same except for disk handling:
  - Use a physical disk if available.
  - If the server supports **hot-add**, no reboot is required.
  - Otherwise, rescan the disk bus or reboot as a last resort.
- For a **clean OS installation**, attaching a new disk is trivial and safe.

---

## Server-Side Configuration (NFS Server)

### Update System and Install Required Packages

```bash
yum update -y
yum install nfs-utils -y
```

---

### Enable and Start NFS Service

```bash
systemctl enable --now nfs-server.service
```

---

### Create NFS Export Directory

```bash
mkdir /nfs_share
chmod 777 /nfs_share
```

---

### Configure NFS Exports

```bash
echo "/nfs_share *(rw,sync,no_subtree_check,no_root_squash)" | tee -a /etc/exports
```

---

### Apply Export Configuration

```bash
exportfs -arv
```

---

## Firewall Configuration

Allow required NFS-related services through the firewall:

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

---

## Client-Side Configuration (All Kubernetes Nodes)

### Install NFS Client Utilities

```bash
yum install nfs-utils -y
```

---

### Mount the NFS Share

```bash
mount -t nfs 10.0.4.20:/nfs_share /k8s_data/
```

---

### Persistent Mount Configuration (`/etc/fstab`)

To automatically mount the NFS share after reboot, add the following entry to `/etc/fstab`:

```fstab
10.0.4.20:/nfs_share   /k8s_data   nfs   _netdev,noatime,nofail   0   0
```

---

## Notes and Best Practices

- `_netdev` ensures the mount occurs after network initialization
- `nofail` prevents boot failure if NFS is temporarily unavailable
- Avoid `soft` mounts for Kubernetes workloads to prevent data corruption
- For production environments:
  - Use **NFSv4**
  - Enable and properly configure **SELinux**
  - Restrict exports to specific CIDRs instead of `*`
