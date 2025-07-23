# Container

## Description

In software engineering, containerisation is operating-system–level virtualisation or application-level virtualisation over multiple network resources so that software applications can run in isolated user spaces called containers in any cloud or non-cloud environment, regardless of type or vendor.

## Directory

- [Container](#container)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Docker](#docker)
    - [Setting Up Docker](#setting-up-docker)
    - [Setting Up Portainer](#setting-up-portainer)
    - [Docker Usage](#docker-usage)
  - [Linux Containers (LXC)](#linux-containers-lxc)
    - [Create LXC Container](#create-lxc-container)
    - [GPU Passthrough to LXC Container](#gpu-passthrough-to-lxc-container)
    - [Mount SMB share on LXC Container](#mount-smb-share-on-lxc-container)

## References

- [Wikipedia](https://en.wikipedia.org/wiki/Containerization_(computing))

---

## Docker

This section details all topics pertaining to the container runtime, Docker.

### Setting Up Docker

This details the installation and configuration process of Docker on a Linux system:

- [Docker Setup](../topics/docker.md#setup)
- [Installing Docker on OMV](../topics/omv.md#installing-docker)

### Setting Up Portainer

This details how to install and set up a Portainer server in a containerised environment.

- [Setup](../topics/portainer.md#setup)

### Docker Usage

This details some common usage steps for a container runtime solution such as Docker:

- [Docker Compose](../topics/docker.md#docker-compose)
- [Portainer](../topics/portainer.md#usage)
- [Deploying Docker Container on OMV](../topics/omv.md#deploying-docker-container)

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
