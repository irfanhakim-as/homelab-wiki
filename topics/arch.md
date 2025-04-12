# Arch Linux

## Description

Arch Linux is an independently developed, x86-64 general-purpose GNU/Linux distribution that strives to provide the latest stable versions of most software by following a rolling-release model. The default installation is a minimal base system, configured by the user to only add what is purposely required.

## Directory

- [Arch Linux](#arch-linux)
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
    - [References](#references-2)
    - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Update Hostname](#update-hostname)

## References

- [Arch Linux](https://archlinux.org)
- [ArchWiki](https://wiki.archlinux.org)

---

## Installation

### Description

This details the installation process for Arch Linux as a server on a virtual machine.

### References

- [Arch Linux Downloads](https://archlinux.org/download)
- [Archinstall - Arch Linux Made Easy](https://youtu.be/G-mLyrHonvU)
- [Installing Arch Linux in 2024](https://youtu.be/qZ7toS5kJkc)
- [Proxmox/Install Arch Linux as a guest](https://wiki.archlinux.org/title/Proxmox/Install_Arch_Linux_as_a_guest)
- [Guided installation](https://archinstall.archlinux.page/installing/guided.html)

### Minimum Requirements

> [!TIP]  
> It is recommended to use the minimum requirements for a base VM and expand them as needed for extended VMs.

- CPU: `1`
- Memory: `1024MB`
- Storage: `10GB`

### Steps

1. [Configure `pacman`](package-manager.md#configuration) by updating the installer's mirror list and enabling parallel downloads.

2. Run the installer:

    ```sh
    archinstall
    ```

3. From the installer, configure the following:

   - Disk configuration:
     - Partitioning:
       - Select a partitioning option: `Use a best-effort default partition layout`
       - Select one or more devices to use and configure: Select the virtual disk drive (i.e. `QEMU QEMU HARDDISK`)
       - Select which filesystem your main partition should use: `ext4`
     - LVM - Logical Volume Management (BETA):
       - Select a LVM option: `Default layout`
       - Select which filesystem your main partition should use: `ext4`
   - Bootloader: `Grub`
   - Hostname: Set a unique name for said server (i.e. `arch-server.example.com`)
   - Root password: Set a secure, minimum 12 character password for the root user
   - User account:
     - Select the **Add a user** option:
       - Enter username: Set a username suitable for your homelab
       - Password for user: Set a secure, minimum 12 character password **different** from the password set for the root user
       - Should user be a superuser (sudo): `yes`
     - Select the **Confirm and exit** option.
   - Profile:
     - Type: `Server`
     - Which servers to install: Select the `sshd` option
   - Audio: `No audio server`
   - Kernels: `linux-lts`
   - Network configuration: `Copy ISO network configuration to installation`
   - Timezone: `UTC` or select your local timezone (i.e. `Asia/Kuala_Lumpur`)
   - Automatic time sync (NTP): `True`
   - Save configuration: `Save all`

    Select the **Install** option once all has been configured.

4. When asked if you would **like to chroot into the newly created installation and perform post-installation configuration?**: Answer `no`.

5. Complete the installation.

---

## Configuration

### Description

This details some recommended configuration options for Arch Linux as a server.

### Base VM

1. [Configure `pacman`](package-manager.md#configuration) by updating the system's mirror list and enabling parallel downloads.

2. Perform a [full system upgrade](package-manager.md#update-software) on the system using `pacman`.

3. [Install](package-manager.md#install-software) the following packages using `pacman`:

   - `base-devel`
   - `curl`
   - `git`
   - `htop`
   - `inetutils`
   - `nano`
   - `net-tools`
   - `openssh`
   - `pciutils`
   - `python`
   - `python-pip`
   - `vim`
   - `wget`

4. **(VM Only)** [Install](package-manager.md#install-software) packages required by your [hypervisor](../courses/hypervisor.md) using `pacman`:

   - ESXi: `open-vm-tools`
   - Proxmox: `qemu-guest-agent`

        > [!TIP]  
        > On Proxmox, you may also need to [enable](systemd.md#enable-service) the `qemu-guest-agent` service yourself.

5. [Clean up](package-manager.md#clean-up) the system using `pacman` to recover some storage space.

6. [Enable the SSH service](ssh.md#enable-remote-access) (if it is not already enabled).

7. [Copy over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system.

8. Decide on a new SSH port (i.e. `2222`) to replace the default port, `22`.

9. Update the VM's [SSH configuration](ssh.md#configuration), which includes:

   - Changing the default SSH port to a new port (i.e. `2222`).

   - Disabling root login via SSH.

   - Disabling password authentication via SSH.

        > [!WARNING]  
        > Make sure you have [copied over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system before applying this change!

10. Set up the VM firewall (Firewalld):

    - [Install](package-manager.md#install-software) the `firewalld` package using `pacman`.

    - [Enable the firewall](firewall.md#enablement).

    - Check the [firewall status](firewall.md#status) to ensure it is active.

    - [Allow the connection](firewall.md#adding-allow-rule) to the new SSH port (i.e. `2222`) using the `tcp` protocol.

    - [Remove the (default) firewall rule](firewall.md#delete-rule) allowing the old SSH port using its service name, `ssh`.

11. **(Optional)** [Install and set up `yay`](package-manager.md#setup) as an alternative package manager to `pacman` to get easy access to the packages in the [AUR](https://aur.archlinux.org).

12. Clear the VM's Bash history:

    ```sh
    history -c
    cat /dev/null > ~/.bash_history && history -c && history -w
    ```

    > [!NOTE]  
    > There should be no other active sessions on the VM while doing this.

13. Reboot the VM to apply all changes:

    ```sh
    sudo reboot now
    ```

### Extended VM

These configuration options are expected to be applied on top of the changes that were made to the [base VM](#base-vm) inherited by the extended VM:

1. Perform a [full system upgrade](package-manager.md#update-software) on the system using `pacman` or `yay`, especially if it's been a while since the base VM was last upgraded.

2. [Set a static IP address and update the DNS server](#set-static-ip-and-update-dns) for the VM.

3. [Update the system hostname](#update-hostname).

---

## Networking

> [!NOTE]  
> Proxmox [LXC Containers](proxmox.md#linux-containers-lxc) have their own, much easier way of managing these settings graphically. If you choose to configure them manually, [make sure they persist](proxmox.md#persist-configurations-in-lxc).

### Description

This details the process of updating certain networking configurations on the system.

### References

- [Network files](https://wiki.archlinux.org/title/Systemd-networkd#network_files)
- [hosts(5)](https://man.archlinux.org/man/hosts.5.en)

### Set Static IP and Update DNS

This details the process of setting a static IP address and updating the DNS server on a system.

1. Determine the name of the active network interface on the system:

    ```sh
    route | grep '^default' | grep -o '[^ ]*$'
    ```

    Sample output:

    ```
    enp6s18
    ```

    **Alternatively**, you may also use the following command:

    ```sh
    ip link
    ```

    Sample output:

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether fg:LA:Og:mQ:LQ:7Q brd ff:ff:ff:ff:ff:ff
    ```

    In this example, `enp6s18` is the name of the active network interface.

2. Create a network configuration file:

    ```sh
    sudo nano /etc/systemd/network/10-wired.network
    ```

    > [!TIP]  
    > The `wired` part of the name is arbitrary. Feel free to change it with something more or as descriptive (i.e. the name of the network interface).

3. Update the configuration as such:

    Add the following lines with your values:

    ```conf
    [Match]
    Name=<network-interface>

    [Network]
    DHCP=no
    Address=<ip-address>/24
    Gateway=<gateway>
    DNS=<dns1>
    DNS=<dns2>
    ```

    The following is an example of the updated configuration:

    ```conf
    [Match]
    Name=enp6s18

    [Network]
    DHCP=no
    Address=192.168.0.106/24
    Gateway=192.168.0.1
    DNS=1.1.1.1
    DNS=8.8.8.8
    ```

4. Save all changes made to the file and apply the new configuration by [enabling](systemd.md#enable-service) and [restarting](systemd.md#restart-service) the following services:

   - `systemd-networkd.service`
   - `systemd-resolved.service`

### Update Hostname

This details the simple process of updating system's hostname:

1. Update the system's hostname using `hostnamectl`:

    ```sh
    sudo hostnamectl set-hostname "<hostname>.<domain>"
    ```

    As an example, if the new `<hostname>` is `arch-1` and the `<domain>` is `example.com`:

    ```sh
    sudo hostnamectl set-hostname "arch-1.example.com"
    ```

2. **(Optional)** Update the system's hosts file:

    ```sh
    sudo nano /etc/hosts
    ```

    Sample of the original hosts file:

    ```
    # Static table lookup for hostnames.
    # See hosts(5) for details.
    ```

    As an example, set the hostname to `arch-1` and update the hosts file with these changes:

    ```diff
    # Static table lookup for hostnames.
    # See hosts(5) for details.
    + 127.0.0.1       localhost
    + 127.0.1.1       arch-1.example.com arch-1
    +
    + # The following lines are desirable for IPv6 capable hosts
    + ::1     localhost ip6-localhost ip6-loopback
    + ff02::1 ip6-allnodes
    + ff02::2 ip6-allrouters
    ```

    Sample of the updated hosts file:

    ```
    # Static table lookup for hostnames.
    # See hosts(5) for details.
    127.0.0.1       localhost
    127.0.1.1       arch-1.example.com arch-1

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ```

3. Reboot the VM to apply the changes:

    ```sh
    sudo reboot now
    ```
