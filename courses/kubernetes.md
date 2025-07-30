# Kubernetes

## Description

Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications.

## Directory

- [Kubernetes](#kubernetes)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [References](#references-1)
    - [Client Setup](#client-setup)
    - [Cluster Setup](#cluster-setup)
  - [Kubernetes Cluster](#kubernetes-cluster)
    - [Creating a Kubernetes Cluster](#creating-a-kubernetes-cluster)
    - [Reverse Proxy](#reverse-proxy)
  - [Helm](#helm)
    - [Installing Helm](#installing-helm)
    - [Helm Chart Repository](#helm-chart-repository)
    - [Helm Charts](#helm-charts)
    - [Helm Values](#helm-values)

## References

- [Kubernetes](https://kubernetes.io)

---

## Setup

This details the setup process of a Kubernetes environment from a client perspective, and a server perspective.

### References

- [Login node](https://github.com/irfanhakim-as/orked#login-node-1)
- [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux)
- [kubectx: Manual Installation (macOS and Linux)](https://github.com/ahmetb/kubectx#manual-installation-macos-and-linux)
- [pv-migrate: Installation](https://github.com/utkuozdemir/pv-migrate/blob/master/INSTALL.md)
- [k9s: Installation](https://github.com/derailed/k9s#installation)
- [kubectl df-pv: Installation](https://github.com/yashbhutwala/kubectl-df-pv#installation)

### Client Setup

> [!NOTE]  
> If you have already [set up a Kubernetes cluster](#cluster-setup) based on RKE2 following the guide provided by [Orked](https://github.com/irfanhakim-as/orked), the [Login node](https://github.com/irfanhakim-as/orked#login-node-1) should already have all of these set up. Use the following guide as a reference to set up any other machines you wish to use to manage the cluster.

This details the setup process of a Kubernetes environment in a client node:

1. [Install](../topics/package-manager.md#install-software) the `kubectl` package using your package manager (i.e. `apt`) if it is available in your software repository, **alternatively**, install from source:

   - Download the latest release of `kubectl`:

      ```sh
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      ```

   - **(Optional)** Download the `kubectl` checksum file:

      ```sh
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
      ```

      and validate the downloaded `kubectl` binary:

      ```sh
      echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
      ```

   - Install `kubectl` on the client node:

      ```sh
      sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
      ```

2. [Install Helm](#installing-helm) on the client node.

3. Install both `kubectx` and `kubens` by [installing](../topics/package-manager.md#install-software) the `kubectx` package using your package manager (i.e. `apt`) if it is available in your software repository, **alternatively**, install from source:

   - Clone the source repository to the client node:

      ```sh
      git clone https://github.com/ahmetb/kubectx ~/.kubectx
      ```

   - Install both `kubectx` and `kubens` to the system via symlinks:

      ```sh
      sudo ln -s ~/.kubectx/kubectx ~/.kubectx/kubens /usr/local/bin/
      ```

   - **(Optional)** Install the completion scripts corresponding to the default shell on the client node (i.e. `bash`):

      ```sh
      COMPDIR=$(pkg-config --variable=completionsdir bash-completion)
      ln -sf ~/.kubectx/completion/kubens.bash $COMPDIR/kubens
      ln -sf ~/.kubectx/completion/kubectx.bash $COMPDIR/kubectx
      cat << EOF >> ~/.bashrc


      #kubectx and kubens
      export PATH=~/.kubectx:\$PATH
      EOF
      ```

3. [Install](../topics/package-manager.md#install-software) the `k9s` package using your package manager (i.e. `apt`) if it is available in your software repository, **alternatively**, install from source:

   - Get the latest version number of `k9s`:

      ```sh
      curl -s "https://api.github.com/repos/derailed/k9s/releases/latest" | perl -lne 'print $& if /"tag_name": "v\K[0-9.]+/'
      ```

      Sample output:

      ```
        0.50.9
      ```

   - Download the latest versioned archive of `k9s` (i.e. `0.50.9`) from source:

      ```sh
      curl -Lo /tmp/k9s.tar.gz "https://github.com/derailed/k9s/releases/download/v<version>/k9s_Linux_amd64.tar.gz"
      ```

      For example:

      ```sh
      curl -Lo /tmp/k9s.tar.gz "https://github.com/derailed/k9s/releases/download/v0.50.9/k9s_Linux_amd64.tar.gz"
      ```

   - Unpack the downloaded `k9s` archive:

      ```sh
      mkdir -p /tmp/k9s && tar -xvf /tmp/k9s.tar.gz -C /tmp/k9s
      ```

   - Install `k9s` to the system:

      ```sh
      sudo mv /tmp/k9s/k9s /usr/local/bin/
      ```

4. **(Optional)** [Install](../topics/package-manager.md#install-software) the `pv-migrate` package using your package manager (i.e. `apt`) if it is available in your software repository, **alternatively**, install from source:

   - Get the latest version number of `pv-migrate`:

      ```sh
      curl -s "https://api.github.com/repos/utkuozdemir/pv-migrate/releases/latest" | perl -lne 'print $& if /"tag_name": "v\K[0-9.]+/'
      ```

      Sample output:

      ```
        2.2.1
      ```

   - Download the latest versioned archive of `pv-migrate` (i.e. `2.2.1`) from source:

      ```sh
      curl -Lo /tmp/pv-migrate.tar.gz "https://github.com/utkuozdemir/pv-migrate/releases/download/v<version>/pv-migrate_v<version>_linux_x86_64.tar.gz"
      ```

      For example:

      ```sh
      curl -Lo /tmp/pv-migrate.tar.gz "https://github.com/utkuozdemir/pv-migrate/releases/download/v2.2.1/pv-migrate_v2.2.1_linux_x86_64.tar.gz"
      ```

   - Unpack the downloaded `pv-migrate` archive:

      ```sh
      mkdir -p /tmp/pv-migrate && tar -xvf /tmp/pv-migrate.tar.gz -C /tmp/pv-migrate
      ```

   - Install `pv-migrate` to the system:

      ```sh
      sudo mv /tmp/pv-migrate/pv-migrate /usr/local/bin/
      ```

   - **(Optional)** Install the completion scripts corresponding to the default shell on the client node (i.e. `bash`):

      ```sh
      pv-migrate completion bash > /etc/bash_completion.d/pv-migrate
      ```

5. **(Optional)** [Install](../topics/package-manager.md#install-software) the `df-pv` package using your package manager (i.e. `apt`) if it is available in your software repository, **alternatively**, install from source:

   - Get the latest version number of `df-pv`:

      ```sh
      curl -s "https://api.github.com/repos/yashbhutwala/kubectl-df-pv/releases/latest" | perl -lne 'print $& if /"tag_name": "v\K[0-9.]+/'
      ```

      Sample output:

      ```
        0.3.0
      ```

   - Download the latest versioned archive of `df-pv` (i.e. `0.3.0`) from source:

      ```sh
      curl -Lo /tmp/df-pv.tar.gz "https://github.com/yashbhutwala/kubectl-df-pv/releases/download/v<version>/kubectl-df-pv_v<version>_linux_amd64.tar.gz"
      ```

      For example:

      ```sh
      curl -Lo /tmp/df-pv.tar.gz "https://github.com/yashbhutwala/kubectl-df-pv/releases/download/v0.3.0/kubectl-df-pv_v0.3.0_linux_amd64.tar.gz"
      ```

   - Unpack the downloaded `df-pv` archive:

      ```sh
      mkdir -p /tmp/df-pv && tar -xvf /tmp/df-pv.tar.gz -C /tmp/df-pv
      ```

   - Install `df-pv` to the system:

      ```sh
      sudo mv /tmp/df-pv/df-pv /usr/local/bin/
      ```

### Cluster Setup

This details the setup process of a Kubernetes cluster from a server perspective:

- [Creating a Kubernetes Cluster](#creating-a-kubernetes-cluster)
- [Reverse Proxy](#reverse-proxy)

---

## Kubernetes Cluster

This details topics pertaining the provisioning, setup, management, and use case of a Kubernetes cluster in a homelab environment.

### Creating a Kubernetes Cluster

This details the process of creating and setting up a Kubernetes cluster:

- [RKE2](../topics/rke2.md#creating-a-kubernetes-cluster)

### Reverse Proxy

This details how to set up a reverse proxy on the Kubernetes cluster for serving applications and services publicly:

- [RKE2](../topics/rke2.md#reverse-proxy)

---

## Helm

This details the installation and usage of Helm to deploy and manage Kubernetes applications in your cluster.

### Installing Helm

This details how to install Helm on your system:

- [Helm](../topics/helm.md#installation)

### Helm Chart Repository

This details how to add, update, or remove a Helm chart repository on your system and query available Helm charts to install:

- [Helm](../topics/helm.md#helm-chart-repository)

### Helm Charts

This details how to search or deploy a Helm chart:

- [Helm](../topics/helm.md#helm-charts)

### Helm Values

This details how to get or update Helm values from a Helm chart or deployment (release):

- [Helm](../topics/helm.md#helm-values)
