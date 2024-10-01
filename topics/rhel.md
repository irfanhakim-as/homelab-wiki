# Red Hat Enterprise Linux (RHEL)

## Description

Red Hat Enterprise Linux is a commercial open-source Linux distribution developed by Red Hat for the commercial market.

## Directory

- [Red Hat Enterprise Linux (RHEL)](#red-hat-enterprise-linux-rhel)
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
    - [References](#references-2)
    - [Base VM](#base-vm)
    - [Extended VM](#extended-vm)
  - [Software Manager](#software-manager)
    - [Description](#description-3)
    - [References](#references-3)
    - [Install Software](#install-software)
    - [Update Software](#update-software)
    - [Remove Software](#remove-software)
    - [Search Software](#search-software)
    - [Clean Up](#clean-up)
  - [Networking](#networking)
    - [Description](#description-4)
    - [References](#references-4)
    - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Update Hostname](#update-hostname)

## References

- [Red Hat Enterprise Linux](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)
- [Rocky Linux](https://rockylinux.org)

---

## Installation

### Description

This details the installation process for Rocky Linux as a server on a virtual machine.

### References

- [Download Rocky](https://rockylinux.org/download)

### Minimum Requirements

> [!TIP]  
> It is recommended to use the minimum requirements for a base VM and expand them as needed for extended VMs.

- CPU: `1`
- Memory: `1GB`
- Storage: `10GB`

### Steps

1. Pick the **Install Rocky Linux 8** option when given the option.

2. Go through the installation wizard as per usual.

3. In the **Network & Hostname** section, select the network interface you wish to use (i.e. `Ethernet (ens192)`), toggle the switch to **ON** to enable it, and click the **Done** button.

4. In the **Time & Date** section, update the **Region** and **City**, and click **Done**.

5. In the **Installation Destination** section, select the disk you wish to use for the installation, ensure that it has a checkmark, and click the **Done** button.

6. In the **Root Password** section, enter a secure password for the root user with a minimum of 12 characters, and click the **Done** button.

7. In the **User Creation** section:

    - Full name: Enter a name suitable for your homelab
    - User name: Enter a username suitable for your homelab
    - Make this user administrator: `Enabled`
    - Require a password to use this account: `Enabled`
    - Password: Enter a secure password for the user with a minimum of 12 characters **different** from the password set for the `root` user
    - Confirm password: Confirm the password you have set

    Click the **Done** button.

8. Click the **Begin Installation** button.

9. Click the **Reboot System** button once the installation is complete.

---

## Configuration

### Description

This details some recommended configuration options for Rocky Linux as a server.

### References

- [How To Install EPEL Repository (Repo) on an RHEL 8](https://www.cyberciti.biz/faq/install-epel-repo-on-an-rhel-8-x)

### Base VM

1. Perform a [full system upgrade](#update-software) on the system.

2. Enable the EPEL repository by [installing](#install-software) `https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`.

3. [Group install](#install-software) the `'Development Tools'` group.

4. [Install](#install-software) the following packages:

   - `curl`
   - `git`
   - `htop`
   - `nano`
   - `net-tools`
   - `openssh-server`
   - `policycoreutils-python-utils`
   - `python3`
   - `python3-pip`
   - `wget`

    > [!TIP]  
    > The `policycoreutils-python-utils` package is required for the `semanage` command, which is used to manage SELinux policy.

5. [Install](#install-software) packages required by your [hypervisor](../courses/hypervisor.md):

   - ESXi: `open-vm-tools`
   - Proxmox: `qemu-guest-agent`

        > [!TIP]  
        > On Proxmox, you may also need to [enable](systemd.md#enable-service) the `qemu-guest-agent` service yourself.

6. [Clean up](#clean-up) the system to recover some storage space.

7. [Enable the SSH service](ssh.md#enable-remote-access) (if it is not already enabled).

8. [Copy over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system.

9. Set up the VM firewall (Firewalld):

   - [Enable the firewall](firewall.md#enablement).

   - Check the [firewall status](firewall.md#status) to ensure it is active and no rules have been configured.

   - Choose a new SSH port (i.e. `2222`) and [allow the connection](firewall.md#adding-allow-rule) to the chosen port using the `tcp` protocol.

10. Update the VM's [SSH configuration](ssh.md#configuration), which includes:

   - Changing the default SSH port to what you had chosen and allowed in the system firewall.

   - Disabling root login via SSH.

   - Disabling password authentication via SSH.

        > [!WARNING]  
        > Make sure you have [copied over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system before applying this change!

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

1. Perform a [full system upgrade](#update-software) on the system, especially if it's been a while since the base VM was last upgraded.

2. [Set a static IP address and update the DNS server](#set-static-ip-and-update-dns) for the VM.

3. [Update the system hostname](#update-hostname).

---

## Software Manager

### Description

This details some basic usage of the default package manager for Rocky Linux, `dnf`.

### References

- [DNF: Dandified Yum](https://docs.rockylinux.org/books/admin_guide/13-softwares/#dnf-dandified-yum)
- [DNF Command Reference](https://dnf.readthedocs.io/en/latest/command_ref.html)

### Install Software

1. To install a single package:

    ```sh
    sudo dnf install <package>
    ```

    As an example, to install the `vim` package:

    ```sh
    sudo dnf install vim
    ```

2. **Optionally**, use the `-y` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo dnf install -y <package>
    ```

3. To install multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo dnf install <package1> <package2> <package3>
    ```

4. To install any of the predefined groups of packages, use the `groupinstall` command:

    ```sh
    sudo dnf groupinstall '<group>'
    ```

    As an example, to install the `Development Tools` group:

    ```sh
    sudo dnf groupinstall 'Development Tools'
    ```

### Update Software

1. To update a single package:

    ```sh
    sudo dnf upgrade <package>
    ```

    As an example, to upgrade the `vim` package:

    ```sh
    sudo dnf upgrade vim
    ```

2. To update the entire system:

    ```sh
    sudo dnf upgrade
    ```

### Remove Software

1. To uninstall a single package:

    ```sh
    sudo dnf remove <package>
    ```

    As an example, to uninstall the `vim` package:

    ```sh
    sudo dnf remove vim
    ```

2. **Optionally**, use the `-y` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo dnf remove -y <package>
    ```

3. To uninstall multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo dnf remove <package1> <package2> <package3>
    ```

### Search Software

1. To search for a package:

    ```sh
    dnf search <package>
    ```

    As an example, to search for the `vim` package:

    ```sh
    dnf search vim
    ```

2. To list down all available predefined groups of packages:

    ```sh
    dnf grouplist
    ```

### Clean Up

1. To remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed:

    ```sh
    sudo dnf autoremove
    ```

2. To delete cached package files and any data that have been left behind:

    ```sh
    sudo dnf clean all
    ```

---

## Networking

### Description

This details the process of updating certain networking configurations on the system.

### References

- [Network Configuration - Rocky Linux](https://docs.rockylinux.org/guides/network/basic_network_configuration)

### Set Static IP and Update DNS

> [!IMPORTANT]  
> This part of the guide assumes that the system is running Rocky Linux 8.

This details the process of setting a static IP address and updating the DNS server on a system.

1. Determine the name of the active network interface on the system:

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

2. Backup the existing network configuration file:

    ```sh
    sudo cp /etc/sysconfig/network-scripts/ifcfg-<network-interface> /etc/sysconfig/network-scripts/ifcfg-<network-interface>.bak
    ```

    As an example, if the network interface is `enp6s18`:

    ```sh
    sudo cp /etc/sysconfig/network-scripts/ifcfg-enp6s18 /etc/sysconfig/network-scripts/ifcfg-enp6s18.bak
    ```

3. Update the network configuration file:

    ```sh
    sudo nano /etc/sysconfig/network-scripts/ifcfg-enp6s18
    ```

4. Update the configuration as such:

    Original configuration which uses DHCP to dynamically assign an IP address:

    ```sh
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    BOOTPROTO=dhcp
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    NAME=enp6s18
    UUID=mNT1Ol1Q-Bq2m-UpAp-n70e-pLvCrMKpbi3N
    DEVICE=enp6s18
    ONBOOT=yes
    ```

    Make the following changes:

    ```diff
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    - BOOTPROTO=dhcp
    + BOOTPROTO=none
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    - IPV6INIT=yes
    + IPV6INIT=no
    - IPV6_AUTOCONF=yes
    + IPV6_AUTOCONF=no
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    NAME=enp6s18
    UUID=mNT1Ol1Q-Bq2m-UpAp-n70e-pLvCrMKpbi3N
    DEVICE=enp6s18
    ONBOOT=yes
    + IPADDR=<ip-address>
    + PREFIX=8
    + GATEWAY=<gateway>
    + DNS1=<dns1>
    + DNS2=<dns2>
    ```

    The following is an example of the updated configuration:

    ```sh
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    BOOTPROTO=none
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=no
    IPV6_AUTOCONF=no
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    NAME=enp6s18
    UUID=mNT1Ol1Q-Bq2m-UpAp-n70e-pLvCrMKpbi3N
    DEVICE=enp6s18
    ONBOOT=yes
    IPADDR=192.168.0.106
    PREFIX=8
    GATEWAY=192.168.0.1
    DNS1=1.1.1.1
    DNS2=8.8.8.8
    ```

5. Save all changes made to the file and apply the new configuration by [restarting](systemd.md#restart-service) the `NetworkManager.service` service.

### Update Hostname

This details the simple process of updating system's hostname:

1. Update the system's hostname using `hostnamectl`:

    ```sh
    sudo hostnamectl set-hostname "<hostname>.<domain>"
    ```

    As an example, if the new `<hostname>` is `rocky-1` and the `<domain>` is `example.com`:

    ```sh
    sudo hostnamectl set-hostname "rocky-1.example.com"
    ```

2. **(Optional)** Update the system's hosts file:

    ```sh
    sudo nano /etc/hosts
    ```

    Sample of the original hosts file:

    ```
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    ```

    As an example, add the `rocky-1` hostname to the hosts file:

    ```diff
    - 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    + 127.0.0.1   rocky-1 rocky-1.example.com localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    ```

3. Reboot the VM to apply the changes:

    ```sh
    sudo reboot now
    ```
