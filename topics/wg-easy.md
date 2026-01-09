# WireGuard Easy

## Description

wg-easy is the easiest way to run WireGuard VPN + Web-based Admin UI.

## Directory

- [WireGuard Easy](#wireguard-easy)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Post-Install Setup](#post-install-setup)
  - [Usage](#usage)
    - [Description](#description-2)
    - [Creating Client Profile](#creating-client-profile)

## References

- [wg-easy](https://wg-easy.github.io/wg-easy/latest)
- [GitHub](https://github.com/wg-easy/wg-easy)

---

## Setup

### Description

This details how to install and set up a WireGuard server in a containerised environment.

### References

- [Basic Installation](https://wg-easy.github.io/wg-easy/latest/examples/tutorials/basic-installation)
- [No Reverse Proxy](https://wg-easy.github.io/wg-easy/latest/examples/tutorials/reverse-proxyless)
- [Optional Configuration](https://wg-easy.github.io/wg-easy/latest/advanced/config/optional-config)
- [Setting up a WireGuard VPN using Docker](https://pimylifeup.com/wireguard-docker)

### Installation

This details the installation steps for WireGuard as a containerised application:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime). The following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the system or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule): `<wireguard-port>/udp` (i.e. `51820/udp`)

2. **(Optional)** [Set up a domain](../courses/network.md#registering-subdomains) (non-proxied) to be used as the endpoint to your VPN server (i.e. `vpn.example.com`) - while this is technically _optional_, it is highly recommended to set up if you do not have a static public IP, which is very likely in a homelab setting.

3. Ensure that incoming traffic to your home network, through public IP (i.e. `203.0.113.0`) or domain (i.e. `vpn.example.com`), is routed accordingly to your VPN server by setting up [port forwarding](../courses/network.md#port-forwarding):

   - Service name: Set any suitable name that best describes the service or rule (i.e. `vpn`)
   - Device IP address: Enter the private IP address of the WireGuard server (i.e. `192.168.0.106`)
   - Internal port: Enter the port number you have allocated and allowed for the WireGuard server or leave as default (i.e. `51820`)
   - External port: Enter the same port number you had set as the internal port (i.e. `51820`)
   - Protocol: `UDP`
   - Enabled: Toggle or check this box to enable the port forwarding rule

4. [Deploy the WireGuard stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/wireguard`): This will be used for at least the config volume directory which specifically requires local storage.

   - **(Optional)** A remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/wireguard`): This can be used for anything else that supports remote storage such as the WireGuard stack's deployment files.

   - A Compose file for the WireGuard stack on the app directory (i.e. `/mnt/smb/docker/wireguard/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        wg-easy:
          container_name: ${APP_CONTAINER}
          image: ghcr.io/wg-easy/wg-easy:${APP_VERSION}
          ports:
            - ${WIREGUARD_PORT}:${WIREGUARD_PORT}/udp
            - ${WEBUI_PORT}:${WEBUI_PORT}/tcp
          environment:
            - PORT=${WEBUI_PORT}
            - INSECURE=${NO_REVERSE_PROXY}
            - DISABLE_IPV6=${DISABLE_IPV6}
            #- HOST=0.0.0.0
          volumes:
            - ${APP_DIR}/config:/etc/wireguard
            - /lib/modules:/lib/modules:ro
          cap_add:
            - NET_ADMIN
            - SYS_MODULE
            #- NET_RAW # ⚠️ Uncomment if using Podman
          sysctls:
            - net.ipv4.ip_forward=1
            - net.ipv4.conf.all.src_valid_mark=1
            - net.ipv6.conf.all.disable_ipv6=0
            - net.ipv6.conf.all.forwarding=1
            - net.ipv6.conf.default.forwarding=1
          networks:
            wg:
              ipv4_address: 10.42.42.42
              ipv6_address: fdcc:ad94:bacf:61a3::2a
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        wg:
          driver: bridge
          enable_ipv6: true
          ipam:
            driver: default
            config:
              - subnet: 10.42.42.0/24
              - subnet: fdcc:ad94:bacf:61a3::/64
      ```

   - An env file for the WireGuard stack on the app directory (i.e. `/mnt/smb/docker/wireguard/.env`):

      ```sh
      SERVICE_NAME=wireguard
      APP_CONTAINER=wg-easy
      APP_VERSION=15.1.0
      APP_DIR=/home/myuser/.local/share/docker/wireguard
      WIREGUARD_PORT=51820
      WEBUI_PORT=51821
      NO_REVERSE_PROXY=false
      DISABLE_IPV6=false
      ```

      Replace all of the values in the env file with your own accordingly.

5. **(Optional)** Go through the [post-installation setup steps](#post-install-setup) to complete the WireGuard server setup.

### Post-Install Setup

This details the post-installation steps of the WireGuard server for a complete setup:

1. Launch the WireGuard server web interface at `http://<wireguard-server-host>:<wireguard-webui-port>` (i.e. `http://192.168.0.106:51821`) on a web browser.

2. For the initial launch of the WireGuard web interface, on the "Welcome" screen, click the **Continue** button.

3. When prompted to register an administrator account, fill in the following information in the provided form:

   - Username: Set a unique, descriptive username for the WireGuard admin user (i.e. `admin`)
   - Password: Set a secure password for the WireGuard admin user
   - Confirm password: Confirm the password you have set for the WireGuard admin user

    Submit the form by clicking the **Create Account** button.

4. When asked if you have an existing setup, click the **No** button to set up a brand new WireGuard server.

5. When prompted for the server host and port, fill in the following information in the provided form:

   - Host: Set this to the domain you had set up and configured for the WireGuard server (i.e. `vpn.example.com`), **alternatively**, click the **Suggest** button and select the `Public` IP address of your home network from the provided dropdown (i.e. `203.0.113.0`)
   - Port: Enter the port number you have allocated and allowed for the WireGuard server or leave as default (i.e. `51820`)

    Submit the form by clicking the **Continue** button.

6. At the end of the initial setup process, click the **Sign In** button to get to the login page of the WireGuard server web interface.

7. If you had [deployed the WireGuard server](#installation) without disabling reverse proxy (i.e. by setting the `NO_REVERSE_PROXY` environment variable to `true`), you will need to set up a [reverse proxy](../courses/network.md#reverse-proxy) for a domain dedicated to the WireGuard server web interface (i.e. `vpn-portal.example.com`), pointing to the WireGuard server host (i.e. `192.168.0.106`) and web UI port (i.e. `51821/tcp`).

8. [Create a client profile](#creating-client-profile) for each client device that needs to connect to the VPN server.

---

## Usage

### Description

This details some common usage steps for a WireGuard server.

### Creating Client Profile

This details the process of creating a VPN client profile for connecting to the VPN server:

1. Launch the WireGuard server web interface on a web browser through its [domain you had set up during post-install](#post-install-setup) (i.e. `https://vpn-portal.example.com`) or at `http://<wireguard-server-host>:<wireguard-webui-port>` (i.e. `http://192.168.0.106:51821`) if you had [deployed the WireGuard server](#installation) with reverse proxy disabled.

2. In the **Login** view of the WireGuard server web interface, fill in the admin user credentials, and click the **Sign In** button.

3. In the home view containing the list of **Clients**, click the corresponding **New** button to create a new client profile.

4. In the provided **New Client** form, configure the following:

   - Name: Set a unique, descriptive name for the client profile (i.e. `my-device-1`)
   - **(Optional)** Expire Date: Set an expiration date for the client profile if you wish to make it temporary (i.e. `31-12-2025`)

    Submit the form by clicking the **Create Client** button.

5. Once you have created the client profile, click the profile's corresponding **Download Configuration** button to download the profile's configuration file, or click its **Show QR Code** button to display a QR code to download the client profile.

6. For steps on how to establish a connection to the VPN server using these profiles on the client devices, please refer to [this guide](https://github.com/irfanhakim-as/linux-wiki/blob/master/topics/wireguard.md#connecting-to-wireguard).
