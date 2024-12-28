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
    - [Manual](#manual)
    - [Cloudflare DDNS](#cloudflare-ddns)
  - [Dynamic DNS](#dynamic-dns)
    - [Description](#description-4)
    - [References](#references-4)
    - [Get Zone ID](#get-zone-id)
    - [Create API Token](#create-api-token)
    - [Cloudflare DDNS (Helm)](#cloudflare-ddns-helm)

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

2. On the left hand side of the dashboard, select the **Account Home** menu item.

3. On the **Account Home** page, click the **Add a domain** button.

4. Enter your apex domain in the **Enter an existing domain** field and click **Continue**.

5. When provided with several subscription plans to choose from, scroll down to the bottom of the page and select the **Free** plan and click **Continue**.

6. In the **Review your DNS records** page, we are informed that we will need to set Cloudflare as the nameserver for our domain. Scroll down to the bottom of the page and click the **Continue to activation** button.

7. Copy the nameserver endpoints supplied by Cloudflare and [update the nameservers of our domain](dns.md#update-authoritative-nameserver) according to our domain provider (i.e. Porkbun, Squarespace, etc.).

8. Once we have configured our domain to use Cloudflare as the nameserver, head back to where we were on Cloudflare and click the **Check nameservers now** button located at the very bottom of the page.

9. Click the **Continue** button.

10. In the **Overview** page for your domain, you can monitor the progress by scrolling down and clicking the **Check nameservers now** button to check once per hour.

    - You can refresh the page after some time to see if your domain has officially used Cloudflare as its nameservers by seeing a message such as **Great news! Cloudflare is now protecting your site**.
    - You may also receive an email from Cloudflare stating, for example, **Your domain is now active on a Cloudflare Free plan**.

---

## Register a Subdomain

### Description

This details how to register a subdomain on Cloudflare.

### References

- [Create DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/#create-dns-records)

### Manual

This method of registering a subdomain relies on you to manually update the DNS entries with your home network's public IP endpoint:

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com) page on a web browser. Log into your Cloudflare account if you have not already.

2. On the left hand side of the dashboard, select the **Account Home** menu item.

3. On the **Account Home** page, select the link on the name of the domain you wish to register a subdomain to (i.e. `example.com`).

4. On the left hand side of the dashboard, expand the **DNS** group and select the **Records** menu item.

5. In the **DNS Records** page, click the **Add record** button.

6. In the provided form, configure as such:

   - Type: Expand the dropdown and select the DNS type (i.e. `A`)
   - Name: Add the name of the subdomain you wish to register (i.e. `mysubdomain`)
   - IPv4 address: Set this to the public IP address of your home network (i.e. `237.84.2.178`)
   - Proxy status: Toggle to decide whether the hostname traffic should be proxied through Cloudflare
   - TTL: Expand the dropdown and set the duration for record updates to reach end users (i.e. `Auto`)

    Click the **Save** button.

### Cloudflare DDNS

This ensures all of the subdomains that were registered to your domain using this method will have its public IP endpoint automatically updated:

1. Deploy [Cloudflare DDNS on Helm](#cloudflare-ddns-helm) for each zone (domain) if you have not already.

2. To register subdomain(s) to the particular zone, update the release's `cloudflareddns.subdomains` value in its release values file (i.e. `~/values.yaml`).

   - For example:

      ```diff
      - subdomains: []
      + subdomains:
      +   - hostname: "mysubdomain"
      +     proxied: "false"
      ```

      This sample change adds or updates the `mysubdomain` DNS record in the zone specified in the `cloudflareddns.zoneID` value (i.e. `mysubdomain.example.com`).

   - To register more subdomains, simply add more of them to the `cloudflareddns.subdomains` array as such:

      ```yaml
      subdomains:
        - hostname: ""
          proxied: "true"
        - hostname: "mysubdomain"
          proxied: "false"
      ```

      > [!TIP]  
      > Setting an empty `hostname` value adds or updates a DNS record equalling to the apex or root domain of the specified zone (i.e. `example.com`).

3. Deploy the update to the existing `mika/cloudflareddns` release:

    ```sh
    helm -n <namespace> upgrade --install <release-name> mika/cloudflareddns -f ~/values.yaml
    ```

   - Replace `<namespace>` with the namespace where the `mika/cloudflareddns` release was deployed (i.e. `cloudflare`)
   - Replace `<release-name>` with the name of the `mika/cloudflareddns` release (i.e. `example-com`)

4. You may need to kill the release's pod manually for the changes to take effect:

    ```sh
    kubectl -n <namespace> delete pods -l app.kubernetes.io/instance=<release-name>
    ```

5. Verify the deployment is running:

    ```sh
    kubectl -n <namespace> logs deployments/<release-name>-cloudflareddns
    ```

    Sample output indicating the deployment is running and updating DNS records successfully:

    ```
    🕰️ Updating IPv4 (A) records every 300 seconds
    📡 Updating record {'type': 'A', 'name': 'example.com', 'content': '237.84.2.178', 'proxied': True, 'ttl': 300}
    📡 Updating record {'type': 'A', 'name': 'mysubdomain.example.com', 'content': '237.84.2.178', 'proxied': False, 'ttl': 300}
    ```

---

## Dynamic DNS

### Description

This details how to keep DNS records up-to-date dynamically on Cloudflare.

### References

- [Dynamically update DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/managing-dynamic-ip-addresses)
- [Cloudflare DDNS](https://github.com/timothymiller/cloudflare-ddns)

### Get Zone ID

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com) page on a web browser. Log into your Cloudflare account if you have not already.

2. On the left hand side of the dashboard, select the **Account Home** menu item.

3. On the **Account Home** page, select a domain you wish to configure for dynamic DNS (i.e. `example.com`).

4. On the bottom right corner of the dashboard, within the **API** section, copy and make note of the value of the **Zone ID** (i.e. `SEgYD5N77TVkBa2P1m01eryDC4IuajmS`).

### Create API Token

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com) page on a web browser. Log into your Cloudflare account if you have not already.

2. On the left hand side of the dashboard, select the **Account Home** menu item.

3. On the **Account Home** page, select a domain you wish to configure for dynamic DNS (i.e. `example.com`).

4. On the bottom right corner of the dashboard, within the **API** section, click the **Get your API token** link.

5. In the **User API Tokens** page, click the **Create Token** button.

6. In the **Create API Token** page, click the **Get started** button corresponding to the **Create Custom Token** option.

7. In the **Create Custom Token** form, configure the following:

   - Token name: Set a descriptive and concise name for the token (i.e. `LE DNS Validation (example.com)`)
   - Permissions:
     - `Zone`: `Zone`: `Read`
     - `Zone`: `DNS`: `Edit`
   - Zone Resources:
     - `Include`: `Specific zone`: The domain this token should allow access to (i.e. `example.com`)
     - **Alternatively**, make the token applicable to all of your zones (domains) by configuring it as such instead:
       - `Include`: `All zones`
   - TTL: Update the `Start Date` and `End Date` as to define how long the token will stay active (leave empty for indefinite)

    Click the **Continue to summary** button.

8. In the **API token summary** page, review the summary of the new API token and click the **Create Token** button to create the token.

9. Once the API token has been created, copy the token value and keep it somewhere safe. For security reasons, the value of this API token will no longer be shown again on Cloudflare.

### Cloudflare DDNS (Helm)

1. Add the [mika](https://github.com/irfanhakim-as/charts) Helm chart repository to your system if you have not already.

2. Get the latest updates from available Helm chart repositories:

    ```sh
    helm repo update
    ```

3. Download the template values file of the `mika/cloudflareddns` chart to the home directory (i.e. `~/values.yaml`):

    ```sh
    helm get values mika/cloudflareddns > ~/values.yaml
    ```

4. Update the template values file with the required values, including:

   - `cloudflareddns.token`: Set the value to the dedicated Cloudflare [API Token](#create-api-token) previously created
   - `cloudflareddns.zoneID`: Set the value to the [Zone ID](#get-zone-id) of the domain we wish the deployment to manage

5. Deploy the `mika/cloudflareddns` chart:

    ```sh
    helm -n <namespace> upgrade --install <release-name> mika/cloudflareddns -f ~/values.yaml
    ```

   - Replace `<namespace>` with the namespace where the `mika/cloudflareddns` chart should be deployed (i.e. `cloudflare`)
   - Replace `<release-name>` with a unique, descriptive name for the `mika/cloudflareddns` release (i.e. `example-com`)

6. Verify the deployment is running:

    ```sh
    kubectl -n <namespace> logs deployments/<release-name>-cloudflareddns
    ```

    Sample output indicating the deployment is running and working successfully:

    ```
    🕰️ Updating IPv4 (A) records every 300 seconds
    ```

7. To register and update subdomain(s) (i.e. with the latest public IP endpoint) on the release's specified zone, [update the release's list of subdomains](#cloudflare-ddns).
