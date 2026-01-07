# Ceph Storage with MicroCeph and MicroK8s

With the 1.28 release, MicroK8s introduced a new `rook-ceph` addon that allows users to easily set up, import, and manage Ceph deployments via Rook.

This guide demonstrates how to set up a Ceph cluster with MicroCeph, add three virtual disks backed by local files, and import the Ceph cluster into MicroK8s using the `rook-ceph` addon.

---

## 1. Install MicroCeph

MicroCeph is a lightweight way of deploying a Ceph cluster with a focus on reduced ops.  
It is distributed as a snap and can be installed with:

```bash
sudo snap install microceph --channel=latest/edge
```

---

## 2. Bootstrap the Ceph Cluster

Initialize the Ceph cluster:

```bash
microceph cluster bootstrap
```

Check the status of the cluster and query the list of available disks (should be empty):

```bash
microceph.ceph status
```

**Sample output:**

```
  cluster:
    id:     0246729e-d9d4-45c1-84c7-a449e27e7ba4
    health: HEALTH_OK

  services:
    mon: 1 daemons, quorum docker (age 16s)
    mgr: docker(active, since 9s)
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

---

## 3. Add Virtual Disks

Create three files under `/mnt` to back loop devices. Each virtual disk is then added as an OSD to Ceph:

```bash
for l in a b c; do
  loop_file="$(mktemp -p /mnt XXXX.img)"
  truncate -s 1G "${loop_file}"
  loop_dev="$(losetup --show -f "${loop_file}")"
  minor="${loop_dev##/dev/loop}"
  mknod -m 0660 "/dev/sdi${l}" b 7 "${minor}"
  microceph disk add --wipe "/dev/sdi${l}"
done
```

---

## 4. Check Disk Status

Check the Ceph cluster status:

```bash
microceph.ceph status
```

**Sample output:**

```
  cluster:
    id:     0246729e-d9d4-45c1-84c7-a449e27e7ba4
    health: HEALTH_OK

  services:
    mon: 1 daemons, quorum docker (age 2m)
    mgr: docker(active, since 2m)
    osd: 3 osds: 3 up (since 15s), 3 in (since 18s)

  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 449 KiB
    usage:   81 MiB used, 2.9 GiB / 3 GiB avail
    pgs:     1 active+clean
```

List disks configured in MicroCeph:

```bash
microceph disk list
```

**Sample output:**

```
Disks configured in MicroCeph:
+-----+----------+-----------+
| OSD | LOCATION |   PATH    |
+-----+----------+-----------+
| 1   | docker   | /dev/sdia |
| 2   | docker   | /dev/sdib |
| 3   | docker   | /dev/sdic |
+-----+----------+-----------+
```

---

## 5. Customize Ceph Setup

As this cluster is local and used by a local MicroK8s deployment, set the replica count to 2, disable manager redirects, and set the bucket type for CRUSH rule:

```bash
microceph.ceph config set global osd_pool_default_size 2
microceph.ceph config set mgr mgr_standby_modules false
microceph.ceph config set osd osd_crush_chooseleaf_type 0
```

---

## 6. Connect MicroCeph to MicroK8s

The `rook-ceph` addon first appeared with the 1.28 release, so select a MicroK8s deployment channel greater or equal to 1.28.

Enable the Ceph addon:

```bash
microk8s enable rook-ceph
```

Import the external Ceph cluster managed by MicroCeph:

```bash
microk8s connect-external-ceph
```

