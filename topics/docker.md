# Docker

## Description

Docker is a set of platform as a service (PaaS) products that use OS-level virtualisation to deliver software in packages called containers.

## Directory

- [Docker](#docker)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Configuration](#configuration)

## References

- [Docker](https://www.docker.com)
- [Wikipedia](https://en.wikipedia.org/wiki/Docker_(software))

---

## Setup

### Description

This details the installation and configuration process of Docker on a Linux system.

### References

- [Install Docker Engine](https://docs.docker.com/engine/install)
- [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
- [ArchWiki](https://wiki.archlinux.org/title/Docker)

### Installation

> [!NOTE]  
> Take note of steps that may be different on specific Linux distributions.

This details how to install Docker on a Linux system.

1. [Install](package-manager.md#install-software) the following package dependencies using your package manager (i.e. `apt`):

    **Debian/Ubuntu**:

   - `ca-certificates`
   - `curl`

    **RHEL**

   - `dnf-plugins-core`

2. Add and set up the Docker repository:

    **Debian**:

    ```sh
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

    **Ubuntu**:

    ```sh
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

    **RHEL**:

    ```sh
    sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
    ```

3. [Update the system](package-manager.md#update-software)'s software repository using your package manager (i.e. `apt`).

4. [Install](package-manager.md#install-software) the latest version of Docker packages using your package manager (i.e. `apt`):

    **Arch Linux**:

    - `docker`
    - `docker-buildx`
    - `docker-compose`

    **Debian/Ubuntu/RHEL**:

    - `docker-ce`
    - `docker-ce-cli`
    - `containerd.io`
    - `docker-buildx-plugin`
    - `docker-compose-plugin`

5. [Start and enable](systemd.md#enable-service) the following Docker services:

   - `docker.service`
   - `containerd.service`

### Configuration

This details how to configure Docker to allow a non-root user to run it.

1. [Create the `docker` group](linux.md#create-group) on the system.

2. [Add the service (non-root) user to the `docker` group](linux.md#add-user-to-group) to give the user root-level privileges for running Docker.

3. Reboot the server to apply the changes.
