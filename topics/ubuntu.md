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
    - [References](#references-2)
    - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Update Hostname](#update-hostname)
  - [User Management](#user-management)
    - [Description](#description-4)
    - [Create User](#create-user)
    - [Add User to Group](#add-user-to-group)
  - [Sudo](#sudo)
    - [Description](#description-5)
    - [Steps](#steps-1)

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

2. [Create a service user](#create-user) on the system if there isn't one already and [grant it sudo privileges](#sudo).

3. [Install](package-manager.md#install-software) the following packages using `apt`:

   - `curl`
   - `git`
   - `htop`
   - `nano`
   - `net-tools`
   - `openssh-server`
   - `pciutils`
   - `python3`
   - `python3-pip`
   - `rsync`
   - `wget`

4. **(VM Only)** [Install](package-manager.md#install-software) packages required by your [hypervisor](../courses/hypervisor.md) using `apt`:

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

11. For systems running `cloud-init` (i.e. on a Raspberry Pi), password prompts may not be required when using `sudo` by default.

    - To fix this, check the content of the `/etc/sudoers` file:

      ```sh
      sudo cat /etc/sudoers
      ```

    - Identify whether the following line is found in the file and not commented:

      ```
      %sudo   ALL=(ALL:ALL) ALL
      ```

      If it is not found, add the line to the end of the file accordingly. Likewise, uncomment the line if it is commented.

    - Now, check for a configuration file in the `/etc/sudoers.d/` directory that is responsible for the current behaviour:

      ```sh
      sudo ls -l /etc/sudoers.d/
      ```

      Sample output:

      ```
      total 8
      -r--r----- 1 root root 149 Apr 21  2022 90-cloud-init-users
      -r--r----- 1 root root 958 Feb  3  2020 README
      ```

      In this example, the configuration file is `90-cloud-init-users`.

    - Update the configuration file (i.e. `90-cloud-init-users`) to change this behaviour:

      ```sh
      sudo nano /etc/sudoers.d/90-cloud-init-users
      ```

      Sample configuration:

      ```
      # Created by cloud-init v. 22.2-0ubuntu1~20.04.3 on Thu, 21 Apr 2022 12:54:58 +0000

      # User rules for <user>
      <user> ALL=(ALL) NOPASSWD:ALL
      ```

      Comment the line with the `NOPASSWD:ALL` option to prevent it from allowing the `<user>` user from using `sudo` without a password:

      ```diff
        # Created by cloud-init v. 22.2-0ubuntu1~20.04.3 on Thu, 21 Apr 2022 12:54:58 +0000

        # User rules for <user>
      - <user> ALL=(ALL) NOPASSWD:ALL
      + #<user> ALL=(ALL) NOPASSWD:ALL
      ```

      **Alternatively**, if you think it is safe, you may also delete the configuration file entirely:

      ```sh
      sudo rm -f /etc/sudoers.d/90-cloud-init-users
      ```

    - Finally, disable `cloud-init` completely to prevent this from being overwritten:

      ```sh
      sudo touch /etc/cloud/cloud-init.disabled
      ```

      **Alternatively**, disable `cloud-init` from managing any `sudo` configuration for the user:

      ```sh
      sudo nano /var/lib/cloud/instance/user-data.txt
      ```

      Look for the `NOPASSWD:ALL` option and comment or remove it:

      ```diff
      - sudo: ALL=(ALL) NOPASSWD:ALL
      + #sudo: ALL=(ALL) NOPASSWD:ALL
      ```

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

1. Perform a [full system upgrade](package-manager.md#update-software) on the system using `apt`, especially if it's been a while since the base VM was last upgraded.

2. [Set a static IP address and update the DNS server](#set-static-ip-and-update-dns) for the VM.

3. [Update the system hostname](#update-hostname).

---

## Networking

> [!NOTE]  
> Proxmox [LXC Containers](proxmox.md#linux-containers-lxc) have their own, much easier way of managing these settings graphically. If you choose to configure them manually, [make sure they persist](proxmox.md#persist-configurations-in-lxc).

### Description

This details the process of updating certain networking configurations on the system.

### References

- [How to Configure Static IP Address on Ubuntu 20.04](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04)

### Set Static IP and Update DNS

This details the process of setting a static IP address and updating the DNS server on a system.

1. Ensure the system's network configuration is not provisioned by `cloud-init`:

    ```sh
    ls -l /etc/netplan/*-cloud-init.yaml
    ```

    If the output is not empty, then the system is provisioned by `cloud-init` and needs to be disabled.

2. To disable `cloud-init` from managing the system's network configuration:

    ```sh
    sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```

    Add and save the following configuration to the file:

    ```
    network: {config: disabled}
    ```

3. Backup the existing network configuration file, if available:

    ```sh
    sudo cp /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.bak
    ```

4. Determine the name of the active network interface on the system:

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

5. Update or create the network configuration file:

    ```sh
    sudo nano /etc/netplan/00-installer-config.yaml
    ```

6. Update the configuration as such:

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

7. Save all changes made to the file and apply the new configuration:

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

2. **(Optional)** If you intend to update the system's hosts file, Ensure the system's hosts file is not managed by `cloud-init`:

    ```sh
    cat /etc/hosts
    ```

    If the start of the hosts file has the following disclaimer, then it is likely managed by `cloud-init` and needs to be disabled:

    ```
    # Your system has configured 'manage_etc_hosts' as True.
    # As a result, if you wish for changes to this file to persist
    # then you will need to either
    # a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
    # b.) change or remove the value of 'manage_etc_hosts' in
    #     /etc/cloud/cloud.cfg or cloud-config from user-data
    #
    ```

3. To disable `cloud-init` completely:

    ```sh
    sudo touch /etc/cloud/cloud-init.disabled
    ```

4. **(Optional)** Update the system's hosts file:

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

5. Reboot the VM to apply the changes:

    ```sh
    sudo reboot now
    ```

---

## User Management

### Description

This details topics pertaining to user and group management on the system.

### Create User

This details how to create a service user on the system:

1. As the `root` user, run the following command to create a user:

    ```sh
    adduser <username>
    ```

    For example, if the username of the service user is `foo`:

    ```sh
    adduser foo
    ```

2. After providing the necessary user information including password, the following sample output is expected:

    ```
      Adding user `foo' ...
      Adding new group `foo' (1000) ...
      Adding new user `foo' (1000) with group `foo' ...
    ```

### Add User to Group

This details how to add a user to a specific group:

1. As the `root` user, run the following command to add a user to a group:

    ```sh
    usermod -aG <group> <username>
    ```

    For example, if the username of the service user is `foo` and the group is `bar`:

    ```sh
    usermod -aG bar foo
    ```

2. Verify that the user has been added to the group:

   - Replace `<username>` with the username of the service user:

      ```sh
      id <username>
      ```

      For example, if the username of the service user is `foo`:

      ```sh
      id foo
      ```

   - Sample output:

      ```
        uid=1000(<username>) gid=1000(<username>) groups=1000(<username>),999(<group>)
      ```

      Based on the output, ensure that the user (i.e. `foo`) has the group (i.e. `bar`) added to their `groups` list.

---

## Sudo

### Description

This details the process of granting temporary superuser (sudo) privileges to a user.

### Steps

1. Switch to the `root` user if you are not logged in as root:

    ```sh
    su -
    ```

    Proceed with the rest of the following steps as root.

2. [Install](package-manager.md#install-software) the `sudo` package using `apt` if it is not already installed.

3. [Create a service user](#create-user) that is to be granted sudo privileges if you have not already.

4. [Add the service user to the `sudo` group](#add-user-to-group) to give them sudo privileges.

5. Log out and log back in to apply and test the changes:

   - Press <kbd>Ctrl + D</kbd> to log out of the root user.
   - Press <kbd>Ctrl + D</kbd> again if you are logged in as the service user.
   - Log back in as the service user.
   - As the service user, execute a command that requires root privileges using `sudo`.
