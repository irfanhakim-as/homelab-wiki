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
    - [References](#references-2)
    - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Update Hostname](#update-hostname)
  - [Storage](#storage)
    - [Description](#description-4)
    - [References](#references-3)
    - [Partition Storage](#partition-storage)
    - [Mount Storage](#mount-storage)
    - [Resize Storage](#resize-storage)
  - [User Management](#user-management)
    - [Description](#description-5)
    - [Create User](#create-user)
    - [Create Group](#create-group)
    - [Add User to Group](#add-user-to-group)
  - [Sudo](#sudo)
    - [Description](#description-6)
    - [References](#references-4)
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

11. For systems running on a [Raspberry Pi](raspberry-pi.md) (i.e. Raspberry Pi OS), password prompts may not be required when using `sudo` by default.

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
        -r--r----- 1 root root   36 May 13 08:14 010_pi-nopasswd
        -r--r----- 1 root root 1096 Jun 27  2023 README
        ```

        In this example, the configuration file is `010_pi-nopasswd`.

    - Update the configuration file (i.e. `010_pi-nopasswd`) to change this behaviour:

        ```sh
        sudo nano /etc/sudoers.d/010_pi-nopasswd
        ```

        Sample configuration:

        ```
        <user> ALL=(ALL) NOPASSWD:ALL
        ```

        Comment the line with the `NOPASSWD:ALL` option to prevent it from allowing the `<user>` user from using `sudo` without a password:

        ```diff
        - <user> ALL=(ALL) NOPASSWD:ALL
        + #<user> ALL=(ALL) NOPASSWD:ALL
        ```

        **Alternatively**, if you think it is safe, you may also delete the configuration file entirely:

        ```sh
        sudo rm -f /etc/sudoers.d/010_pi-nopasswd
        ```

12. Clear the VM's Bash history:

    ```sh
    history -c
    cat /dev/null > ~/.bash_history && history -c && history -w
    ```

    > [!NOTE]  
    > There should be no other active sessions on the VM while doing this.

13. Reboot the system to apply all changes:

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

- [Setting Up IPv6 Networking on Debian 12](https://reintech.io/blog/setting-up-ipv6-networking-debian-12)

### Set Static IP and Update DNS

> [!IMPORTANT]  
> On systems that use NetworkManager to manage their networking (i.e. Raspberry Pi OS), follow [this guide](networkmanager.md#set-static-ip-and-update-dns) instead.

> [!WARNING]  
> The IPv6 portion of this guide is **EXPERIMENTAL** - more testing and contributions from actual experts are required.

This details the process of setting a static IP address and updating the DNS server on a system.

1. Get some information of the existing network configuration:

   - Get the name of the network interface in use on the system:

        ```sh
        ip -4 route | grep default | grep -oP 'dev \K\w+'
        ```

        Sample output:

        ```sh
          enp6s18
        ```

   - **(Optional)** Get the system's current IPv4 address:

        ```sh
        ip -4 route get 1.1.1.1 | grep -oP 'src \K[0-9.]+'
        ```

        Sample output:

        ```sh
          192.168.0.106
        ```

   - Get the gateway address (IPv4) of the local network:

        ```sh
        ip -4 route | grep default | grep -oP 'via \K[0-9.]+'
        ```

        Sample output:

        ```sh
          192.168.0.1
        ```

   - Based on the system's current IPv4 address (i.e. `192.168.0.106`), get the network address and subnet range:

        ```sh
        ip -4 route | grep <ipv4-address> | grep -oP '[0-9.]+/[0-9]+'
        ```

        For example:

        ```sh
        ip -4 route | grep 192.168.0.106 | grep -oP '[0-9.]+/[0-9]+'
        ```

        Sample output:

        ```sh
          192.168.0.0/24
        ```

        This sample value (i.e. `192.168.0.0/24`) represents the IPv4 network address and CIDR notation, indicating the range of the local network (i.e. from `192.168.0.1` to `192.168.0.254`).

   - Based on the CIDR notation value (i.e. `/24`), get the corresponding subnet mask of the local network. In most cases, the subnet mask value (corresponding to the CIDR notation) is as follows:

     - `/24`: `255.255.255.0`
     - `/16`: `255.255.0.0`
     - `/8`: `255.0.0.0`

2. **(Optional)** Get some additional information of the existing network configuration pertaining to IPv6:

   - **(Optional)** Get the system's current IPv6 address:

        ```sh
        ip -6 route get 2606:4700:4700::1001 | grep -oP 'src \K[0-9a-f:]+'
        ```

        Sample output:

        ```sh
          2001:db8:1234:5678:a1c2:d3e4:f567:89ab
        ```

   - Get the gateway address (IPv6) of the local network:

        ```sh
        ip -6 route | grep default | grep -oP 'via \K[0-9a-f:]+'
        ```

        Sample output:

        ```sh
          fe80::a1b2:c3d4:e5f6:7890
        ```

   - Based on the network interface in use on the system (i.e. `enp6s18`), get the network address and prefix length:

        ```sh
        ip -6 route | grep <network-interface> | grep -E '^[0-9a-f:]+/[0-9]+.*proto (kernel|ra)' | grep -v fe80 | grep -oP '^[0-9a-f:]+/[0-9]+'
        ```

        For example:

        ```sh
        ip -6 route | grep enp6s18 | grep -E '^[0-9a-f:]+/[0-9]+.*proto (kernel|ra)' | grep -v fe80 | grep -oP '^[0-9a-f:]+/[0-9]+'
        ```

        Sample output:

        ```sh
          2001:db8:1234:5678::/64
        ```

        This sample value (i.e. `2001:db8:1234:5678::/64`) represents the IPv6 network prefix, indicating the network portion of the address. The `/64` means the first 64 bits identify the network, and the remaining 64 bits are for host addresses within that network.

   - For IPv6, there is no separate subnet mask - the prefix length (i.e. `/64`) is used directly in the configuration. Common IPv6 prefix lengths:

     - `/64`: Standard for most local networks
     - `/56`: Sometimes used for home networks with multiple subnets
     - `/48`: Typically assigned to organisations

3. Backup the existing network configuration file:

    ```sh
    sudo cp /etc/network/interfaces /etc/network/interfaces.bak
    ```

4. Update the networking configuration to set a static IP address and update the DNS server(s):

   - Update the network configuration file:

        ```sh
        sudo nano /etc/network/interfaces
        ```

        Sample original configuration which uses DHCP to dynamically assign an IP address:

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

   - Make the following changes to the configuration to set a static IPv4 address and update the DNS server(s):

        ```diff
          # The primary network interface
        - allow-hotplug <network-interface>
        - iface <network-interface> inet dhcp
        + auto <network-interface>
        + iface <network-interface> inet static
        +   address <ipv4-address>
        +   netmask <subnet-mask>
        +   gateway <ipv4-gateway>
        +   dns-nameservers <ipv4-dns1> <ipv4-dns2>
        ```

        Sample updated configuration:

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

   - **(Optional)** Make the following changes to the configuration to set a static IPv6 address and update the DNS server(s):

        ```diff
          # The primary network interface
          auto <network-interface>
          iface <network-interface> inet static
            address <ipv4-address>
            netmask <subnet-mask>
            gateway <ipv4-gateway>
            dns-nameservers <ipv4-dns1> <ipv4-dns2>
        +
        + iface <network-interface> inet6 static
        +   address <ipv6-address>/<ipv6-prefix-length>
        +   gateway <ipv6-gateway>
        +   dns-nameservers <ipv6-dns1> <ipv6-dns2>
        ```

        Sample updated configuration:

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

        iface enp6s18 inet6 static
          address 2001:db8:1234:5678::106/64
          gateway fe80::a1b2:c3d4:e5f6:7890
          dns-nameservers 2606:4700:4700::1111 2001:4860:4860::8888
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

3. Reboot the system to apply the changes:

    ```sh
    sudo reboot now
    ```

---

## Storage

### Description

This details the process of updating certain storage related configurations on the system.

### References

- [How to List All Block Devices in Linux | lsblk Command](https://www.geeksforgeeks.org/lsblk-command-in-linux-with-examples)
- [Fstab Usage](https://wiki.archlinux.org/title/Fstab#Usage)

### Partition Storage

After attaching a new storage device to the system, this details how to partition the new storage device before mounting it:

1. First and foremost, identify the exact storage device that you wish to partition:

    ```sh
    sudo fdisk -l
    ```

    Sample output of the storage device we wish to partition:

    ```
      Disk /dev/sdX: 57.73 GiB, 61991813120 bytes, 121077760 sectors
      Disk model: DataTraveler 3.0
      Units: sectors of 1 * 512 = 512 bytes
      Sector size (logical/physical): 512 bytes / 512 bytes
      I/O size (minimum/optimal): 512 bytes / 512 bytes
      Disklabel type: dos
      Disk identifier: 0x5ac83f1b

      Device     Boot Start       End   Sectors   Size Id Type
      /dev/sdX1  *       63 121077759 121077697  57.7G  b W95 FAT32
    ```

    Take note of the storage device name (i.e. `/dev/sdX`).

2. Delete any existing partition(s) on the storage device (i.e. `/dev/sdX`):

    ```sh
    sudo wipefs -a <storage-device>
    ```

    For example:

    ```sh
    sudo wipefs -a /dev/sdX
    ```

    Sample successful output:

    ```
      /dev/sdX: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
      /dev/sdX: calling ioctl to re-read partition table: Success
    ```

3. Create a new partition on the storage device (i.e. `/dev/sdX`):

   - Start `fdisk` for the storage device (i.e. `/dev/sdX`):

        ```sh
        sudo fdisk <storage-device>
        ```

        For example:

        ```sh
        sudo fdisk /dev/sdX
        ```

   - In the interactive `fdisk` session, enter the following to print the partition table:

        ```sh
        p
        ```

        Sample output:

        ```
          Disk /dev/sdX: 57.73 GiB, 61991813120 bytes, 121077760 sectors
          Disk model: DataTraveler 3.0
          Units: sectors of 1 * 512 = 512 bytes
          Sector size (logical/physical): 512 bytes / 512 bytes
          I/O size (minimum/optimal): 512 bytes / 512 bytes
          Disklabel type: dos
          Disk identifier: 0x5ac83f1b
        ```

        The output should show that there are no partitions on the storage device (i.e. `/dev/sdX`).

   - Enter the following to create a new partition:

        ```sh
        n
        ```

        When prompted for the partition type, for example:

        ```
          Partition type
              p   primary (0 primary, 0 extended, 4 free)
              e   extended (container for logical partitions)
        ```

        Enter the intended partition type or press <kbd>Enter</kbd> to leave as default (i.e. `p`):

        ```sh
        p
        ```

        When prompted for the partition number, enter the appropriate partition number, or press <kbd>Enter</kbd> to leave as default (i.e. `1`):

        ```sh
        1
        ```

        When prompted for the start of the `First sector`, enter the intended value, or press <kbd>Enter</kbd> to leave as default (i.e. `2048`):

        ```sh
        2048
        ```

        When prompted for the end of the `Last sector`, enter the intended value, or press <kbd>Enter</kbd> to leave as default to use the full available space (i.e. `121077759`):

        ```sh
        121077759
        ```

        Sample successful output:

        ```
          Created a new partition 1 of type 'Linux' and of size 57.7 GiB.
        ```

   - Print the partition table to verify our new partition:

        ```sh
        p
        ```

        Sample output:

        ```
          Disk /dev/sdX: 57.73 GiB, 61991813120 bytes, 121077760 sectors
          Disk model: DataTraveler 3.0
          Units: sectors of 1 * 512 = 512 bytes
          Sector size (logical/physical): 512 bytes / 512 bytes
          I/O size (minimum/optimal): 512 bytes / 512 bytes
          Disklabel type: dos
          Disk identifier: 0x5689dd05

          Device     Boot Start       End   Sectors  Size Id Type
          /dev/sdX1        2048 121077759 121075712 57.7G 83 Linux
        ```

        Ensure that the new partition has been created with the correct `Size` and `Type` (i.e. `Linux`).

   - **(Optional)** If the partition should be of a different type, enter the following to configure it:

        ```sh
        t
        ```

        When prompted for the partition number, enter the number of the partition we had just created (i.e. `1`):

        ```sh
        1
        ```

        When asked for the partition type, we are supposed to enter the intended partition type's (i.e. `Linux LVM`) corresponding alias or `Id` (i.e. `8e`). To find the alias for our partition type, run the following command:

        ```sh
        L
        ```

        Sample output:

        ```
          00 Empty            27 Hidden NTFS Win  82 Linux swap / So  c1 DRDOS/sec (FAT-
          01 FAT12            39 Plan 9           83 Linux            c4 DRDOS/sec (FAT-
          02 XENIX root       3c PartitionMagic   84 OS/2 hidden or   c6 DRDOS/sec (FAT-
          03 XENIX usr        40 Venix 80286      85 Linux extended   c7 Syrinx
          04 FAT16 <32M       41 PPC PReP Boot    86 NTFS volume set  da Non-FS data
          05 Extended         42 SFS              87 NTFS volume set  db CP/M / CTOS / .
          06 FAT16            4d QNX4.x           88 Linux plaintext  de Dell Utility
          07 HPFS/NTFS/exFAT  4e QNX4.x 2nd part  8e Linux LVM        df BootIt
          08 AIX              4f QNX4.x 3rd part  93 Amoeba           e1 DOS access
          09 AIX bootable     50 OnTrack DM       94 Amoeba BBT       e3 DOS R/O
          0a OS/2 Boot Manag  51 OnTrack DM6 Aux  9f BSD/OS           e4 SpeedStor
          0b W95 FAT32        52 CP/M             a0 IBM Thinkpad hi  ea Linux extended
          0c W95 FAT32 (LBA)  53 OnTrack DM6 Aux  a5 FreeBSD          eb BeOS fs
          0e W95 FAT16 (LBA)  54 OnTrackDM6       a6 OpenBSD          ee GPT
          0f W95 Ext'd (LBA)  55 EZ-Drive         a7 NeXTSTEP         ef EFI (FAT-12/16/
          10 OPUS             56 Golden Bow       a8 Darwin UFS       f0 Linux/PA-RISC b
          11 Hidden FAT12     5c Priam Edisk      a9 NetBSD           f1 SpeedStor
          12 Compaq diagnost  61 SpeedStor        ab Darwin boot      f4 SpeedStor
          14 Hidden FAT16 <3  63 GNU HURD or Sys  af HFS / HFS+       f2 DOS secondary
          16 Hidden FAT16     64 Novell Netware   b7 BSDI fs          f8 EBBR protective
          17 Hidden HPFS/NTF  65 Novell Netware   b8 BSDI swap        fb VMware VMFS
          18 AST SmartSleep   70 DiskSecure Mult  bb Boot Wizard hid  fc VMware VMKCORE
          1b Hidden W95 FAT3  75 PC/IX            bc Acronis FAT32 L  fd Linux raid auto
          1c Hidden W95 FAT3  80 Old Minix        be Solaris boot     fe LANstep
          1e Hidden W95 FAT1  81 Minix / old Lin  bf Solaris          ff BBT
          24 NEC DOS
        ```

        Then, as an example, to configure the partition as a `Linux LVM` partition, enter the corresponding alias (i.e. `8e`) accordingly:

        ```sh
        8e
        ```

   - Enter the following to write our changes to the disk:

        ```sh
        w
        ```

        Sample output:

        ```
          The partition table has been altered.
          Calling ioctl() to re-read partition table.
          Syncing disks.
        ```

4. Create a filesystem (i.e. `ext4`) on the new partition (i.e. `/dev/sdX1`):

    ```sh
    sudo mkfs.<filesystem> <storage-partition>
    ```

    For example:

    ```sh
    sudo mkfs.ext4 /dev/sdX1
    ```

    Sample output:

    ```
      mke2fs 1.47.0 (5-Feb-2023)
      Creating filesystem with 15134464 4k blocks and 3784704 inodes
      Filesystem UUID: 6b2a4cd7-5e43-4f0b-9c02-9d531d2c4ea1
      Superblock backups stored on blocks:
              32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
              4096000, 7962624, 11239424

      Allocating group tables: done
      Writing inode tables: done
      Creating journal (65536 blocks): done
      Writing superblocks and filesystem accounting information: done
    ```

5. Verify that our partition (i.e. `/dev/sdX1`) has been created with the right filesystem (i.e. `ext4`):

    ```sh
    sudo blkid <storage-partition> | grep -i <filesystem>
    ```

    For example:

    ```sh
    sudo blkid /dev/sdX1 | grep -i ext4
    ```

    Sample output:

    ```
      /dev/sdX1: LABEL="data" UUID="6b2a4cd7-5e43-4f0b-9c02-9d531d2c4ea1" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="a4d2c519-01"
    ```

    Take note of the storage partition UUID (i.e. `6b2a4cd7-5e43-4f0b-9c02-9d531d2c4ea1`) which will be needed for mounting.

6. [Mount the newly created storage partition](#mount-storage) on the system.

### Mount Storage

This details how to mount a storage device on the system:

1. Create a mountpoint directory (i.e. `/mnt/data`) for the storage partition:

    ```sh
    sudo mkdir -p <mountpoint>
    ```

    For example:

    ```sh
    sudo mkdir -p /mnt/data
    ```

2. Test mounting the storage partition (i.e. `/dev/sdX1`) to the mountpoint (i.e. `/mnt/data`):

    ```sh
    sudo mount <storage-partition> <mountpoint>
    ```

    For example:

    ```sh
    sudo mount /dev/sdX1 /mnt/data
    ```

3. If the storage partition was mounted successfully, change ownership of the mountpoint (i.e. `/mnt/data`) to the current user:

    ```sh
    sudo chown $USER: <mountpoint>
    ```

    For example:

    ```sh
    sudo chown $USER: /mnt/data
    ```

    This ensures that the current user has the correct permissions to write to the mounted storage.

4. Test writing to the mountpoint (i.e. `/mnt/data`) to ensure the current user has the correct permissions to write to the storage device:

    ```sh
    touch <mountpoint>/testfile
    ```

    For example:

    ```sh
    touch /mnt/data/testfile
    ```

5.  After verifying that the mounted storage partition is writable, unmount it from its mountpoint (i.e. `/mnt/data`):

    ```sh
    sudo umount <mountpoint>
    ```

    For example:

    ```sh
    sudo umount /mnt/data
    ```

6. After verifying that the mounted storage partition is writable, add a mounting entry to the `fstab` file so that it mounts automatically on boot:

   - Update the system's `fstab` file:

        ```sh
        sudo nano /etc/fstab
        ```

   - Add this generic line to the end of the file:

        ```
        <source>   <mountpoint>   <filesystem>   <options>   <dump>   <fsck>
        ```

   - Update the `<source>` with the unique identifier of the storage disk or partition we wish to mount.

        For example, to identify the storage source by `UUID`, replace `<source>` with the following:

        ```
        UUID=<partition-uuid>
        ```

        In such example, replace `<partition-uuid>` with the UUID of the storage partition (i.e. `6b2a4cd7-5e43-4f0b-9c02-9d531d2c4ea1`):

        ```
        UUID=6b2a4cd7-5e43-4f0b-9c02-9d531d2c4ea1
        ```

   - Update the `<mountpoint>` with the path to the directory the storage device should be mounted to.

        For example, replace `<mountpoint>` with the intended mountpoint (i.e. `/mnt/data`):

        ```
        /mnt/data
        ```

   - Update the `<filesystem>` with the type of filesystem (or mount driver) used to access the source.

        For example, replace `<filesystem>` with the filesystem of the storage partition (i.e. `ext4`):

        ```
        ext4
        ```

   - Update the `<options>` with comma-separated settings that control how the storage is mounted or behaves.

        For example, replace `<options>` with the following common mount options:

        ```
        defaults,noatime
        ```

        In such example, `defaults` sets the default mount options for the filesystem and `noatime` prevents the filesystem from updating the access time of files. Use your own combination of mount options as necessary.

   - Update the `<dump>` with your preference on whether the filesystem should be dumped (i.e. `1`) or not (i.e. `0`).

        For example, replace `<dump>` with the following value to skip dumps:

        ```
        0
        ```

        Filesystem dumps are no longer commonly used and should be skipped in almost all cases.

   - Update the `fsck` with your preference on the order for filesystem checks on boot.

        For example, to perform the filesystem check on a secondary storage device, replace `<fsck>` with the following value:

        ```
        2
        ```

        In most cases, the following values are recommended:

        - Root storage device: `1`
        - Other storage partitions: `2`
        - Storage that does not support or require checking (i.e. remote `cifs` storage): `0`

   - Sample storage mount line after all of the required updates:

        ```
        UUID=6b2a4cd7-5e43-4f0b-9c02-9d531d2c4ea1   /mnt/data   ext4   defaults,noatime   0   2
        ```

   - Save the `fstab` file and [reload the `systemd` manager configuration](systemd.md#reload-systemd-manager-configuration) to apply the changes made.

7. Remount the storage partition to its corresponding mountpoint (i.e. `/mnt/data`) to verify our `fstab` entry is configured correctly:

    ```sh
    sudo mount <mountpoint>
    ```

    For example:

    ```sh
    sudo mount /mnt/data
    ```

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

        When asked for the partition type, we are supposed to enter the intended partition type's (i.e. `Linux LVM`) corresponding alias or `Id` (i.e. `43`). To find the alias for our partition type, run the following command:

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
        Adding new user `foo' to supplemental / extra groups `users' ...
        Adding user `foo' to group `users' ...
    ```

### Create Group

This details how to create a group on the system:

1. With root privileges, run the following command to create a group:

   - Replace `<group>` with the name of the group:

        ```sh
        groupadd <group>
        ```

        For example, if the group name is `bar`:

        ```sh
        groupadd bar
        ```

   - If the group needs a specific group ID (GID), add the `-g` flag to the same command:

        ```sh
        groupadd <group> -g <gid>
        ```

        Replace `<gid>` with the intended GID value for the group (i.e. `1001`).

2. Verify that the group has been created:

   - Replace `<group>` with the name of the group:

        ```sh
        cat /etc/group | grep <group>
        ```

        For example, if the group name is `bar`:

        ```sh
        cat /etc/group | grep bar
        ```

   - Sample output:

        ```
            bar:x:1001:
        ```

        Based on the sample output, this tells that the group (i.e. `bar`) has been created.

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

### References

- [How to fix debian sudo command not found](https://linuxhint.com/how-to-fix-debian-sudo-command-not-found)

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
