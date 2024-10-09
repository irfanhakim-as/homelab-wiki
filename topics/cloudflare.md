# Cloudflare

## Description

Cloudflare, Inc. is an American company that provides content delivery network services, cloud cybersecurity, DDoS mitigation, wide area network services, reverse proxies, Domain Name Service, and ICANN-accredited domain registration services.

## Directory

- [Cloudflare](#cloudflare)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Buying a Domain](#buying-a-domain)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Cloudflare as Nameserver](#cloudflare-as-nameserver)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)
  - [Register a Subdomain](#register-a-subdomain)
    - [Description](#description-3)
    - [References](#references-3)
    - [Steps](#steps-2)

## References

- [Cloudflare](https://www.cloudflare.com)
- [Wikipedia](https://en.wikipedia.org/wiki/Cloudflare)

---

## Buying a Domain

### Description

This details how to purchase a domain name on Cloudflare.

### References

- [Register a new domain](https://developers.cloudflare.com/registrar/get-started/register-domain)

### Steps

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com) page on a web browser. Log into your Cloudflare account if you have not already.

2. On the left hand side of the dashboard, expand the **Domain Registration** group and select the **Register Domains** menu item.

3. In the **Register Domain** page, use the provided search bar to look for a domain name.

4. From the list of available domains, select a domain name of your choice for purchase by clicking its corresponding **Purchase** button.

5. In the **Complete your registration** form, expand the **Payment option** dropdown and select the duration of your domain registration (i.e. `1 year`).

6. Fill in the rest of the required fields including **Registrant information**, **Payment**, and **Billing address** and click the **Complete purchase** button to complete the order.

---

## Cloudflare as Nameserver

> [!IMPORTANT]  
> This section is not required if your domain was [purchased](#buying-a-domain) on Cloudflare.

### Description

This details how to set Cloudflare as the authoritative DNS nameserver for a domain.

### References

- [Setup](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup)
- [Add your domain to Cloudflare](https://developers.cloudflare.com/learning-paths/get-started/add-domain-to-cf)
- [EXPOSE your home network to the INTERNET!! (it's safe)](https://youtu.be/ey4u7OUAF3c)

### Steps

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com) page on a web browser. Log into your Cloudflare account if you have not already.

2. On the left hand side of the dashboard, select the **Websites** menu item.

3. Click the **Add a Domain** button.

4. Enter your apex domain in the **Enter an existing domain** field and select the **Manually enter DNS records** option. Click **Continue**.

5. When provided with several subscription plans to choose from, scroll down to the bottom of the page and select the **Free** plan and click **Continue**.

6. In the **Review your DNS records** page, we are informed that we will need to set Cloudflare as the nameserver for our domain. Scroll down to the bottom of the page and click **Continue**.

7. Copy the nameserver endpoints supplied by Cloudflare and [update the nameserver of our domain](domain.md#update-authoritative-nameserver) according to our domain provider (i.e. Porkbun, Squarespace, etc.).

8. Once we have configured our domain to use Cloudflare as the nameserver, head back to where we were on Cloudflare and click the **Done, check nameservers** button located at the very bottom of the page.

9. In the **Quick Start Guide** page, click the **Finish later** link under the **Get started** button.

10. In the **Zone Overview** page for your domain, scroll down and click the **Check nameservers** button.

    - You can refresh the page after some time to see if your domain has officially used Cloudflare as its nameservers by seeing a message such as **Great news! Cloudflare is now protecting your site**.
    - You may also receive an email from Cloudflare stating, for example, **Your domain is now active on a Cloudflare Free plan**.

---

## Register a Subdomain

### Description

This details how to register a subdomain on Cloudflare.

### References

- [Create DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/#create-dns-records)

### Steps

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com) page on a web browser. Log into your Cloudflare account if you have not already.

2. On the left hand side of the dashboard, expand the **DNS** group and select the **Records** menu item.

3. In the **DNS Records** page, click the **Add record** button.

4. In the provided form, configure as such:

   - Type: Expand the dropdown and select the DNS type (i.e. `A`)
   - Name: Add the name of the subdomain you wish to register (i.e. `mysubdomain`)
   - IPv4 address: Set this to the Public IP address of your home network (i.e. `237.84.2.178`)
   - Proxy status: Toggle to decide whether the hostname traffic should be proxied through Cloudflare
   - TTL: Expand the dropdown and set the duration for record updates to reach end users (i.e. `Auto`)

    Click the **Save** button.
