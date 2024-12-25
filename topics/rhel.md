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
  - [Networking](#networking)
    - [Description](#description-3)
    - [References](#references-3)
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

3. In the **Network & Hostname** section, select the network interface you wish to use (i.e. `Ethernet (ens192)`), toggle the switch to `ON` to enable it, and click the **Done** button.

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
- [SELinux security](https://docs.rockylinux.org/guides/security/learning_selinux)
- [Change SSH Port on CentOS or RHEL or Fedora With SELinux](https://computingforgeeks.com/change-ssh-port-centos-rhel-fedora-with-selinux)

### Base VM

1. Perform a [full system upgrade](package-manager.md#update-software) on the system using `dnf`.

2. Enable the EPEL repository by [installing](package-manager.md#install-software) `https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm` using `dnf`.

3. [Group install](package-manager.md#install-software) the `'Development Tools'` group using `dnf`.

4. [Install](package-manager.md#install-software) the following packages using `dnf`:

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

    > [!NOTE]  
    > The `policycoreutils-python-utils` package is required for the `semanage` command, which is used to manage SELinux policy.

5. [Install](package-manager.md#install-software) packages required by your [hypervisor](../courses/hypervisor.md) using `dnf`:

   - ESXi: `open-vm-tools`
   - Proxmox: `qemu-guest-agent`

        > [!TIP]  
        > On Proxmox, you may also need to [enable](systemd.md#enable-service) the `qemu-guest-agent` service yourself.

6. [Clean up](package-manager.md#clean-up) the system using `dnf` to recover some storage space.

7. [Enable the SSH service](ssh.md#enable-remote-access) (if it is not already enabled).

8. [Copy over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system.

9. Decide on a new SSH port (i.e. `2222`) to replace the default port, `22`.

10. SELinux is enabled by default on Rocky Linux and needs to be configured to allow access to the new SSH port.

    - Add the new SSH port to the `ssh_port_t` SELinux policy:

        ```sh
        sudo semanage port -a -t ssh_port_t -p tcp <ssh-port>
        ```

        For example, if the new SSH port is `2222`:

        ```sh
        sudo semanage port -a -t ssh_port_t -p tcp 2222
        ```

    - Verify that the port is now accessible from the `ssh` service:

        ```sh
        sudo semanage port -l | grep ssh_port_t
        ```

        Sample output:

        ```
        ssh_port_t                     tcp      2222, 22
        ```

11. Update the VM's [SSH configuration](ssh.md#configuration), which includes:

    - Changing the default SSH port to a new port (i.e. `2222`).

    - Disabling root login via SSH.

    - Disabling password authentication via SSH.

        > [!WARNING]  
        > Make sure you have [copied over your public SSH key(s)](ssh.md#copy-ssh-keys) to the system before applying this change!

12. Set up the VM firewall (Firewalld):

    - [Enable the firewall](firewall.md#enablement).

    - Check the [firewall status](firewall.md#status) to ensure it is active.

    - [Allow the connection](firewall.md#adding-allow-rule) to the new SSH port (i.e. `2222`) using the `tcp` protocol.

    - [Remove the (default) firewall rule](firewall.md#delete-rule) allowing the old SSH port using its service name, `ssh`.

13. Clear the VM's Bash history:

    ```sh
    history -c
    cat /dev/null > ~/.bash_history && history -c && history -w
    ```

    > [!NOTE]  
    > There should be no other active sessions on the VM while doing this.

14. Reboot the VM to apply all changes:

    ```sh
    sudo reboot now
    ```

### Extended VM

These configuration options are expected to be applied on top of the changes that were made to the [base VM](#base-vm) inherited by the extended VM:

1. Perform a [full system upgrade](package-manager.md#update-software) on the system using `dnf`, especially if it's been a while since the base VM was last upgraded.

2. [Set a static IP address and update the DNS server](#set-static-ip-and-update-dns) for the VM.

3. [Update the system hostname](#update-hostname).

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
