# WireGuard

## Description

WireGuard is an extremely simple yet fast and modern VPN that utilises state-of-the-art cryptography.

## Directory

- [WireGuard](#wireguard)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Client Profile](#client-profile)

## References

- [WireGuard](https://www.wireguard.com)

---

## Setup

### Description

This details the process of setting up a WireGuard server.

### References

- [PiVPN](https://pivpn.io)
- [Build your OWN WireGuard VPN! Here's how](https://youtu.be/5NJ6V8i1Xd8)
- [Ansible 101 - Episode 9 - First 5 min server security with Ansible](https://www.youtube.com/live/gV_16dU7XjM?feature=share)
- [Long KFluff – Connecting to WireGuard VPN from Plasma NetworkManager (plasma-nm)](https://rabbitictranslator.com/wireguard-plasmanm/#plasma-gui)
- [Set up your own VPN at home with Raspberry Pi](https://notthebe.ee/blog/set-up-your-own-vpn-on-raspberry-pi)

### Installation

1. Prepare and [configure](linux.md#configuration) a Debian or Ubuntu system to host the WireGuard server. This system can be a bare metal device (i.e. Raspberry Pi) or a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template).

2. [Set up a domain](dns.md#register-a-subdomain) to be used as the endpoint to your VPN server (i.e. `vpn.example.com`).

3. While the domain has been set up to point towards your home network, ensure that your home network is able to carry forward and route the traffic to your VPN server. This can be achieved through [port forwarding](router.md#port-forwarding):

   - Service Name: Set any suitable name that best describes the service or rule (i.e. `vpn`)
   - Device IP Address: Enter the private IP address of the WireGuard server (i.e. `192.168.0.106`)
   - External Port: Enter the default port number for WireGuard (`51820`) or set a custom port number (i.e. `51821`)
   - Internal Port: Enter the same port number you had set for the external port (i.e. `51820`)
   - Protocol: `UDP`
   - Enabled: Toggle or check this box to enable it

4. Run the PiVPN installation wizard:

    ```sh
    curl -L https://install.pivpn.io | bash
    ```

5. Follow the guided installation process, choosing these options when prompted:

   - Installation mode: `WireGuard`
   - Default wireguard Port: Enter a custom value if you wish to allocate a different port or stick with the default value of `51820`
   - DNS Provider: Select one of the available DNS provider (i.e. `Cloudflare`) or pick `Custom` if you wish to add a custom DNS server (i.e. your own Pi-Hole server)
   - Public IP or DNS: Select the `DNS Entry` option to point towards your public WireGuard server endpoint via DNS
     - What is the public DNS name of this Server?: Set this to the domain you had set up and configured for the WireGuard server (i.e. `vpn.example.com`)

6. Reboot the WireGuard server once the installation and setup process is complete.

### Client Profile

1. On the WireGuard server, use the following command to create a profile for a single client:

    ```sh
    pivpn add -n <profile-name>
    ```

   - Replace `<profile-name>` with a suitable name for the client profile (i.e. `my-device-1`)

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

    - Replace `<profile-name>` with the name of the client profile (i.e. `my-device-1`)

3. For steps on how to establish a connection to the VPN server using these profiles on the client devices, please refer to [this guide](https://github.com/irfanhakim-as/linux-wiki/blob/master/topics/wireguard.md#connecting-to-wireguard).
