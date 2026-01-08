# MicroK8s Notes

## Prerequisites

* Ubuntu or any other Linux distribution that supports **snapd**
* If using **Windows** or **macOS**, MicroK8s can be installed via **Multipass**
* Minimum hardware requirements:

  * **Disk**: 20 GB
  * **Memory**: 4 GB

---

## Installation

### Install MicroK8s

```bash
sudo snap install microk8s --classic --channel=1.33
```

---

## MicroK8s Channels

When installing MicroK8s, you can specify a **channel**. A channel consists of:

* **Track**: Kubernetes version (for example: `1.29`)
* **Risk level**: `stable`, `candidate`, `beta`, or `edge`

If no channel is specified during installation, Snap defaults to:

```
latest/stable
```

### Example: Install MicroK8s v1.29 (stable)

```bash
sudo snap install microk8s --classic --channel=1.29/stable
```

### List Available Channels

```bash
sudo snap info microk8s
```

---

## Post-Installation Setup

### Add User to microk8s Group

```bash
sudo usermod -a -G microk8s $USER
```

### Configure kubeconfig Directory

```bash
mkdir -p ~/.kube
chmod 0700 ~/.kube
```

Re-login to apply group membership:

```bash
su - $USER
```

---

## Check Cluster Status

```bash
microk8s status --wait-ready
```

---

## Accessing Kubernetes

MicroK8s provides its own `kubectl` command to interact with the Kubernetes API:

```bash
microk8s kubectl
```

You can also install a standalone `kubectl` binary like any other Kubernetes distribution. It will still read from the standard kubeconfig file.

---

## kubectl Alias and Autocompletion

To reduce typing when working with Kubernetes, you can alias `kubectl` to `k` and enable shell autocompletion.

---

### Alias kubectl to k

#### Current Shell Only

```bash
alias k='kubectl'
```

#### Permanent Alias

Add the alias to your shell configuration file.

---

### Enable Autocompletion

#### Bash

```bash
kubectl completion bash
```

Edit your Bash configuration:

```bash
vi ~/.bashrc
```

---

#### Zsh

```bash
kubectl completion zsh
```

Edit your Zsh configuration:

```bash
vi ~/.zshrc
```

Add the following lines:

```bash
alias k='kubectl'
echo 'alias k=kubectl' >> ~/.zshrc
echo 'compdef __start_kubectl k' >> ~/.zshrc
source <(kubectl completion zsh)
```

Reload your shell or open a new terminal for changes to take effect.

---

### Fix: compdef Command Not Found

If you encounter the error:

```
command not found: compdef
```

Add the following **at the beginning** of your `~/.zshrc` file:

```bash
autoload -Uz compinit
compinit
```

---

## References

* Kubernetes kubectl installation guide:

  * [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

