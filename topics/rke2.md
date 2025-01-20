# RKE2

## Description

RKE2, also known as RKE Government, is Rancher's next-generation Kubernetes distribution. It is a fully conformant Kubernetes distribution that focuses on security and compliance within the U.S. Federal Government sector.

## Directory

- [RKE2](#rke2)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Creating a Kubernetes Cluster](#creating-a-kubernetes-cluster)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)

## References

- [RKE2](https://docs.rke2.io)

---

## Creating a Kubernetes Cluster

### Description

This details the process of creating and setting up a Kubernetes cluster based on RKE2.

### References

- [Orked](https://github.com/irfanhakim-as/orked)
- [Quick Start](https://docs.rke2.io/install/quickstart)

### Steps

1. [Provision](../courses/vm.md#creating-a-virtual-machine-from-a-template) the required number of nodes as the Master node, Worker node, and **optionally**, the Login node.

    Refer to [Orked](https://github.com/irfanhakim-as/orked)'s [prerequisites](https://github.com/irfanhakim-as/orked#hardware) and [hardware requirements](https://github.com/irfanhakim-as/orked#hardware-requirements) for the exact details.

    > [!TIP]  
    > It is highly recommended that you [create a VM template](../courses/vm.md#creating-a-virtual-machine-template) out of each of these categories of nodes for ease of reproducibility and scaling in the future.

2. Follow the detailed steps laid out in the Orked [installation guide](https://github.com/irfanhakim-as/orked#installation) to setup a RKE2-based Kubernetes cluster from the ground up using a collection of scripts.
