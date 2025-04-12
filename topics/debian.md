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
  - [Storage](#storage)
    - [Description](#description-4)
    - [References](#references-2)
    - [Resize Storage](#resize-storage)
  - [Sudo](#sudo)
    - [Description](#description-5)
    - [References](#references-3)
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
   - `pciutils`
   - `python3`
   - `python3-pip`
   - `ufw`
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

> [!NOTE]  
> Proxmox [LXC Containers](proxmox.md#linux-containers-lxc) have their own, much easier way of managing these settings graphically. If you choose to configure them manually, [make sure they persist](proxmox.md#persist-configurations-in-lxc).

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

## Storage

### Description

This details the process of updating certain storage related configurations on the system.

### References

- [How to List All Block Devices in Linux | lsblk Command](https://www.geeksforgeeks.org/lsblk-command-in-linux-with-examples)

### Resize Storage

After the _physical_ storage of the system itself has been expanded, the following steps should be performed for the new storage capacity to be reflected on the system:

1. List the block devices on the system to determine the partition and logical volume we wish to resize:

    ```sh
    lsblk
    ```

    Sample output:

    ```
        NAME                          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
        sda                             8:0    0   25G  0 disk
        ├─sda1                          8:1    0  512M  0 part /boot/efi
        ├─sda2                          8:2    0  488M  0 part /boot
        └─sda3                          8:3    0    9G  0 part
          ├─debian--server--vg-root   254:0    0  8.1G  0 lvm  /
          └─debian--server--vg-swap_1 254:1    0  980M  0 lvm  [SWAP]
        sr0                            11:0    1 1024M  0 rom
    ```

    Take note of the following details that we will need for the later steps:

      - The logical volume we wish to resize (i.e. `debian--server--vg-root`)
      - The partition containing the logical volume we wish to resize (i.e. `sda3` or `/dev/sda3`)
      - The block device containing the partition we wish to resize (i.e. `sda` or `/dev/sda`)

2. Resize the partition using `fdisk`:

   - Start `fdisk` for the disk which contains the partition we wish to resize (i.e. `/dev/sda`):

        ```sh
        sudo fdisk /dev/sda
        ```

   - In the interactive `fdisk` session, enter the following to print the partition table:

        ```sh
        p
        ```

        Sample output:

        ```
            Disk /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectors
            Disk model: QEMU HARDDISK
            Units: sectors of 1 * 512 = 512 bytes
            Sector size (logical/physical): 512 bytes / 512 bytes
            I/O size (minimum/optimal): 512 bytes / 512 bytes
            Disklabel type: gpt
            Disk identifier: 123E4567-E89B-12D3-A456-426614174000

            Device       Start      End  Sectors  Size Type
            /dev/sda1     2048  1050623  1048576  512M EFI System
            /dev/sda2  1050624  2050047   999424  488M Linux filesystem
            /dev/sda3  2050048 20969471 18919424    9G Linux LVM
        ```

        Take note of the following details that we will need for the later steps:

        - The number of the partition we wish to resize (i.e. `3` for `/dev/sda3`)
        - The type of the partition we wish to resize (i.e. `Linux LVM`)

   - Enter the following to delete the partition we wish to resize:

        ```sh
        d
        ```

        When prompted for the partition number, enter the number of the partition we wish to resize (i.e. `3`):

        ```sh
        3
        ```

        Sample output:

        ```
            Partition 3 has been deleted.
        ```

   - Enter the following to create a new partition:

        ```sh
        n
        ```

        When prompted for the partition number, enter the number of the partition we had deleted and wish to recreate (i.e. `3`):

        ```sh
        3
        ```

        When prompted for the start of the `First sector`, press <kbd>Enter</kbd> to leave as default.

        When prompted for the end of the `Last sector`, press <kbd>Enter</kbd> to leave as default to use the full available space.

        If asked if we wish to remove the `LVM2_member` signature, refuse it by entering:

        ```sh
        N
        ```

   - Enter the following to configure the partition type:

        ```sh
        t
        ```

        When prompted for the partition number, enter the number of the partition we had just created (i.e. `3`):

        ```sh
        3
        ```

        When asked for the partition type, we are supposed to enter the **alias** for our partition type (i.e. `Linux LVM`). To find the alias for our partition type, run the following command:

        ```sh
        L
        ```

        Sample output:

        ```
            43 Linux LVM                      E6D6D379-F507-44C2-A23C-238F2A3DF928
        ```

        In this example, the alias for our `Linux LVM` partition is `43`. Enter the alias accordingly:

        ```sh
        43
        ```

   - Print the partition table to verify our new partition:

        ```sh
        p
        ```

        Sample output:

        ```
            Disk /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectors
            Disk model: QEMU HARDDISK
            Units: sectors of 1 * 512 = 512 bytes
            Sector size (logical/physical): 512 bytes / 512 bytes
            I/O size (minimum/optimal): 512 bytes / 512 bytes
            Disklabel type: gpt
            Disk identifier: 123E4567-E89B-12D3-A456-426614174000

            Device       Start      End  Sectors  Size Type
            /dev/sda1     2048  1050623  1048576  512M EFI System
            /dev/sda2  1050624  2050047   999424  488M Linux filesystem
            /dev/sda3  2050048 52426751 50376704   24G Linux LVM
        ```

        Ensure that the new partition has been created with the correct `Size` and `Type`.

   - Enter the following to write our changes to the disk:

        ```sh
        w
        ```

        Sample output:

        ```
            The partition table has been altered.
            Syncing disks.
        ```

3. Reboot the system if necessary to apply the partition changes:

    ```sh
    sudo reboot now
    ```

4. Resize the physical volume of our newly resized partition (i.e. `/dev/sda3`):

    ```sh
    sudo pvresize /dev/sda3
    ```

    Sample output:

    ```
        Physical volume "/dev/sda3" changed
        1 physical volume(s) resized or updated / 0 physical volume(s) not resized
    ```

5. Resize the logical volume (i.e. `debian--server--vg-root`) and its partition (i.e. `/dev/sda3`):

    ```sh
    sudo lvresize /dev/mapper/debian--server--vg-root /dev/sda3
    ```

    Sample output:

    ```
        Size of logical volume debian-server-vg/root changed from 8.06 GiB (2064 extents) to 23.06 GiB (5904 extents).
        Logical volume debian-server-vg/root successfully resized.
    ```

    **Alternatively**, you could also use this command to simply extend the size of the logical volume (i.e. `debian--server--vg-root`):

    ```sh
    sudo lvextend -l +100%FREE /dev/mapper/debian--server--vg-root
    ```

6. Resize the filesystem of the logical volume (i.e. `debian--server--vg-root`):

    ```sh
    sudo resize2fs /dev/mapper/debian--server--vg-root
    ```

    Sample output:

    ```
        resize2fs 1.47.0 (5-Feb-2023)
        Filesystem at /dev/mapper/debian--server--vg-root is mounted on /; on-line resizing required
        old_desc_blocks = 2, new_desc_blocks = 3
        The filesystem on /dev/mapper/debian--server--vg-root is now 6045696 (4k) blocks long.
    ```

7. Verify the new size of the logical volume (i.e. `debian--server--vg-root`) by checking the filesystem size:

    ```sh
    df -h
    ```

    Sample output:

    ```
        Filesystem                           Size  Used Avail Use% Mounted on
        udev                                 1.7G     0  1.7G   0% /dev
        tmpfs                                338M  696K  337M   1% /run
        /dev/mapper/debian--server--vg-root   23G  2.1G   20G  10% /
        tmpfs                                1.7G     0  1.7G   0% /dev/shm
        tmpfs                                5.0M     0  5.0M   0% /run/lock
        /dev/sda2                            456M   93M  339M  22% /boot
        /dev/sda1                            511M  5.9M  506M   2% /boot/efi
        tmpfs                                338M     0  338M   0% /run/user/1000
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
