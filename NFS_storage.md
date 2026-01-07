# NFS Storage with MicroK8s

We will use the upstream NFS CSI driver. First, deploy the NFS provisioner using the official Helm chart.

---

## 1. Enable Helm3 and Add NFS CSI Driver Repository

```bash
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update
```

---

## 2. Install the NFS CSI Driver

Install the Helm chart under the `kube-system` namespace:

```bash
microk8s helm3 install \
    csi-driver-nfs \
    csi-driver-nfs/csi-driver-nfs \
    --namespace kube-system \
    --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet
```

---

## 3. Wait for CSI Controller and Node Pods

```bash
microk8s kubectl wait pod --selector app.kubernetes.io/name=csi-driver-nfs --for condition=ready --namespace kube-system
```

**Example output:**

```
pod/csi-nfs-controller-764db9c84b-l7ldl condition met
pod/csi-nfs-node-279c6 condition met
pod/csi-nfs-node-55qls condition met
pod/csi-nfs-node-qdhmr condition met
```

---

## 4. List Available CSI Drivers

```bash
microk8s kubectl get csidrivers
```

**Expected output:**

```
NAME                            ATTACHREQUIRED   PODINFOONMOUNT   STORAGECAPACITY   TOKENREQUESTS   REQUIRESREPUBLISH   MODES        AGE
nfs.csi.k8s.io                  false            false            false             <unset>         false               Persistent   4m27s
rook-ceph.cephfs.csi.ceph.com   true             false            false             <unset>         false               Persistent   10m
rook-ceph.rbd.csi.ceph.com      true             false            false             <unset>         false               Persistent   10m
```

---

## 5. Create a StorageClass for NFS

Edit the following manifest to match your NFS server and mount point.

**sc-nfs.yaml**
```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 10.0.4.20        # Edit to match your NFS server
  share: /nfs_share        # Edit to match your NFS mountpoint
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```

Apply the manifest:

```bash
kubectl apply -f sc-nfs.yaml
```

---

## 6. Verify StorageClass

```bash
kubectl get sc
```

**Sample output:**

```
NAME                          PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
ceph-rbd                      rook-ceph.rbd.csi.ceph.com   Delete          Immediate              true                   21m
microk8s-hostpath (default)   microk8s.io/hostpath         Delete          WaitForFirstConsumer   false                  12d
nfs-csi                       nfs.csi.k8s.io               Delete          Immediate              false                  36s
```

---

## 7. Create a PersistentVolumeClaim (PVC)

**pvc-nfs.yaml**
```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-test
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

Apply the manifest:

```bash
kubectl apply -f pvc-nfs.yaml
```

---

## 8. Check PVC Status

```bash
kubectl get pvc
```

**Sample output:**

```
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        VOLUMEATTRIBUTESCLASS   AGE
my-pvc-test   Bound    pvc-b242a112-dddf-4229-b4e5-b953cf47c5de   5Gi        RWO            nfs-csi             <unset>                 4m35s
```

---

## 9. Describe PVC Details

```bash
kubectl describe pvc my-pvc-test
```

**Sample output:**

```
Name:          my-pvc-test
Namespace:     default
StorageClass:  nfs-csi
Status:        Bound
Volume:        pvc-b242a112-dddf-4229-b4e5-b953cf47c5de
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: nfs.csi.k8s.io
               volume.kubernetes.io/storage-provisioner: nfs.csi.k8s.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      5Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       <none>
Events:        <none>
```


