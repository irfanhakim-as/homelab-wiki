# Debian

## Description

Debian, also known as Debian GNU/Linux, is a free and open source Linux distribution, developed by the Debian Project, which was established by Ian Murdock in August 1993. Debian is the basis for many other distributions, such as Debian, Linux Mint, Tails, Proxmox, Kali Linux, Pardus, TrueNAS SCALE, and Astra Linux.

## Directory

- [Debian](#debian)
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
  - [Sudo](#sudo)
    - [Description](#description-4)
    - [References](#references-2)
    - [Steps](#steps-1)

## References

- [Debian](https://www.debian.org)

---

## Installation

### Description

This details the installation process for Debian as a server on a virtual machine.

### References

- [Debian CD](https://cdimage.debian.org/debian-cd/current/amd64/bt-cd)
- [Debian 12 "Net Install" Installation Walkthrough](https://youtu.be/gddlhr9ST9Y)

### Minimum Requirements

> [!TIP]  
> It is recommended to use the minimum requirements for a base VM and expand them as needed for extended VMs.

- CPU: `1`
- Memory: `512MB`
- Storage: `10GB`

### Steps

1. Pick the **Install** option when given the option.

2. Go through the installation wizard as per usual.

3. In the **Configure the network** section:

    - Hostname: Set a unique name for said server without the domain name (i.e. `debian-server`)
    - Domain name: Leave this blank or add in your domain name (i.e. `example.com`)

4. In the **Set up users and passwords** section:

    - Root password: Set a secure, minimum 12 character password for the `root` user
    - Re-enter root password to verify: Confirm the password you have set for the `root` user
    - Full name for the new user: Set a display name suitable for your homelab
    - Username for your account: Set a username suitable for your homelab
    - Choose a password for the new user: Set a secure, minimum 12 character password **different** from the password set for the `root` user
    - Re-enter password to verify: Confirm the password you have set

5. In the **Partition disks** section:

    - Partitioning method: Select the ~~**Guided - use entire disk**~~ **Guided - use entire disk and set up LVM** option
    - Select disk to partition: Select the disk you wish to use for the installation
    - Partitioning scheme: Select the **All files in one partition (recommended for new users)** option
    - Overview of the new partitions: Select the **Finish partitioning and write changes to disk** option
    - Write the changes to disks: Select the **Yes** option

6. When asked to scan extra installation media, select **No**.

7. In the **Debian archive mirror country** selection section: Select the country closest to you (i.e. `Singapore`).

8. In the **Debian archive mirror** selection section: Select the mirror you think is best (i.e. `deb.debian.org`).

9. Leave the **HTTP proxy information** section blank and select **Continue**.

10. Participate in the package usage survey: Select **No**.

11. In the **Choose software to install** section, ensure everything is `Disabled` except the following:

    - SSH server: `Enabled`
    - Standard system utilities: `Enabled`

    Select the **Continue** option.

12. If asked to **Install the GRUB boot loader to your primary drive**, select **Yes**.

13. For the **Device for boot loader installation**, select the disk you have installed Debian on i.e. `/dev/sda`.

14. Complete the installation.

---

## Configuration

### Description

This details some recommended configuration options for Debian as a server.

### Base VM

1. Setup [`sudo`](#sudo) for the system's service user as root.

2. Perform a [full system upgrade](package-manager.md#update-software) on the system using `apt`.

3. [Install](package-manager.md#install-software) the following packages using `apt`:

   - `curl`
   - `git`
   - `htop`
   - `nano`
   - `net-tools`
   - `openssh-server`
   - `python3`
   - `python3-pip`
   - `ufw`
   - `wget`

4. [Install](package-manager.md#install-software) packages required by your [hypervisor](../courses/hypervisor.md) using `apt`:

   - ESXi: `open-vm-tools`
   - Proxmox: `qemu-guest-agent`

        > [!TIP]  
        > On Proxmox, you may also need to [enable](systemd.md#enable-service) the `qemu-guest-agent` service yourself.

5. [Clean up](package-manager.md#clean-up) the system using `apt` to recover some storage space.

6. [Enable the SSH service](ssh.md#enable-remote-access) (if it is not already enabled).

7. [Copy over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system.

8. Decide on a new SSH port (i.e. `2222`) to replace the default port, `22`.

9. Update the VM's [SSH configuration](ssh.md#configuration), which includes:

   - Changing the default SSH port to a new port (i.e. `2222`).

   - Disabling root login via SSH.

   - Disabling password authentication via SSH.

        > [!WARNING]  
        > Make sure you have [copied over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system before applying this change!

10. Set up the VM firewall (UFW):

    - [Enable the firewall](firewall.md#enablement).

    - Check the [firewall status](firewall.md#status) to ensure it is active and no rules have been configured.

    - [Allow the connection](firewall.md#adding-allow-rule) to the new SSH port (i.e. `2222`) using the `tcp` protocol.

11. Clear the VM's Bash history:

    ```sh
    history -c
    cat /dev/null > ~/.bash_history && history -c && history -w
    ```

    > [!NOTE]  
    > There should be no other active sessions on the VM while doing this.

12. Reboot the VM to apply all changes:

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
    sudo cp /etc/network/interfaces /etc/network/interfaces.bak
    ```

2. Determine the name of the active network interface on the system:

    ```sh
    route | grep '^default' | grep -o '[^ ]*$'
    ```

    Sample output:

    ```
    enp6s18
    ```

    Alternatively, you may also use the following command:

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

3. Update the network configuration file:

    ```sh
    sudo nano /etc/network/interfaces
    ```

4. Update the configuration as such:

    Original configuration which uses DHCP to dynamically assign an IP address:

    ```
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    source /etc/network/interfaces.d/*

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    allow-hotplug <network-interface>
    iface <network-interface> inet dhcp
    ```

    Make the following changes:

    ```diff
    # The primary network interface
    - allow-hotplug <network-interface>
    + auto <network-interface>
    - iface <network-interface> inet dhcp
    + iface <network-interface> inet static
    + address <ip-address>
    + netmask <subnet-mask>
    + 
    + gateway <gateway>
    + dns-nameservers <dns1> <dns2>
    ```

    The following is an example of the updated configuration:

    ```
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    source /etc/network/interfaces.d/*

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    auto enp6s18
    iface enp6s18 inet static
    address 192.168.0.106
    netmask 255.255.255.0

    gateway 192.168.0.1
    dns-nameservers 1.1.1.1 8.8.8.8
    ```

5. Save all changes made to the file and apply the new configuration by [restarting](systemd.md#restart-service) the `networking.service` service.

### Update Hostname

This details the simple process of updating system's hostname:

1. Update the system's hostname using `hostnamectl`:

    ```sh
    sudo hostnamectl set-hostname "<hostname>.<domain>"
    ```

    As an example, if the new `<hostname>` is `debian-1` and the `<domain>` is `example.com`:

    ```sh
    sudo hostnamectl set-hostname "debian-1.example.com"
    ```

2. **(Optional)** Update the system's hosts file:

    ```sh
    sudo nano /etc/hosts
    ```

    Sample of the original hosts file:

    ```
    127.0.0.1       localhost
    127.0.1.1       debian-server.example.com   debian-server

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ```

    As an example, update the hostname from `debian-server` to `debian-1`:

    ```diff
    127.0.0.1       localhost
    - 127.0.1.1       debian-server.example.com   debian-server
    + 127.0.1.1       debian-1.example.com   debian-1

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ```

3. Reboot the VM to apply the changes:

    ```sh
    sudo reboot now
    ```

---

## Sudo

### Description

This details the process of setting up `sudo` and adding the user to the `sudo` group.

### References

- [How to fix debian sudo command not found](https://linuxhint.com/how-to-fix-debian-sudo-command-not-found)

### Steps

1. Switch to the root user if you are not logged in as root:

    ```sh
    su -
    ```

    Enter the root password when prompted.

2. Perform a [full system upgrade](package-manager.md#update-software) on the system using `apt` (without `sudo` since we're running as root).

3. [Install](package-manager.md#install-software) `sudo` using `apt`.

4. Add your service user (non-root user) to the `sudo` group:

    ```sh
    usermod -aG sudo <username>
    ```

    As an example, if the username of the service user is `foo`:

    ```sh
    usermod -aG sudo foo
    ```

5. Verify that the user has been added to the `sudo` group:

    ```sh
    id <username>
    ```

    Example output, assuming the username is `foo`:

    ```sh
    uid=1000(foo) gid=1000(foo) groups=1000(foo),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
    ```

6. Log out and log back in to apply and test the changes:

   - Press <kbd>Ctrl + D</kbd> to log out of the root user.
   - Press <kbd>Ctrl + D</kbd> again to log out of the service user.
   - Log back in as the service user.
