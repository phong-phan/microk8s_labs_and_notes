Kubernetes manages containerised applications based on images. These images can be created locally, or more commonly are fetched from a remote image registry

Using Docker to build a custom image base on nginx

Content of the Dockerfile:
FROM nginx

Build the image and tag it with 'local' in the directory where Dockerfile placed

docker build . -t mynginx:local

This will generate a new local image tagged mynginx:local

Working with locally built image without a registry

When an image is built it is cached on the Docker daemon used during the build. Having run the docker build . -t mynginx:local command, you can see the newly built image by running
docker images

The image we created is known to Docker. However, Kubernetes is not aware of the newly built image. This is because your local Docker daemon is not part of the MicroK8s Kubernetes cluster. We can export the built image from the local Docker daemon and “inject” it into the MicroK8s image cache like this:

docker save mynginx > myimage.tar
microk8s ctr image import myimage.tar

Now we can list the images present in MicroK8s
microk8s ctr images ls

Testing access to this image using a sample manifest file:

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

Note: 
- Make sure the imagePullPolicy is set to Ǹever as shown above, otherwise MicroK8s will continue to try and pull from Dockerhub, even if the image is present in the local registry.
- containerd will not cache images with the latest tag so make sure you avoid it.
