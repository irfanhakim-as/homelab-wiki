# Hypervisor

## Description

A hypervisor (also known as a virtual machine monitor, VMM, or virtualizer) is a type of computer software, firmware or hardware that creates and runs virtual machines.

## Directory

- [Hypervisor](#hypervisor)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Installation](#installation)
    - [ESXi](#esxi)
    - [Proxmox](#proxmox)
  - [Updates](#updates)
    - [References](#references-1)
  - [Backups](#backups)
    - [References](#references-2)
  - [Configuration](#configuration)
    - [Adding Additional Storage](#adding-additional-storage)
    - [Adding a Network Card](#adding-a-network-card)
    - [PCIe Passthrough](#pcie-passthrough)
  - [Migration](#migration)
    - [References](#references-3)
  - [Clustering](#clustering)
    - [References](#references-4)
  - [Monitoring](#monitoring)
    - [References](#references-5)

## References

- [Hypervisor](https://en.wikipedia.org/wiki/Hypervisor)

---

## Installation

This details the process of setting up a hypervisor including installation and configuration.

### ESXi

- [Acquire License](../topics/esxi.md#acquire-license)
- [Custom ISO](../topics/esxi.md#custom-iso)
- [Installation](../topics/esxi.md#installation)

### Proxmox

- [Setup](../topics/proxmox.md#setup)

---

## Updates

This details the process of performing updates on the hypervisor.

### References

- [Proxmox: Updates](../topics/proxmox.md#updates)

---

## Backups

This details the process of backing up our hypervisor.

### References

- [ESXi: Periodic Config Backups](../topics/esxi.md#periodic-config-backups)

---

## Configuration

This details some configuration options for the hypervisor.

### Adding Additional Storage

This topic details how to add an additional storage device to the hypervisor.

- [ESXi: Adding a Datastore](../topics/esxi.md#adding-a-datastore)

### Adding a Network Card

This topic details how to add an additional network device to the hypervisor.

- [ESXi: Adding a Network Card](../topics/esxi.md#adding-a-network-card)

### PCIe Passthrough

This details the process of passing through PCIe devices to a virtual machine on the hypervisor.

- [ESXi: Disk Pass-through](../topics/esxi.md#disk-pass-through)
- [Proxmox: PCIe Passthrough](../topics/proxmox.md#pcie-passthrough)

---

## Migration

This details the process of migrating a virtual machine from an existing hypervisor to another.

### References

- [Proxmox: Migrating to Proxmox](../topics/proxmox.md#migrating-to-proxmox)

---

## Clustering

This details the process of clustering multiple hypervisor nodes together.

### References

- [Proxmox: Clustering](../topics/proxmox.md#clustering)

---

## Monitoring

This details the setup process of a monitoring system on the hypervisor.

### References

- [Proxmox](../topics/proxmox.md#monitoring)
