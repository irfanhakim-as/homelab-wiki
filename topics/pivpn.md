# PiVPN

## Description

PiVPN is a set of shell scripts developed to easily turn your Raspberry Pi™ into a VPN server using two free, open-source protocols.

## Directory

- [PiVPN](#pivpn)
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

- [PiVPN](https://www.pivpn.io)
- [GitHub](https://github.com/pivpn/pivpn)

---

## Setup

### Description

This details how to install and set up a WireGuard VPN server using PiVPN.

### References

- [Build your OWN WireGuard VPN! Here's how](https://youtu.be/5NJ6V8i1Xd8)
- [Ansible 101 - Episode 9 - First 5 min server security with Ansible](https://www.youtube.com/live/gV_16dU7XjM)
- [Set up your own VPN at home with Raspberry Pi](https://notthebe.ee/blog/set-up-your-own-vpn-on-raspberry-pi)

### Installation

This details the installation steps for WireGuard using PiVPN:

1. Prepare and [configure](linux.md#configuration) a Debian or Ubuntu system to host the WireGuard server. This system can be running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container). The following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the system or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule): `51820/udp`

2. **(Optional)** [Set up a domain](../courses/network.md#registering-subdomains) (non-proxied) to be used as the endpoint to your VPN server (i.e. `vpn.example.com`) - while this is technically _optional_, it is highly recommended to set up if you do not have a static public IP, which is very likely in a homelab setting.

3. Ensure that incoming traffic to your home network, through public IP (i.e. `203.0.113.0`) or domain (i.e. `vpn.example.com`), is routed accordingly to your VPN server by setting up [port forwarding](../courses/network.md#port-forwarding):

   - Service name: Set any suitable name that best describes the service or rule (i.e. `vpn`)
   - Device IP address: Enter the private IP address of the WireGuard server (i.e. `192.168.0.106`)
   - Internal port: Enter the port number you have allocated and allowed for the WireGuard server or leave as default (i.e. `51820`)
   - External port: Enter the same port number you had set as the internal port (i.e. `51820`)
   - Protocol: `UDP`
   - Enabled: Toggle or check this box to enable the port forwarding rule

4. Run the PiVPN installation wizard:

    ```sh
    curl -L https://install.pivpn.io | bash
    ```

5. Follow the guided installation process, choosing these options when prompted:

   - Installation mode: `WireGuard`
   - Default wireguard Port: Enter the port number you have allocated and allowed for the WireGuard server or leave as default (i.e. `51820`)
   - DNS Provider: Select one of the available DNS provider (i.e. `Cloudflare`) or pick `Custom` if you wish to add a custom DNS server (i.e. your own [local DNS](../courses/network.md#local-dns) server)
   - Public IP or DNS: Select the `DNS Entry` option to point towards your public WireGuard server endpoint via DNS
     - What is the public DNS name of this Server?: Set this to the domain you had set up and configured for the WireGuard server (i.e. `vpn.example.com`)

6. Reboot the WireGuard server once the installation and setup process is complete.

7. **(Optional)** Go through the [post-installation setup steps](#post-install-setup) to complete the WireGuard server setup.

### Post-Install Setup

This details the post-installation steps of the PiVPN WireGuard server for a complete setup:

1. [Create a client profile](#creating-client-profile) for each client device that needs to connect to the VPN server.

---

## Usage

### Description

This details some common usage steps for a PiVPN VPN server.

### Creating Client Profile

This details the process of creating a VPN client profile for connecting to the VPN server:

1. On the WireGuard server, use the following command to create a profile for a single client:

    ```sh
    pivpn add -n <profile-name>
    ```

    Replace `<profile-name>` with a suitable name for the client profile (i.e. `my-device-1`)

2. Copy over the generated profile to the client system remotely:

    ```sh
    scp ~/configs/<profile-name>.conf <client-user>@<client-host>:~
    ```

    - Replace `<profile-name>` with the name of the client profile (i.e. `my-device-1`)
    - Replace `<client-user>` with the username of the client system (i.e. `pi`)
    - Replace `<client-host>` with the hostname or IP address of the client system (i.e. `192.168.0.101`)

    **Alternatively**, if the client device has a camera installed (i.e. a mobile phone), you can use the following command to generate a QR code of the client profile:

    ```sh
    pivpn qr <profile-name>
    ```

    Replace `<profile-name>` with the name of the client profile (i.e. `my-device-1`)

3. For steps on how to establish a connection to the VPN server using these profiles on the client devices, please refer to [this guide](https://github.com/irfanhakim-as/linux-wiki/blob/master/topics/wireguard.md#connecting-to-wireguard).
