Install clusterctl
The clusterctl CLI tool handles the lifecycle of a Cluster API management cluster

Setup a management cluster
The management cluster hosts the CAPI providers. You can use a MicroK8s cluster as a management cluster:

Quick setup guide to bring up a MicroK8s cluster

sudo snap install microk8s --classic
sudo microk8s status --wait-ready
mkdir -p ~/.kube/
sudo microk8s.config  > ~/.kube/config
sudo microk8s enable dns

When setting up the management cluster place its kubeconfig under ~/.kube/config so other tools such as clusterctl can discover and interact with it.


Prepare the infrastructure provider
As the name indicated, this will be the provider (mostly cloud) where you will bring your infrastructure up on
