# Linux

## Description

Linux is both an open-source Unix-like kernel and a generic name for a family of open-source Unix-like operating systems based on the Linux kernel, an operating system kernel first released on September 17, 1991, by Linus Torvalds.

## Directory

- [Linux](#linux)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Installation](#installation)
    - [Description](#description-1)
    - [References](#references-1)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [References](#references-2)
  - [Networking](#networking)
    - [Description](#description-3)
    - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Update Hostname](#update-hostname)
  - [Storage](#storage)
    - [Description](#description-4)
    - [Partition Storage](#partition-storage)
    - [Mount Storage](#mount-storage)
    - [Resize Storage](#resize-storage)
  - [User Management](#user-management)
    - [Description](#description-5)
    - [Create User](#create-user)
    - [Create Group](#create-group)
    - [Add User to Group](#add-user-to-group)
    - [Sudo](#sudo)

## References

- [Linux](https://en.wikipedia.org/wiki/Linux)
- [Linux distribution](https://en.wikipedia.org/wiki/Linux_distribution)

---

## Installation

### Description

This details the installation process for the following Linux distributions as a server on a virtual machine.

### References

- [Arch Linux](arch.md#installation)
- [Debian](debian.md#installation)
- [RHEL](rhel.md#installation)
- [Ubuntu](ubuntu.md#installation)

---

## Configuration

### Description

This details some recommended configuration options for the following Linux distributions as a server.

### References

- [Arch Linux](arch.md#configuration)
- [Debian](debian.md#configuration)
- [RHEL](rhel.md#configuration)
- [Ubuntu](ubuntu.md#configuration)

---

## Networking

### Description

This details the process of updating certain networking configurations for the following Linux distributions.

### Set Static IP and Update DNS

This details the process of setting a static IP address and updating the DNS server on a system:

- [Arch Linux](arch.md#set-static-ip-and-update-dns)
- [Debian](debian.md#set-static-ip-and-update-dns)
- [RHEL](rhel.md#set-static-ip-and-update-dns)
- [Ubuntu](ubuntu.md#set-static-ip-and-update-dns)
- [NetworkManager](networkmanager.md#set-static-ip-and-update-dns)

### Update Hostname

This details the simple process of updating the system's hostname:

- [Arch Linux](arch.md#update-hostname)
- [Debian](debian.md#update-hostname)
- [RHEL](rhel.md#update-hostname)
- [Ubuntu](ubuntu.md#update-hostname)

---

## Storage

### Description

This details the process of updating certain storage related configurations for the following Linux distributions.

### Partition Storage

After attaching a new storage device to the system, this details how to partition the new storage device before mounting it:

- [Debian](debian.md#partition-storage)

### Mount Storage

This details how to mount a storage device on the system:

- [Debian](debian.md#mount-storage)

### Resize Storage

After the _physical_ storage of the system itself has been expanded, the following steps should be performed for the new storage capacity to be reflected on the system:

- [Debian](debian.md#resize-storage)

---

## User Management

### Description

This details topics pertaining to user and group management for the following Linux distributions.

### Create User

This details how to create a service user on the system:

- [Arch Linux](arch.md#create-user)
- [Debian](debian.md#create-user)
- [RHEL](rhel.md#create-user)
- [Ubuntu](ubuntu.md#create-user)

### Create Group

This details how to create a group on the system:

- [Arch Linux](arch.md#create-group)
- [Debian](debian.md#create-group)
- [RHEL](rhel.md#create-group)
- [Ubuntu](ubuntu.md#create-group)

### Add User to Group

This details how to add a user to a specific group:

- [Arch Linux](arch.md#add-user-to-group)
- [Debian](debian.md#add-user-to-group)
- [RHEL](rhel.md#add-user-to-group)
- [Ubuntu](ubuntu.md#add-user-to-group)

### Sudo

This details the process of granting temporary superuser (sudo) privileges to a user:

- [Arch Linux](arch.md#sudo)
- [Debian](debian.md#sudo)
- [RHEL](rhel.md#sudo)
- [Ubuntu](ubuntu.md#sudo)
