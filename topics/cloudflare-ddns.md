# Cloudflare DDNS

## Description

Cloudflare DDNS is a dynamic DNS updater that automatically keeps DNS records in Cloudflare up-to-date with the current public IP address of a host.

## Directory

- [Cloudflare DDNS](#cloudflare-ddns)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
      - [Install Cloudflare DDNS on Docker](#install-cloudflare-ddns-on-docker)
      - [Install Cloudflare DDNS on Helm](#install-cloudflare-ddns-on-helm)
    - [Post-Install Setup](#post-install-setup)
      - [Cloudflare DDNS Post-Install Setup on Docker](#cloudflare-ddns-post-install-setup-on-docker)
      - [Cloudflare DDNS Post-Install Setup on Helm](#cloudflare-ddns-post-install-setup-on-helm)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [References](#references-2)
    - [Add Domain to Cloudflare DDNS](#add-domain-to-cloudflare-ddns)
      - [Add Domain to Cloudflare DDNS on Docker](#add-domain-to-cloudflare-ddns-on-docker)
      - [Add Domain to Cloudflare DDNS on Helm](#add-domain-to-cloudflare-ddns-on-helm)

## References

- [Cloudflare DDNS](https://github.com/timothymiller/cloudflare-ddns)

---

## Setup

### Description

This details how to deploy Cloudflare DDNS in a containerised environment.

### References

- [Cloudflare DDNS](https://github.com/irfanhakim-as/cloudflare-ddns)
- [moekai/cloudflareddns](https://charts.moekai.com/moekai/cloudflareddns)

### Installation

#### Install Cloudflare DDNS on Docker

This details the installation steps for Cloudflare DDNS as a containerised application using Docker:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime). The following considerations should be noted:

   - Get the [Zone ID](cloudflare.md#get-zone-id) and [API Token](cloudflare.md#create-api-token) for the Cloudflare zone (domain) you wish to manage if you have not already done so. When creating the API token, configure the following permissions:

     - `Zone`: `Zone`: `Read`
     - `Zone`: `DNS`: `Edit`

2. [Deploy the Cloudflare DDNS stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - An app directory, on local or remote storage (i.e. `/home/myuser/.local/share/docker/cloudflareddns`): This will be used for the configuration file.

   - A Compose file for the Cloudflare DDNS stack on the app directory (i.e. `/home/myuser/.local/share/docker/cloudflareddns/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        cloudflareddns:
          container_name: ${APP_CONTAINER}
          image: ghcr.io/irfanhakim-as/cloudflare-ddns:${APP_VERSION}
          environment:
            - CONFIG_PATH=/opt/cloudflare-ddns
          volumes:
            - ${APP_DIR}/config.json:/opt/cloudflare-ddns/config.json:ro
          networks:
            - default
          restart: always
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

   - An env file for the Cloudflare DDNS stack on the app directory (i.e. `/home/myuser/.local/share/docker/cloudflareddns/.env`):

      ```sh
      SERVICE_NAME=cloudflareddns
      APP_CONTAINER=cloudflare-ddns
      APP_VERSION=1.0.3-master-r1
      APP_DIR=/home/myuser/.local/share/docker/cloudflareddns
      ```

      Replace all of the values in the env file with your own accordingly.

   - A `config.json` configuration file on the app directory (i.e. `/home/myuser/.local/share/docker/cloudflareddns/config.json`):

      ```json
      {
        "cloudflare": [
          {
            "authentication": {
              "api_token": "<cloudflare-api-token>"
            },
            "zone_id": "<cloudflare-zone-id>",
            "subdomains": []
          }
        ],
        "a": true,
        "aaaa": false,
        "purgeUnknownRecords": false,
        "ttl": 300
      }
      ```

      For example, for a zone `example.com`:

      ```json
      {
        "cloudflare": [
          {
            "authentication": {
              "api_token": "Na9E7VEY58COhA03l1ytm1r70u7jBsf8bNqh5AlZ"
            },
            "zone_id": "71fovu74p100z856k795umzl32h3240p",
            "subdomains": []
          }
        ],
        "a": true,
        "aaaa": false,
        "purgeUnknownRecords": false,
        "ttl": 300
      }
      ```

      The key configuration options are:

      - `authentication.api_token`: The Cloudflare [API Token](cloudflare.md#create-api-token) for the zone.
      - `zone_id`: The [Zone ID](cloudflare.md#get-zone-id) of the domain to manage.
      - `subdomains`: The list of subdomains for the deployment to manage. Leave as an empty list for now, or [configure it accordingly](#add-domain-to-cloudflare-ddns-on-docker).
      - `a`: Set to `true` to update A (IPv4) records.
      - `aaaa`: Set to `true` to update AAAA (IPv6) records.
      - `purgeUnknownRecords`: Set to `true` to remove DNS records not defined in this configuration.
      - `ttl`: How long DNS records are cached, in seconds.

3. Go through the [post-installation setup steps](#cloudflare-ddns-post-install-setup-on-docker) to complete the Cloudflare DDNS setup.

#### Install Cloudflare DDNS on Helm

This details the installation steps for Cloudflare DDNS on a Kubernetes cluster using Helm:

1. Ensure Helm is [installed](helm.md#installation) on your system. The following considerations should be noted:

   - Get the [Zone ID](cloudflare.md#get-zone-id) and [API Token](cloudflare.md#create-api-token) for the Cloudflare zone (domain) you wish to manage if you have not already done so. When creating the API token, configure the following permissions:

     - `Zone`: `Zone`: `Read`
     - `Zone`: `DNS`: `Edit`

2. Add the Moekai [Helm chart repository](helm.md#adding-chart-repository) to your system:

   - Repository name: `moekai`
   - Repository source: `https://charts.moekai.com`

3. Get the [Helm values file](helm.md#get-helm-chart-values) as a `values.yaml` file for the following chart:

   - Repository: `moekai`
   - Chart: `cloudflareddns`

4. [Prepare the values file](helm.md#update-helm-values) with the following configuration considerations:

   - `cloudflareddns.token`: Set the value to the dedicated Cloudflare [API Token](cloudflare.md#create-api-token) previously created.

      ```yaml
      cloudflareddns:
        token: "<cloudflare-api-token>"
      ```

      For example, if the generated API token is `Na9E7VEY58COhA03l1ytm1r70u7jBsf8bNqh5AlZ`:

      ```yaml
      cloudflareddns:
        token: "Na9E7VEY58COhA03l1ytm1r70u7jBsf8bNqh5AlZ"
      ```

   - `cloudflareddns.zoneID`: Set the value to the [Zone ID](cloudflare.md#get-zone-id) of the domain we wish the deployment to manage.

      ```yaml
      cloudflareddns:
        zoneID: "<cloudflare-zone-id>"
      ```

      For example, if the domain's Zone ID is `71fovu74p100z856k795umzl32h3240p`:

      ```yaml
      cloudflareddns:
        zoneID: "71fovu74p100z856k795umzl32h3240p"
      ```

   - **(Optional)** `cloudflareddns.subdomains`: The list of subdomains for the deployment to manage. Leave as an empty list for now, or [configure it accordingly](#add-domain-to-cloudflare-ddns-on-helm).

      ```yaml
      cloudflareddns:
        subdomains: []
      ```

5. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you had prepared with the following recommended options:

   - Namespace: `cloudflare` (include the flag that creates the namespace if it does not exist)
   - Release: A descriptive name for this Cloudflare DDNS deployment (i.e. `example-com`)
   - Repository: `moekai`
   - Chart: `cloudflareddns`

6. Go through the [post-installation setup steps](#cloudflare-ddns-post-install-setup-on-helm) to complete the Cloudflare DDNS setup.

7. **(Optional)** Keep the Helm release values file (i.e. `values.yaml`) for future use (i.e. for updates).

### Post-Install Setup

#### Cloudflare DDNS Post-Install Setup on Docker

> [!NOTE]
> This guide assumes that you have [installed Cloudflare DDNS](#install-cloudflare-ddns-on-docker) as a containerised application.

This details the post-installation steps of Cloudflare DDNS for a complete Docker setup:

1. Verify the deployment is running:

    ```sh
    docker logs <container>
    ```

    For example:

    ```sh
    docker logs cloudflare-ddns
    ```

    Sample output indicating the deployment is running:

    ```
      🕰️ Updating IPv4 (A) records every 300 seconds
    ```

2. [Add the subdomains](#add-domain-to-cloudflare-ddns-on-docker) you wish the deployment to manage.

#### Cloudflare DDNS Post-Install Setup on Helm

> [!NOTE]
> This guide assumes that you have [installed Cloudflare DDNS](#install-cloudflare-ddns-on-helm) on a Kubernetes cluster using Helm.

This details the post-installation steps of Cloudflare DDNS for a complete Helm setup:

1. Verify the deployment is running:

    ```sh
    kubectl --context <cluster> --namespace <namespace> logs deployments/<release>-cloudflareddns
    ```

    For example:

    ```sh
    kubectl --context my-cluster --namespace cloudflare logs deployments/example-com-cloudflareddns
    ```

    Sample output indicating the deployment is running:

    ```
      🕰️ Updating IPv4 (A) records every 300 seconds
    ```

2. [Add the subdomains](#add-domain-to-cloudflare-ddns-on-helm) you wish the deployment to manage.

---

## Configuration

### Description

This details how to configure Cloudflare DDNS after it has been deployed.

### References

- [Dynamically update DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/managing-dynamic-ip-addresses)

### Add Domain to Cloudflare DDNS

#### Add Domain to Cloudflare DDNS on Docker

> [!NOTE]
> This guide assumes that you have [installed Cloudflare DDNS](#install-cloudflare-ddns-on-docker) as a containerised application.

This details how to add subdomains for Cloudflare DDNS to manage on a Docker deployment:

1. Update the `subdomains` list in the `config.json` configuration file on the app directory (i.e. `/home/myuser/.local/share/docker/cloudflareddns/config.json`):

   - To add a subdomain, append a new entry to the `subdomains` list:

      ```diff
        "subdomains": [
      +   {
      +     "name": "<subdomain>",
      +     "proxied": false
      +   }
        ]
      ```

      For example, to add `mysubdomain.example.com`:

      ```diff
        "subdomains": [
      +   {
      +     "name": "mysubdomain",
      +     "proxied": false
      +   }
        ]
      ```

   - To add more subdomains, simply append additional entries to the list:

      ```json
      "subdomains": [
        {
          "name": "mysubdomain",
          "proxied": false
        },
        {
          "name": "",
          "proxied": false
        }
      ]
      ```

      Setting an empty `name` value adds or updates a DNS record equalling to the apex or root domain of the specified zone (i.e. `example.com` as opposed to something like `mysubdomain.example.com`).

2. Restart the deployment for the changes to take effect:

    ```sh
    docker restart <container>
    ```

    For example:

    ```sh
    docker restart cloudflare-ddns
    ```

3. Verify the deployment is updating the intended DNS records:

    ```sh
    docker logs <container>
    ```

    For example:

    ```sh
    docker logs cloudflare-ddns
    ```

    Sample output:

    ```
      🕰️ Updating IPv4 (A) records every 300 seconds
      📡 Updating record {'type': 'A', 'name': 'mysubdomain.example.com', 'content': '203.0.113.0', 'proxied': False, 'ttl': 300}
      📡 Updating record {'type': 'A', 'name': 'example.com', 'content': '203.0.113.0', 'proxied': False, 'ttl': 300}
    ```

#### Add Domain to Cloudflare DDNS on Helm

> [!NOTE]
> This guide assumes that you have [installed Cloudflare DDNS](#install-cloudflare-ddns-on-helm) on a Kubernetes cluster using Helm.

This details how to add subdomains for Cloudflare DDNS to manage on a Helm deployment:

1. [Update the values file](helm.md#update-helm-values) (i.e. `values.yaml`) of the existing Helm release (i.e. `example-com`) you have deployed:

   - To add subdomain(s) to the particular zone, update the `cloudflareddns.subdomains` configuration as such:

      ```diff
        cloudflareddns:
      -   subdomains: []
      +   subdomains:
      +     - hostname: "mysubdomain"
      +       proxied: "false"
      ```

      This sample change adds or updates the `mysubdomain` DNS record in the zone specified in the `cloudflareddns.zoneID` value (i.e. `mysubdomain.example.com`).

   - To add more subdomains, simply append them to the `cloudflareddns.subdomains` list:

      ```yaml
      cloudflareddns:
        subdomains:
          - hostname: "mysubdomain"
            proxied: "false"
          - hostname: ""
            proxied: "false"
      ```

      Setting an empty `hostname` value adds or updates a DNS record equalling to the apex or root domain of the specified zone (i.e. `example.com` as opposed to something like `mysubdomain.example.com`).

2. [Redeploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) to apply the updated values file with the following specifications:

   - Namespace: The namespace where the existing Cloudflare DDNS Helm release was deployed (i.e. `cloudflare`)
   - Release: The name of the existing Cloudflare DDNS Helm release (i.e. `example-com`)
   - Repository: `moekai`
   - Chart: `cloudflareddns`

3. You may need to kill the existing release's pod manually for the changes to take effect:

    ```sh
    kubectl --context <cluster> --namespace <namespace> delete pods -l app.kubernetes.io/instance=<release>
    ```

    For example:

    ```sh
    kubectl --context my-cluster --namespace cloudflare delete pods -l app.kubernetes.io/instance=example-com
    ```

4. Verify the deployment is updating the intended DNS records:

    ```sh
    kubectl --context <cluster> --namespace <namespace> logs deployments/<release>-cloudflareddns
    ```

    For example:

    ```sh
    kubectl --context my-cluster --namespace cloudflare logs deployments/example-com-cloudflareddns
    ```

    Sample output:

    ```
      🕰️ Updating IPv4 (A) records every 300 seconds
      📡 Updating record {'type': 'A', 'name': 'mysubdomain.example.com', 'content': '203.0.113.0', 'proxied': False, 'ttl': 300}
      📡 Updating record {'type': 'A', 'name': 'example.com', 'content': '203.0.113.0', 'proxied': False, 'ttl': 300}
    ```
