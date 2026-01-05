Registry use in this guide
https://hub.docker.com/

Generating a token for your docker user name
Use this token to login into Docker

docker login

Pushing to the registry requires that the image is tagged with your-hub-username/image-name:tag
You can build can tag the image 
docker build . -t phantanphong98/mynginx:public
Or tag an existing image using image ID, check the image ID using this command

docker images

The ID can be found in the IMAGE ID column
You then use this ID to tag the image you want
docker tag 058f4935d1cb phantanphong98/mynginx:public

Push the image to Docker Registry
docker push phantanphong98/mynginx:public

At this point we are ready to microk8s kubectl apply -f a deployment with our image:

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


We refer to the image as image: phantanphong98/mynginx:public. Kubernetes will search for the image in its default registry, docker.io.
