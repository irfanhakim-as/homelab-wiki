# Hypervisor

## Description

A hypervisor (also known as a virtual machine monitor, VMM, or virtualizer) is a type of computer software, firmware or hardware that creates and runs virtual machines.

## Directory

- [Hypervisor](#hypervisor)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Installation](#installation)
    - [Installing ESXi](#installing-esxi)
    - [Installing Proxmox](#installing-proxmox)
  - [Updates](#updates)
    - [References](#references-1)
  - [Backups](#backups)
    - [Periodic Config Backups](#periodic-config-backups)
  - [Configuration](#configuration)
    - [Adding Additional Storage](#adding-additional-storage)
    - [Adding a Network Card](#adding-a-network-card)
    - [Hardware Passthrough](#hardware-passthrough)
  - [Migration](#migration)
    - [Migrating to Proxmox](#migrating-to-proxmox)
  - [Clustering](#clustering)
    - [Creating Cluster](#creating-cluster)
    - [Joining Node](#joining-node)
    - [Adding QDevice](#adding-qdevice)
    - [Recommended Configuration](#recommended-configuration)
    - [High Availability (HA)](#high-availability-ha)
  - [Monitoring](#monitoring)
    - [References](#references-2)

## References

- [Hypervisor](https://en.wikipedia.org/wiki/Hypervisor)

---

## Installation

This details the process of setting up a hypervisor including installation and configuration.

### Installing ESXi

- [Acquire License](../topics/esxi.md#acquire-license)
- [Custom ISO](../topics/esxi.md#custom-iso)
- [Installation](../topics/esxi.md#installation)

### Installing Proxmox

- [Setup](../topics/proxmox.md#setup)

---

## Updates

This details the process of performing updates on the hypervisor.

### References

- [Proxmox](../topics/proxmox.md#updates)

---

## Backups

This details the process of backing up our hypervisor.

### Periodic Config Backups

This details how to setup periodic configuration backups on the hypervisor:

- [ESXi](../topics/esxi.md#periodic-config-backups)

---

## Configuration

This details some additional configuration options and use cases for the hypervisor.

### Adding Additional Storage

This topic details how to add an additional storage device to the hypervisor.

- [ESXi](../topics/esxi.md#adding-a-datastore)
- [Proxmox](../topics/proxmox.md#adding-storage)

### Adding a Network Card

This topic details how to add an additional network device to the hypervisor.

- [ESXi](../topics/esxi.md#adding-a-network-card)

### Hardware Passthrough

This details the process of passing through hardware devices to a virtual machine or LXC container on the hypervisor.

- [ESXi: Disk Pass-through](../topics/esxi.md#disk-pass-through)
- [Proxmox: PCIe Passthrough](../topics/proxmox.md#pcie-passthrough)
- [Proxmox: GPU Passthrough to LXC Container](../topics/proxmox.md#gpu-passthrough-to-lxc-container)

---

## Migration

This details the process of migrating a virtual machine from an existing hypervisor to another.

### Migrating to Proxmox

This details the process of migrating a virtual machine from an existing hypervisor to Proxmox:

- [Proxmox](../topics/proxmox.md#migrating-to-proxmox)

---

## Clustering

This details the process of clustering multiple hypervisor nodes together.

### Creating Cluster

This details the process of provisioning a cluster of nodes on a hypervisor.

- [Proxmox](../topics/proxmox.md#creating-cluster)

### Joining Node

This details the steps to add additional node(s) to an existing cluster.

- [Proxmox](../topics/proxmox.md#joining-node)

### Adding QDevice

This details the process of adding a QDevice to achieve and sustain a high availability environment in an even-numbered cluster.

- [Proxmox](../topics/proxmox.md#adding-qdevice)

### Recommended Configuration

This details some recommended configurations for the cluster depending on your setup.

- [Proxmox](../topics/proxmox.md#recommended-configuration)

### High Availability (HA)

This details the steps to set up a high availability environment.

- [Proxmox](../topics/proxmox.md#high-availability-ha)

---

## Monitoring

This details the setup process of a monitoring system on the hypervisor.

### References

- [Proxmox](../topics/proxmox.md#monitoring)
