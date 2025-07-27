# Network

## Description

A computer network is a set of computers sharing resources located on or provided by network nodes. Computers use common communication protocols over digital interconnections to communicate with each other.

## Directory

- [Network](#network)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Router](#router)
    - [Port Forwarding](#port-forwarding)
  - [DNS](#dns)
    - [Setting Up a Domain](#setting-up-a-domain)
    - [Registering Subdomains](#registering-subdomains)
    - [Disable Email Address Obfuscation](#disable-email-address-obfuscation)
    - [Local DNS](#local-dns)
  - [Reverse Proxy](#reverse-proxy)
    - [References](#references-1)
  - [VPN](#vpn)
    - [References](#references-2)

## References

- [Computer network](https://en.wikipedia.org/wiki/Computer_network)

---

## Router

This section details all topics pertaining to router configuration to ensure proper network management.

### Port Forwarding

This details how to set up port forwarding to route external traffic to a local service within the network:

- [Port Forwarding](../topics/router.md#port-forwarding)

---

## DNS

This section details all topics pertaining DNS in order to enable getting our services published online and for general home use.

### Setting Up a Domain

This details how to acquire and set up a(n apex) domain name for your homelab:

- [Acquiring a Domain](../topics/dns.md#acquiring-a-domain)
- [Cloudflare as Nameserver](../topics/dns.md#cloudflare-as-nameserver)

### Registering Subdomains

This details how to register and manage subdomains for your services:

- [Register a Subdomain](../topics/dns.md#register-a-subdomain)

### Disable Email Address Obfuscation

> [!NOTE]  
> This section of the guide is **Optional** - it is included in this course as a reference should you need it at some point, if you use [Cloudflare as your nameserver](../topics/dns.md#cloudflare-as-nameserver).

This details how to disable the Cloudflare Email Address Obfuscation security feature:

- [Cloudflare](../topics/cloudflare.md#disable-email-address-obfuscation)

### Local DNS

This details the process of setting up a local DNS server for use with your local servers or devices:

- [Local DNS](../topics/dns.md#local-dns)

---

## Reverse Proxy

This details how to set up a reverse proxy for serving applications and services publicly.

### References

- [Kubernetes](kubernetes.md#reverse-proxy)

---

## VPN

This section details all topics pertaining VPN in order to allow us access to our local services securely from outside the home network.

### References

- [Setup](../topics/vpn.md#setup)
