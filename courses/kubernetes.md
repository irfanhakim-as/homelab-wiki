# Kubernetes

## Description

Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications.

## Directory

- [Kubernetes](#kubernetes)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Kubernetes Cluster](#kubernetes-cluster)
    - [Creating a Kubernetes Cluster](#creating-a-kubernetes-cluster)
    - [Reverse Proxy](#reverse-proxy)
    - [Networking Setup](#networking-setup)
  - [Helm](#helm)
    - [References](#references-1)

## References

- [Kubernetes](https://kubernetes.io)

---

## Kubernetes Cluster

This details topics pertaining the provisioning, setup, management, and use case of a Kubernetes cluster in a homelab environment.

### Creating a Kubernetes Cluster

This details the process of creating and setting up a Kubernetes cluster:

- [RKE2](../topics/rke2.md#creating-a-kubernetes-cluster)

### Reverse Proxy

This details how to set up a reverse proxy on the Kubernetes cluster for serving applications and services publicly:

- [RKE2](../topics/rke2.md#reverse-proxy)

### Networking Setup

This details a possible networking setup to enable public access to hosted services on your Kubernetes cluster:

- [Orked](https://github.com/irfanhakim-as/orked/blob/master/README.md#networking-setup)

---

## Helm

This details the installation and usage of Helm to deploy and manage Kubernetes applications in your cluster.

### References

- [Installation](../topics/helm.md#installation)
- [Helm Chart Repository](../topics/helm.md#helm-chart-repository)
- [Helm Charts](../topics/helm.md#helm-charts)
- [Helm Values](../topics/helm.md#helm-values)
