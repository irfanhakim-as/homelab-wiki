# Domain Name System (DNS)

## Description

The Domain Name System (DNS) is a hierarchical and distributed name service that provides a naming system for computers, services, and other resources on the Internet or other Internet Protocol (IP) networks.

## Directory

- [Domain Name System (DNS)](#domain-name-system-dns)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Acquiring a Domain](#acquiring-a-domain)
    - [Description](#description-1)
    - [References](#references-1)
  - [Update Authoritative Nameserver](#update-authoritative-nameserver)
    - [Description](#description-2)
    - [References](#references-2)
  - [Cloudflare as Nameserver](#cloudflare-as-nameserver)
    - [Description](#description-3)
  - [Register a Subdomain](#register-a-subdomain)
    - [Description](#description-4)
    - [References](#references-3)
  - [Dynamic DNS](#dynamic-dns)
    - [Description](#description-5)
    - [References](#references-4)
  - [Local DNS](#local-dns)
    - [Description](#description-6)
    - [References](#references-5)

## References

- [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)

---

## Acquiring a Domain

### Description

This details how to acquire a domain name from a domain name registrar.

### References

- [Cloudflare: Buying a Domain](cloudflare.md#buying-a-domain)
- [FreeDNS: Acquiring a Domain](freedns.md#acquiring-a-domain)
- [Porkbun: Buying a Domain](porkbun.md#buying-a-domain)
- [Squarespace: Buying a Domain](squarespace.md#buying-a-domain)

---

## Update Authoritative Nameserver

### Description

This details how to update the authoritative nameserver for a domain.

### References

- [Porkbun: Update Authoritative Nameserver](porkbun.md#update-authoritative-nameserver)
- [Squarespace: Update Authoritative Nameserver](squarespace.md#update-authoritative-nameserver)

---

## [Cloudflare as Nameserver](cloudflare.md#cloudflare-as-nameserver)

> [!IMPORTANT]  
> This section is not required if your domain was [purchased](#acquiring-a-domain) on Cloudflare or if you have no intention to use Cloudflare as your nameserver.

### Description

This details how to set Cloudflare as the authoritative DNS nameserver for a domain.

---

## Register a Subdomain

### Description

This details how to register a subdomain on a DNS server.

### References

- [Cloudflare: Register a Subdomain](cloudflare.md#register-a-subdomain)
- [FreeDNS: Register a Subdomain](freedns.md#register-a-subdomain)

---

## Dynamic DNS

### Description

This details how to keep DNS records up-to-date dynamically on a DNS server.

### References

- [Cloudflare: Dynamic DNS](cloudflare.md#dynamic-dns)
- [FreeDNS: Dynamic DNS](freedns.md#dynamic-dns)

---

## Local DNS

### Description

This details the process of setting up a local DNS server for use with your local servers or devices.

### References

- [Pi-hole: Setup](pi-hole.md#setup)
