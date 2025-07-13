# Homelab Wiki

> [!IMPORTANT]  
> This repository is a work in progress. For a more desktop-oriented wiki on Linux, check out my [Linux Wiki](https://github.com/irfanhakim-as/linux-wiki).

## Overview

This wiki is a collection of guides and tips mostly focused on Homelabs, and is primarily used for my own reference. It is also used to document my own personal infrastructure, and is not necessarily perfected to be a general guide.

## Directory

- [Homelab Wiki](#homelab-wiki)
  - [Overview](#overview)
  - [Directory](#directory)
  - [Courses](#courses)
  - [Topics](#topics)
  - [TODO](#todo)

## Courses

This is a list of course-specific guides that compiles individual topics in this repository into individual guides catered to the course:

- [Hardware](./courses/hardware.md)
- [Hypervisor](./courses/hypervisor.md)
- [Virtual Machines (VM)](./courses/vm.md)
- [Containers](./courses/container.md)
- [Kubernetes](./courses/kubernetes.md)
- [Network](./courses/network.md)
- [Services](./courses/#)

## Topics

Check out the [topics](./topics/) directory for individual guides and tips.

## TODO

1. [Hardware](courses/hardware.md): Hardware setup as the base of the homelab
2. [Hypervisor](courses/hypervisor.md): The primary core of the homelab used for provisioning VMs
   1. [Proxmox](topics/proxmox.md)
   2. [ESXi](topics/esxi.md)
3. [VM](courses/vm.md): The base of all services and infrastructure hosted in the homelab
   1. Creation and management per [hypervisor](courses/hypervisor.md)
   2. Guest OS:
      1. [Linux](topics/linux.md):
         1. [Arch Linux](topics/arch.md)
         2. [Debian](topics/debian.md)
         3. [Ubuntu](topics/ubuntu.md)
         4. [RHEL](topics/rhel.md)
4. [Container](courses/container.md): The alternative way of hosting services
    1. [LXC](courses/container.md#linux-containers-lxc)
    2. [Docker](topics/docker.md)
    3. Podman
    4. [Portainer](topics/portainer.md)
5. [Kubernetes](courses/kubernetes.md): The secondary core of the homelab used for deploying containerised services
   1. [RKE2](topics/rke2.md)
   2. Rancher
   3. [Helm](topics/helm.md)
6. [Network](courses/network.md): Networking setup that powers the homelab (i.e. serves services hosted on VMs and Kubernetes cluster)
   1. [Router](topics/router.md):
      1. [Port forwarding](topics/router.md#port-forwarding)
   2. [DNS](topics/dns.md):
      1. [Squarespace](topics/squarespace.md) (Google)
      2. [Cloudflare](topics/cloudflare.md): Cloudflare DDNS, Cloudflare Tunnel
      3. [Pi-hole](topics/pi-hole.md)
      4. [Porkbun](topics/porkbun.md)
      5. [FreeDNS](topics/freedns.md)
      6. [Name.com](topics/name.com.md)
      7. [Pi-hole](topics/pi-hole.md)
   3. [VPN](topics/vpn.md):
      1. [WireGuard](topics/wireguard.md)
7. Services: Selection of VM or container based services that can be hosted in the homelab
   1. NAS (VM):
      1. TrueNAS
      2. [OpenMediaVault](topics/omv.md) ([Raspberry Pi](topics/raspberry-pi.md))
   2. [Database](topics/database.md) (Container):
      1. [MariaDB](topics/mariadb.md)
      2. [PostgreSQL](topics/postgresql.md)
      3. [MySQL](topics/mysql.md)
   3. Others:
      1. [Home Assistant](topics/home-assistant.md) (Container)
      2. Jellyfin (Container)
      3. ErsatzTV (Container)
      4. [Immich](topics/immich.md) (Container)
