# Homelab Wiki

> [!IMPORTANT]  
> This repository is (constantly) a work in progress. For a more desktop-oriented wiki on Linux, check out my [Linux Wiki](https://github.com/irfanhakim-as/linux-wiki).

## Overview

This wiki is a collection of guides and tips mostly focused on starting and maintaining a homelab setup - my hope is that it could help others to get started with their own homelabbing ~~addiction~~ journey, for personal or professional purposes. Do note that this wiki leans towards my own personal homelab infrastructure, based on my own experiences, and is not necessarily perfected to be a general guide.

## Directory

- [Homelab Wiki](#homelab-wiki)
  - [Overview](#overview)
  - [Directory](#directory)
  - [Courses](#courses)
    - [Hardware](#hardware)
    - [Hypervisor](#hypervisor)
    - [Host Deployment Environments](#host-deployment-environments)
    - [Network](#network)
    - [Services](#services)
  - [Contributing](#contributing)

## Courses

This is a list of course-specific guides that compiles [individual topics](topics) found in this repository into individual guides catered to the course:

### [Hardware](courses/hardware.md)

Acquiring the right hardware for the job is the first step in setting up a homelab. This course details the hardware used in my homelab (as a guideline) and any necessary configuration steps in setting them up.

Topics include, but (possibly) not limited to:

- [Primary Server](courses/hardware.md#primary-server)
- [Mini Server](courses/hardware.md#mini-server)

### [Hypervisor](courses/hypervisor.md)

At the heart of most homelabs, is a _primary_ server node running a hypervisor. Hypervisors allow for the creation and management of [virtual machines](courses/vm.md), among other things, which are then used to host your applications and services. This course covers the installation, configuration, and management of your chosen hypervisor software that enables you to run multiple systems on a single physical machine.

Topics include, but (possibly) not limited to:

- [ESXi](topics/esxi.md)
- [Proxmox](topics/proxmox.md)

### Host Deployment Environments

Once you have got your hypervisor running, you need to decide how to actually deploy and run your applications. This course covers the different environments and methods available for hosting your services.

Topics include, but (possibly) not limited to:

- [Virtual Machines (VM)](courses/vm.md) and [Linux Containers (LXC)](courses/container.md#linux-containers-lxc)
- [Containers](courses/container.md) including [Docker](topics/docker.md) and [Podman](topics/podman.md)
- [Kubernetes](courses/kubernetes.md) including [RKE2](topics/rke2.md)

### [Network](courses/network.md)

Networking is a vital component of any functional homelab. This course covers the networking services and configurations needed to ensure proper connectivity and secure access to your hosted services, locally or publicly.

Topics include, but (possibly) not limited to:

- [DNS](topics/dns.md) including [Cloudflare](topics/cloudflare.md), [FreeDNS](topics/freedns.md), and [Pi-hole](topics/pi-hole.md)
- [Reverse Proxy](courses/network.md#reverse-proxy)
- [VPN](topics/vpn.md) including [WireGuard](topics/wireguard.md)

### Services

Last but not least, once your homelab is all ready and set up, it is time to _reap the rewards_ by utilising your setup! This course covers the plethora of applications and services that you could host on your homelab, and how to set them up.

Topics include, but (possibly) not limited to:

- [Database](topics/database.md) including [MariaDB](topics/mariadb.md), [MySQL](topics/mysql.md), and [PostgreSQL](topics/postgresql.md)
- NASes including [TrueNAS](topics/truenas.md) (TODO) and [OpenMediaVault (OMV)](topics/omv.md)
- [Home Assistant](topics/home-assistant.md)
- [Immich](topics/immich.md)
- [Jellyfin](topics/jellyfin.md)
- [ErsatzTV](topics/ersatztv.md)
- [Forgejo](topics/forgejo.md)

## Contributing

These guides take up a huge chunk of my time to research, setup, test, document, and the most painful of all... _organise_.

If you appreciate what I do, here's how you could contribute:

- [Sponsor me!](https://github.com/sponsors/irfanhakim-as) Even a single cent to a _stranger_ like me, would be far too generous.
- [Report an issue](https://github.com/irfanhakim-as/homelab-wiki/issues), any issue, including inaccurate instructions, typos, dead or missing links, incomplete guides, etc.
- [Create a pull request](https://github.com/irfanhakim-as/homelab-wiki/pulls) for any type of corrections, improvements to existing guides, or new guides to add.
