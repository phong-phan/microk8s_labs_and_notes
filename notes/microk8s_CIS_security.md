Kubernetes specific CIS configurations is a set of recommendations on the Kubernetes services setup and configuration.

With the v1.28 MicroK8s release a cis-hardening addon is included as part of the core addons.
As described below, this addon reconfigures the cluster nodes to comply with the CIS recommendations v1.24.

Enable addon:
microk8s enable cis-hardening
