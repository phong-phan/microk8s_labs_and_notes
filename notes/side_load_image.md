# Image Side-Loading in MicroK8s

Image side-loading allows administrators to easily import one or more Docker (OCI) images (bundled in `.tar` archives) to all nodes of a MicroK8s cluster using a single command.

This is useful in scenarios such as:

- Environments with limited or constrained access to Docker Hub or other image registries.
- Environments with limited bandwidth and/or connection speeds.
- Importing private images that are not published in any public registry.
- When it is not possible or desired to configure and run a private image registry.

---

## Supported Images

MicroK8s supports importing standard OCI images from `.tar` archives.

> **Note:**  
> Image side-loading using the `microk8s images import` command is available in MicroK8s version 1.25 or newer.

---

## Importing an Image into the Cluster

To import an image into your MicroK8s cluster:

```bash
microk8s images import < myimage.tar
```

Example output when pushing to a 3-node cluster on cluster-agent port 25000:

```
Pushing OCI images to 10.0.4.18:25000
Pushing OCI images to 10.0.4.19:25000
Pushing OCI images to 10.0.4.17:25000
```

This command pushes the image to all nodes in your cluster.

---

## Loading an Image into the Local Node Only

To load an image into the local containerd daemon (not the entire cluster):

```bash
microk8s ctr image import - < nginx.tar
```
