# FreeDNS

## Description

The site FreeDNS began in 2001 because founder Joshua Anderson wanted to have fun with his domain hobby. He wanted to create a safe environment where other programmers could share domain names with one another at no cost, starting with friends. With that in mind, he launched the site from his domain AFRAID.ORG.

## Directory

- [FreeDNS](#freedns)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Acquiring a Domain](#acquiring-a-domain)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Register a Subdomain](#register-a-subdomain)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)

## References

- [FreeDNS](https://freedns.afraid.org)

---

## Acquiring a Domain

### Description

This details how to acquire a domain name on FreeDNS.

### References

- [Register a new domain](https://developers.cloudflare.com/registrar/get-started/register-domain)

### Steps

1. Free and public domains are available on FreeDNS that can be used without the need to purchase. [Register a subdomain](#register-a-subdomain) for free instead.

---

## Register a Subdomain

### Description

This details how to register a subdomain on FreeDNS.

### References

- [Set up your own VPN at home with Raspberry Pi](https://notthebe.ee/blog/set-up-your-own-vpn-on-raspberry-pi)

### Steps

1. Visit the [Main Menu](https://freedns.afraid.org/menu) page on a web browser. Log into your FreeDNS account if you have not already.

2. From the centre of the main menu, click the **Domain Registry** menu item.

3. From the list of available domains, click the corresponding link of the domain you wish to register a subdomain to (i.e. `example.com`).

4. In the **Add a new subdomain** form, configure as such:

   - Type: Expand the dropdown and select the DNS type (i.e. `A`)
   - Subdomain: Add the name of the subdomain you wish to register (i.e. `mysubdomain`)
   - Domain: Leave this as the domain you had picked earlier (i.e. `example.com`)
   - Destination: Set this to the Public IP address of your home network (i.e. `237.84.2.178`)
   - Wildcard: Leave this unchecked

    Type in the CAPTCHA and click the **Save!** button.
