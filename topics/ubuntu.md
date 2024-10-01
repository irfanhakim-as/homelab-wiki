# Ubuntu

## Description

Ubuntu is a Linux distribution derived from Debian and composed mostly of free and open-source software. Ubuntu is officially released in multiple editions: Desktop, Server, and Core for Internet of things devices and robots.

## Directory

- [Ubuntu](#ubuntu)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Installation](#installation)
    - [Description](#description-1)
    - [References](#references-1)
    - [Minimum Requirements](#minimum-requirements)
    - [Steps](#steps)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [Base VM](#base-vm)
    - [Extended VM](#extended-vm)
  - [Networking](#networking)
    - [Description](#description-3)
    - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Update Hostname](#update-hostname)

## References

- [Canonical Ubuntu](https://ubuntu.com)

---

## Installation

### Description

This details the installation process for Ubuntu as a server on a virtual machine.

### References

- [Ubuntu 20.04.6 LTS (Focal Fossa)](https://releases.ubuntu.com/focal)

### Minimum Requirements

> [!TIP]  
> It is recommended to use the minimum requirements for a base VM and expand them as needed for extended VMs.

- CPU: `1`
- Memory: `1GB`
- Storage: `10GB`

### Steps

1. Go through the installation wizard as per usual.

2. In the **Profile setup** section:

    - Your name: Set a display name suitable for your homelab
    - Your server's name: Set a unique name for said server without the domain name (i.e. `ubuntu-server`)
    - Pick a username: Set a username suitable for your homelab
    - Choose a password: Set a secure, minimum 12 character password
    - Confirm your password: Confirm the password you have set

3. In the **SSH Setup** section:

    - Install OpenSSH server: `Enabled`
    - Import SSH identity: `Disabled`

4. Complete the installation.

---

## Configuration

### Description

This details some recommended configuration options for Ubuntu as a server.

### Base VM

1. Perform a [full system upgrade](package-manager.md#update-software) on the system using `apt`.

2. [Install](package-manager.md#install-software) the following packages using `apt`:

   - `curl`
   - `git`
   - `htop`
   - `nano`
   - `net-tools`
   - `openssh-server`
   - `python3`
   - `python3-pip`
   - `wget`

3. [Install](package-manager.md#install-software) packages required by your [hypervisor](../courses/hypervisor.md) using `apt`:

   - ESXi: `open-vm-tools`
   - Proxmox: `qemu-guest-agent`

        > [!TIP]  
        > On Proxmox, you may also need to [enable](systemd.md#enable-service) the `qemu-guest-agent` service yourself.

4. [Clean up](package-manager.md#clean-up) the system using `apt` to recover some storage space.

5. [Enable the SSH service](ssh.md#enable-remote-access) (if it is not already enabled).

6. [Copy over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system.

7. Decide on a new SSH port (i.e. `2222`) to replace the default port, `22`.

8. Update the VM's [SSH configuration](ssh.md#configuration), which includes:

   - Changing the default SSH port to a new port (i.e. `2222`).

   - Disabling root login via SSH.

   - Disabling password authentication via SSH.

        > [!WARNING]  
        > Make sure you have [copied over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system before applying this change!

9. Set up the VM firewall (UFW):

   - [Enable the firewall](firewall.md#enablement).

   - Check the [firewall status](firewall.md#status) to ensure it is active and no rules have been configured.

   - [Allow the connection](firewall.md#adding-allow-rule) to the new SSH port (i.e. `2222`) using the `tcp` protocol.

10. Clear the VM's Bash history:

    ```sh
    history -c
    cat /dev/null > ~/.bash_history && history -c && history -w
    ```

    > [!NOTE]  
    > There should be no other active sessions on the VM while doing this.

11. Reboot the VM to apply all changes:

    ```sh
    sudo reboot now
    ```

### Extended VM

These configuration options are expected to be applied on top of the changes that were made to the [base VM](#base-vm) inherited by the extended VM:

1. Perform a [full system upgrade](package-manager.md#update-software) on the system using `apt`, especially if it's been a while since the base VM was last upgraded.

2. [Set a static IP address and update the DNS server](#set-static-ip-and-update-dns) for the VM.

3. [Update the system hostname](#update-hostname).

---

## Networking

### Description

This details the process of updating certain networking configurations on the system.

### Set Static IP and Update DNS

This details the process of setting a static IP address and updating the DNS server on a system.

1. Backup the existing network configuration file:

    ```sh
    sudo cp /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.bak
    ```

2. Update the network configuration file:

    ```sh
    sudo nano /etc/netplan/00-installer-config.yaml
    ```

3. Update the configuration as such:

    Original configuration which uses DHCP to dynamically assign an IP address:

    ```yaml
    network:
      ethernets:
        <network-interface>:
          dhcp4: true
      version: 2
    ```

    Make the following changes:

    ```diff
    network:
      ethernets:
        <network-interface>:
    +     addresses: [<ip-address>/24]
    -     dhcp4: true
    +     dhcp4: false
    +     nameservers:
    +       addresses: [<dns1>,<dns2>]
      version: 2
    ```

    The following is an example of the updated configuration:

    ```yaml
    network:
      ethernets:
        enp6s18:
          addresses: [192.168.0.106/24]
          dhcp4: false
          nameservers:
            addresses: [1.1.1.1,8.8.8.8]
      version: 2
    ```

4. Save all changes made to the file and apply the new configuration:

    ```sh
    sudo netplan apply
    ```

### Update Hostname

This details the simple process of updating system's hostname:

1. Update the system's hostname using `hostnamectl`:

    ```sh
    sudo hostnamectl set-hostname "<hostname>.<domain>"
    ```

    As an example, if the new `<hostname>` is `ubuntu-1` and the `<domain>` is `example.com`:

    ```sh
    sudo hostnamectl set-hostname "ubuntu-1.example.com"
    ```

2. **(Optional)** Update the system's hosts file:

    ```sh
    sudo nano /etc/hosts
    ```

    Sample of the original hosts file:

    ```
    127.0.0.1 localhost
    127.0.1.1 ubuntu-server

    # The following lines are desirable for IPv6 capable hosts
    ::1     ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ```

    As an example, update the hostname from `ubuntu-server` to `ubuntu-1`:

    ```diff
    127.0.0.1 localhost
    - 127.0.1.1 ubuntu-server
    + 127.0.1.1 ubuntu-1

    # The following lines are desirable for IPv6 capable hosts
    ::1     ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ```

3. Reboot the VM to apply the changes:

    ```sh
    sudo reboot now
    ```
