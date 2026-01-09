# Container

## Description

In software engineering, containerisation is operating-system–level virtualisation or application-level virtualisation over multiple network resources so that software applications can run in isolated user spaces called containers in any cloud or non-cloud environment, regardless of type or vendor.

## Directory

- [Container](#container)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Container Runtimes](#container-runtimes)
    - [Setting Up a Container Runtime](#setting-up-a-container-runtime)
    - [Setting Up a Container Management Platform](#setting-up-a-container-management-platform)
    - [Container Runtime Usage](#container-runtime-usage)
  - [Linux Containers (LXC)](#linux-containers-lxc)
    - [Create LXC Container](#create-lxc-container)
    - [GPU Passthrough to LXC Container](#gpu-passthrough-to-lxc-container)
    - [Mount SMB share on LXC Container](#mount-smb-share-on-lxc-container)

## References

- [Wikipedia](https://en.wikipedia.org/wiki/Containerization_(computing))

---

## Container Runtimes

> [!TIP]  
> In general, services that could be deployed with Docker (Compose) could also be deployed with Podman (Compose) and vice versa.

This section details all topics pertaining to [Open Container Initiative (OCI)](https://opencontainers.org/about/overview) container runtimes such as Docker and Podman.

### Setting Up a Container Runtime

This details the installation and configuration process of container runtimes on a Linux system:

- [Docker](../topics/docker.md#setup)
- [Docker on OpenMediaVault (OMV)](../topics/omv.md#installing-docker)
- [Podman](../topics/podman.md#setup)

### Setting Up a Container Management Platform

This details how to install and set up a container management platform as a means to deploy and manage OCI containers:

- [Portainer](../topics/portainer.md#setup)

### Container Runtime Usage

This details common usage steps for container runtime solutions:

- [Docker Compose](../topics/docker.md#docker-compose)
- [Docker on OpenMediaVault (OMV)](../topics/omv.md#deploying-docker-container)
- [Podman Compose](../topics/podman.md#podman-compose)
- [Portainer](../topics/portainer.md#usage)

---

## Linux Containers (LXC)

This details topics pertaining to Linux Containers (LXC) on Proxmox.

### Create LXC Container

This details how to create an LXC Container from scratch as a template, or from a Container Template:

- [Proxmox: Create LXC Container Template](../topics/proxmox.md#create-lxc-container-template)
- [Proxmox: Create LXC Container from Container Template](../topics/proxmox.md#create-lxc-container-from-container-template)

### GPU Passthrough to LXC Container

This details how to passthrough and share a video device (i.e. GPU or iGPU) from the host system to one or more LXC Container(s):

- [Proxmox](../topics/proxmox.md#gpu-passthrough-to-lxc-container)

### Mount SMB share on LXC Container

This details how to mount an SMB share from the host system on an unprivileged LXC Container:

- [Proxmox](../topics/proxmox.md#mount-smb-share-on-lxc-container)
