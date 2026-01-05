# Using a Public Image Registry with MicroK8s

This guide demonstrates how to use [Docker Hub](https://hub.docker.com/) as a public image registry with MicroK8s.

---

## 1. Generate a Docker Hub Token and Login

First, generate a token for your Docker Hub username (from your Docker Hub account settings).  
Use this token to log in:

```bash
docker login
```

---

## 2. Tagging Your Image for Docker Hub

To push an image to Docker Hub, it must be tagged as `your-hub-username/image-name:tag`.

- **Build and tag the image:**

    ```bash
    docker build . -t phantanphong98/mynginx:public
    ```

- **Or tag an existing image using its IMAGE ID:**

    1. List your images:

        ```bash
        docker images
        ```

    2. Tag the image (replace `IMAGE_ID` with the actual ID):

        ```bash
        docker tag IMAGE_ID phantanphong98/mynginx:public
        ```

---

## 3. Push the Image to Docker Hub

```bash
docker push phantanphong98/mynginx:public
```

---

## 4. Deploying the Image with MicroK8s

You can now use your public image in a Kubernetes deployment:

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
        image: phantanphong98/mynginx:public
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
microk8s kubectl apply -f <deployment-file>.yaml
```

---

## Notes

- Reference the image as `image: phantanphong98/mynginx:public` in your manifests.
- Kubernetes will search for the image in its default registry, `docker.io`.