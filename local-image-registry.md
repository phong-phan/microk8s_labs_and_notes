# Using a Local Image Registry with MicroK8s

Kubernetes manages containerized applications based on images. These images can be created locally or, more commonly, fetched from a remote image registry.

## Building a Custom Image with Docker

Create a `Dockerfile` with the following content:

```dockerfile
FROM nginx
```

Build the image and tag it as `mynginx:local` in the directory where the `Dockerfile` is located:

```bash
docker build . -t mynginx:local
```

This will generate a new local image tagged `mynginx:local`.

## Working with Locally Built Images Without a Registry

When an image is built, it is cached on the Docker daemon used during the build.  

```bash
docker images
```

The image is known to Docker, but Kubernetes (via MicroK8s) is not aware of it.  
This is because your local Docker daemon is not part of the MicroK8s Kubernetes cluster.

To use the image with MicroK8s, export it from Docker and import it into the MicroK8s image cache:

```bash
docker save mynginx > myimage.tar
microk8s ctr image import myimage.tar
```

Now, list the images present in MicroK8s:

```bash
microk8s ctr images ls
```

## Testing the Image with a Sample Deployment

Use the following manifest to deploy your image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: mynginx:local
        imagePullPolicy: Never
        ports:
        - containerPort: 80
```

**Notes:**
- Make sure the `imagePullPolicy` is set to `Never`, otherwise MicroK8s will try to pull from Docker Hub even if the image is present locally.
- `containerd` will not cache images with the `latest` tag, so avoid using it.
