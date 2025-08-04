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
  - [Register a Subdomain](#register-a-subdomain)
    - [Description](#description-3)
    - [Manual Subdomain Registration](#manual-subdomain-registration)
    - [Dynamic DNS](#dynamic-dns)
  - [Local DNS](#local-dns)
    - [Description](#description-4)
    - [References](#references-3)

## References

- [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)

---

## Acquiring a Domain

### Description

This details how to acquire a domain name from a domain name registrar.

### References

- [Cloudflare](cloudflare.md#buying-a-domain)
- [FreeDNS](freedns.md#acquiring-a-domain)
- [Porkbun](porkbun.md#buying-a-domain)
- [Squarespace](squarespace.md#buying-a-domain)
- [Name.com](name.com.md#buying-a-domain)

---

## Update Authoritative Nameserver

### Description

This details how to update the authoritative nameserver for a domain.

### References

- [Porkbun](porkbun.md#update-authoritative-nameserver)
- [Squarespace](squarespace.md#update-authoritative-nameserver)
- [Name.com](name.com.md#update-authoritative-nameserver)

### Cloudflare as Nameserver

> [!IMPORTANT]  
> This part of the guide is not required if your domain was either [purchased on Cloudflare](#acquiring-a-domain) or if you have no intention to use Cloudflare as your nameserver.

This details how to set Cloudflare as the authoritative DNS nameserver for a domain:

- [Cloudflare](cloudflare.md#cloudflare-as-nameserver)

---

## Register a Subdomain

> [!IMPORTANT]  
> This section of the guide requires that you have [acquired and set up a domain name](../courses/network.md#setting-up-a-domain) before attempting to register a subdomain.

> [!NOTE]  
> A registered subdomain only points to the public IP endpoint of your home network. Your home network will still need to route traffic to the appropriate service, typically through [reverse proxy](../courses/network.md#reverse-proxy) or manual [port forwarding](../courses/network.md#port-forwarding).

### Description

This details how to register a subdomain on a DNS server manually or dynamically.

### Manual Subdomain Registration

This details how to register a subdomain manually on a DNS server:

- [Cloudflare](cloudflare.md#manual-subdomain-registration)
- [FreeDNS](freedns.md#register-a-subdomain)

### Dynamic DNS

This details how to keep DNS records up-to-date dynamically on a DNS server:

- [Cloudflare](cloudflare.md#dynamic-cloudflare-ddns)
- [FreeDNS](freedns.md#dynamic-dns)

---

## Local DNS

### Description

This details the process of setting up a local DNS server for use with your local servers or devices.

### References

- [Pi-hole](pi-hole.md#setup)
