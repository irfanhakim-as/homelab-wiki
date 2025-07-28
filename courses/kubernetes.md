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
  - [Helm](#helm)
    - [Installing Helm](#installing-helm)
    - [Helm Chart Repository](#helm-chart-repository)
    - [Helm Charts](#helm-charts)
    - [Helm Values](#helm-values)

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

---

## Helm

This details the installation and usage of Helm to deploy and manage Kubernetes applications in your cluster.

### Installing Helm

This details how to install Helm on your system:

- [Helm](../topics/helm.md#installation)

### Helm Chart Repository

This details how to add, update, or remove a Helm chart repository on your system and query available Helm charts to install:

- [Helm](../topics/helm.md#helm-chart-repository)

### Helm Charts

This details how to search or deploy a Helm chart:

- [Helm](../topics/helm.md#helm-charts)

### Helm Values

This details how to get or update Helm values from a Helm chart or deployment (release):

- [Helm](../topics/helm.md#helm-values)
