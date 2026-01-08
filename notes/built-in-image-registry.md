# Using the Built-in Image Registry with MicroK8s

Having a private Docker registry can significantly improve productivity by reducing the time spent uploading and downloading Docker images.  
MicroK8s ships with a built-in registry hosted within the Kubernetes cluster and exposed as a NodePort service on port `32000` of the localhost.

> **Note:**  
> This is an insecure registry. You may need to take extra steps to limit access.

---

## Building and Tagging Images

Build your image as usual.  
When tagging, use `localhost:32000/image_name:tag` instead of your Docker username to refer to the built-in registry.

```bash
docker build . -t localhost:32000/nginx:registry
```

---

## Configuring Docker to Trust the Insecure Registry

Pushing to this registry may fail unless Docker is configured to trust it.  
Edit `/etc/docker/daemon.json` and add:

```json
{
  "insecure-registries" : ["localhost:32000"]
}
```

Reload the configuration by restarting the Docker daemon:

```bash
sudo systemctl restart docker
```

---

## Pushing the Image

Push your image to the built-in registry:

```bash
docker push localhost:32000/nginx:registry
```

---

## Deploying the Image with MicroK8s

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
        image: localhost:32000/nginx:registry
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
microk8s kubectl apply -f <deployment-file>.yaml
```

---

## Additional Notes

- If you run MicroK8s inside a VM and build Docker images on the host, edit the `daemon.json` file and change the `insecure-registries` value from `localhost` to the IP address of the VM where MicroK8s is running.
- The same configuration is needed when using images from other nodes in your MicroK8s cluster (after joining nodes to form a cluster).
