# K9s Installation Guide

[K9s](https://k9s.io/) is a CLI tool to manage your Kubernetes cluster in style.  
Its key movements are inspired by Vim, which is my primary editor.

---

## Installation on Linux

### 1. Using Homebrew

The easiest way to install K9s is via [Homebrew](https://brew.sh):

```bash
brew install k9s
```

### 2. Building from Source

Alternatively, you can build K9s from source:

1. **Clone the repository:**

    ```bash
    git clone https://github.com/derailed/k9s.git
    cd k9s
    ```

2. **Build and run the executable:**  
   Make sure you have `cmake` installed on your system, as it is required for compiling.

    ```bash
    make build
    ./execs/k9s
    ```

---

## Preflight Checks

- **Terminal Colors:**  
  K9s uses 256-color terminal mode. Set this in your `.bashrc` or `.zshrc`:

    ```bash
    export TERM=xterm-256color
    ```

- **Resource Editing:**  
  If you have issues with the resource edit command, ensure your `EDITOR` and/or `KUBE_EDITOR` environment variables are set:

    ```bash
    export KUBE_EDITOR=vim
    export EDITOR=vim
    ```

---

## Cluster Configuration

K9s looks for your cluster config at `~/.kube/config` by default.  
If you want to use a different location, set the following environment variable:

```bash
export KUBECONFIG=/path/to/your/kubeconfig
```
