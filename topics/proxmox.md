# Proxmox

## Description

Proxmox Virtual Environment is a complete open-source platform for enterprise virtualization. With the built-in web interface you can easily manage VMs and containers, software-defined storage and networking, high-availability clustering, and multiple out-of-the-box tools using a single solution.

## Directory

- [Proxmox](#proxmox)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Configuration](#configuration)
  - [Updates](#updates)
    - [Description](#description-2)
    - [References](#references-2)
    - [Package Update](#package-update)
  - [Adding ISO](#adding-iso)
    - [Description](#description-3)
    - [Steps](#steps)
  - [Virtual Machine](#virtual-machine)
    - [Description](#description-4)
    - [References](#references-3)
    - [Create VM](#create-vm)
    - [Enter the VM](#enter-the-vm)
    - [VM Configuration](#vm-configuration)
    - [Create VM Template](#create-vm-template)
    - [Create VM From VM Template](#create-vm-from-vm-template)
    - [Editing VM Parameters](#editing-vm-parameters)
    - [Adding a Device to a VM](#adding-a-device-to-a-vm)
  - [Linux Containers (LXC)](#linux-containers-lxc)
    - [Description](#description-5)
    - [References](#references-4)
    - [Create LXC Container](#create-lxc-container)
    - [Create LXC Container Template](#create-lxc-container-template)
    - [Create LXC Container from Container Template](#create-lxc-container-from-container-template)
    - [Editing LXC Container Parameters](#editing-lxc-container-parameters)
    - [Adding a Device to an LXC Container](#adding-a-device-to-an-lxc-container)
    - [GPU Passthrough to LXC Container](#gpu-passthrough-to-lxc-container)
    - [Mount SMB share on LXC Container](#mount-smb-share-on-lxc-container)
    - [Update LXC Container Configuration](#update-lxc-container-configuration)
    - [Persist Configurations in LXC](#persist-configurations-in-lxc)
  - [Backups](#backups)
    - [Description](#description-6)
    - [References](#references-5)
    - [Creating Backup](#creating-backup)
    - [Restoring Backup](#restoring-backup)
    - [Exporting Backup](#exporting-backup)
  - [Migrating to Proxmox](#migrating-to-proxmox)
    - [Description](#description-7)
    - [References](#references-6)
    - [Manual](#manual)
    - [ESXi](#esxi)
    - [Post-Migration](#post-migration)
  - [PCIe Passthrough](#pcie-passthrough)
    - [Description](#description-8)
    - [References](#references-7)
    - [Enablement](#enablement)
    - [Adding to VM](#adding-to-vm)
  - [Updating Configuration](#updating-configuration)
    - [Description](#description-9)
    - [References](#references-8)
    - [Datacenter Configuration](#datacenter-configuration)
  - [Clustering](#clustering)
    - [Description](#description-10)
    - [References](#references-9)
    - [Creating Cluster](#creating-cluster)
    - [Joining Node](#joining-node)
    - [Recommended Configuration](#recommended-configuration)
    - [High Availability (HA)](#high-availability-ha)
    - [Adding QDevice](#adding-qdevice)
  - [Monitoring](#monitoring)
    - [Description](#description-11)
    - [References](#references-10)
    - [Install Netdata Agent](#install-netdata-agent)
    - [Configuring Netdata](#configuring-netdata)
    - [Netdata Parent](#netdata-parent)
    - [Netdata Child](#netdata-child)
    - [Streaming Data](#streaming-data)
    - [Netdata Email Alerts](#netdata-email-alerts)
  - [Storage](#storage)
    - [Description](#description-12)
    - [References](#references-11)
    - [Adding Storage](#adding-storage)
    - [Adding SMB/CIFS Storage](#adding-smbcifs-storage)

## References

- [Proxmox](https://www.proxmox.com/en)

---

## Setup

### Description

This details the process of setting up Proxmox including installation and configuration.

### References

- [Let's Install Proxmox 8.0!](https://youtu.be/sZcOlW-DwrU)
- [ZFS on Linux](https://pve.proxmox.com/wiki/ZFS_on_Linux)

### Installation

1. Boot into the Proxmox Virtual Environment installer ISO off of a USB stick.

2. In the Proxmox Virtual Environment welcome page, select the **Install Proxmox VE (Graphical)** option.

    > [!TIP]  
    > If this process gets stuck at `loading drivers`, restart and opt for the same boot entry but with the [`nomodeset` kernel parameter enabled](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#nomodeset_kernel_param).

3. When prompted with the End User License Agreement (EULA) page, click the **I agree** button.

4. Expand the **Target Harddisk** dropdown and select the intended drive to install Proxmox to (i.e. `/dev/nvme0n1`).

    **Optionally**, you may also click the **Options** button to configure additional **Harddisk options** such as selecting a Filesystem (i.e. `zfs (RAID1)`), etc.

    Click the **Next** button once complete.

5. In the **Location and Time Zone selection** page, set the intended **Country**, **Time zone** and **Keyboard Layout** of your choosing.

6. In the **Administration Password and Email Address** page, set the intended **Password** (with confirmation) and **Email** of your choosing.

7. In the **Management Network Configuration** page, configure the following settings:

   - Expand the **Management Interface** dropdown and select the right wired or wireless interface (i.e. `enp4s0`)
   - **Hostname (FQDN)**: Set a suitable hostname to the Proxmox server (i.e. `proxmox.example.com`)
   - **IP Address (CIDR)**: Set a static IP for the Proxmox server (i.e. `192.168.0.106/24`)
   - **Gateway**: Set the local gateway of your network (i.e. `192.168.0.1`)
   - **DNS Server**: Set the DNS server you wish to use (i.e. `1.1.1.1`)

    Click the **Next** button

8. In the **Summary** page, review your installation options **carefully**, check the **Automatically reboot after successful installation** box, and click the **Install** button.

9. Wait for the installation to complete. Once it has, the Proxmox Virtual Environment web interface will be accessible on a browser at the server's IP address and port 8006 (i.e. `https://192.168.0.106:8006`).

### Configuration

1. Launch the Proxmox Virtual Environment web interface on a web browser.

    > [!TIP]  
    > If met with a **Potential Security Risk Ahead** warning, bypass it by selecting the **Advanced...** and **Accept the Risk and Continue** option.

2. At the prompted **Proxmox VE Login** window, fill in your credentials, check the **Save User name** box, and click the **Login** button.

    > [!TIP]  
    > If met with a **No valid subscription** warning, simply click the **OK** button to dismiss it.

3. If you have no valid [licensing subscription](https://www.proxmox.com/en/proxmox-virtual-environment/pricing) (which is entirely optional), you may want to disable any default `Enterprise` package repositories supplied by Proxmox and replace them with their `No-Subscription` counterparts:

   - On the left-hand side of the web interface, under **Datacenter**, click the name of your Proxmox node (i.e. `proxmox`).

   - In the node's available menu options, expand the **Updates** group and click the **Repositories** section.

   - In the **APT Repositories** page, if they are enabled, select each of these repositories and click the **Disable** button:

     - https://enterprise.proxmox.com/debian/ceph-quincy
     - https://enterprise.proxmox.com/debian/pve

   - After disabling them, click the **Add** button, expand and select a **Repository**, and click the **Add** button to enable it.

      Do so for each of these repositories:

     - `No-Subscription`
     - `Ceph Quincy No-Subscription`

---

## Updates

### Description

This details the process of performing updates on Proxmox.

### References

- [How to update Proxmox safely?](https://forum.proxmox.com/threads/how-to-update-proxmox-safely.107228)

### Package Update

To update packages installed or required by Proxmox, do the following:

1. On the Proxmox server shell, run the following to update the software repositories with the latest packages available:

    ```sh
    apt update
    ```

2. To perform the installation of these latest updates, run the following:

    ```sh
    apt full-upgrade
    ```

    > [!TIP]  
    > Running regular `apt upgrade` is not recommended and may break the Proxmox installation.

---

## Adding ISO

### Description

This details the process of adding an ISO to Proxmox.

### Steps

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the target storage you wish to add an ISO to (i.e. `local`).

3. In the storage view, click the **ISO Images** menu option.

4. Click the **Upload** button.

5. In the **Upload** window, click the **Select File** button.

6. In your system file picker, locate and select the ISO file you wish to upload, and submit by clicking the **Open** button.

7. Back in the **Upload** window, click the **Upload** button.

8. Wait for the upload to complete. Once done, close the **Task viewer** window.

---

## Virtual Machine

### Description

This details matters pertaining to virtual machines on Proxmox.

### References

- [Don’t run Proxmox without these settings!](https://youtu.be/VAJWUZ3sTSI)
- [Virtual Machines Settings](https://pve.proxmox.com/wiki/Qemu/KVM_Virtual_Machines#qm_virtual_machines_settings)

### Create VM

This details how to create a virtual machine from scratch.

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. Click the **Create VM** button found on the top right corner of the web interface.

3. In the **Create: Virtual Machine** window, under the **General** tab, configure as such:

   - Node: Expand the dropdown and select your Proxmox node (i.e. `proxmox`)
   - VM ID: Set this to an unused index (i.e. `101`)
   - Name: Set a suitable name for the VM (i.e. `my-vm.example.com`)

    > [!TIP]  
    > There is also an **Advanced** option you can check to enable.

    Click the **Next** button.

4. Under the **OS** tab, configure as such:

   - Use CD/DVD disc image file (iso): Check this box to use this option
     - Storage: Expand the dropdown and select the storage which contains the ISO you have [added](#adding-iso) to Proxmox (i.e. `local`)
     - ISO image: Expand the dropdown and select the ISO you wish to use (i.e. `Rocky-8.6-x86_64-minimal.iso`)
   - Guest OS:
     - Type: Expand the dropdown and select the type of OS you wish to install (i.e. `Linux`)
     - Version: Expand the dropdown and select the version of the OS you wish to install (i.e. `6.x - 2.6 Kernel`)

    Click the **Next** button.

5. Under the **System** tab, configure as such:

   - Graphic card: Expand the dropdown and select the graphics driver most suited for the VM (i.e. `Default`)
     - `Default`: Fine in most cases especially VMs you wish to use headlessly.
     - `VirtIO-GPU`: Recommended for graphical desktops (i.e. on Windows VMs) (requires manual [installation](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers) on Windows)
   - Machine: Expand the dropdown and select the machine type (chipset) you wish to use (i.e. `q35`)
     - `Default (i400fx)`: Legacy, generally the most compatible.
     - `q35`: Modern, generally compatible with most modern systems (required if you wish to enable PCIE passthrough).
   - SCSI Controller: Expand the dropdown and select the SCSI controller you wish to use (i.e. `VirtIO SCSI single`)
   - Qemu Agent: Check this box to enable the Qemu Agent (requires the `qemu-guest-agent` package to be installed on the VM OS)
   - BIOS: Expand the dropdown and select the BIOS you wish to use (i.e. `OVMF (UEFI)`)
     - `Default (SeaBIOS)`: Legacy, generally the most compatible.
     - `OVMF (UEFI)`: Modern, generally compatible with most modern systems.
   - Add TPM: Generally, you may leave this box unchecked except if your VM OS requires it (i.e. Windows 11)
   - OVMF (UEFI):
     - Add EFI Disk: Check this box to enable it
     - EFI Storage: Expand the dropdown and select the storage where you wish to install the EFI disk on (i.e. `local-zfs`)
     - Pre-Enroll keys: Leave this box checked by default as it may be required by few VM OS (i.e. Windows 11)

    Click the **Next** button.

6. Under the **Disks** tab, configure as such for each disk (i.e. **scsi0**):

   - Bus/Device: Leave as default (i.e. `SCSI` and `0`)
   - Storage: Expand the dropdown and select the storage where you wish to install the disk on (i.e. `local-zfs`)
   - Disk size (GiB): Set the size of the disk (i.e. `10`)
   - Cache: Expand the dropdown and select the cache mode you wish to use (i.e. `Default (No cache)`)
   - Discard: Check this box to enable shrinking the disk image in Trim-enabled guest OSes (some guest OS may also require enabling **SSD emulation** on the drive)
   - IO Thread: Leave this box checked to enable better distribution and utilisation of the underlying storage
   - **(Advanced)** SSD emulation: Check this box if you would like the drive to be presented to the guest OS as an SSD
   - **(Advanced)** Backup: Leave this box checked to include the volume in a backup job

    To add more disk to the VM, simply click the **Add** button and configure it as we did before. Once done, click the **Next** button.

7. Under the **CPU** tab, configure as such:

   - Sockets: Set the number of CPU sockets (i.e. `1`)
   - Cores: Set the number of CPU cores (i.e. `1`)
   - Type: Expand the dropdown and select the CPU type you wish for the VM to use or emulate depending on your CPU and requirements (i.e. `x86-64-v3`)

      > [!TIP]  
      > Refer to the [QEMU CPU Types](https://pve.proxmox.com/wiki/Qemu/KVM_Virtual_Machines#qm_cpu) list for the available CPU types

    Click the **Next** button.

8. Under the **Memory** tab, configure as such:

   - Memory (MiB): Set the RAM capacity to allocate (i.e. `1024`)

    Click the **Next** button.

9. Under the **Network** tab, configure as such:

   - No network device: Leave the box unchecked unless you specifically wish for the VM to not have a network device
   - Bridge: Expand the dropdown and select the network bridge you wish to use (i.e. `vmbr0`)
   - Model: Expand the dropdown and select the network model you wish to use (i.e. `VirtIO (paravirtualized)`)
   - Firewall: Leave the box checked to enable the firewall

    Click the **Next** button.

10. Under the **Confirm** tab, review the VM configuration, optionally enable the **Start after created** option, and click the **Finish** button.

11. [Enter the VM](#enter-the-vm) and go through the OS installation process. Refer to the [Linux installation](linux.md#installation) topic for more installation details for Linux based VMs.

12. Once finished installing, if the guest OS asks you to remove the installation medium before rebooting, from the virtual machine view, navigate to the **Hardware** section.

13. Select the **CD/DVD Drive** containing the installation medium or ISO and click the **Edit** button.

14. Select the **Do not use any media** option and click the **OK** button to remove the ISO from the virtual CD/DVD Drive.

15. Go back to the **Console** and press the <kbd>Enter</kbd> key to reboot into the OS.

16. [Configure](#vm-configuration) the VM as necessary.

### Enter the VM

This details how to enter a virtual machine on Proxmox.

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. Select the target virtual machine (illustrated with an icon of a display) you wish to enter.

3. In the virtual machine view, click the **Start** button located at the top right corner if the VM is not already running.

4. In the **Summary** section of the VM, there should be an **IPs** attribute containing the VM's IP address. If it does not (i.e. says **Guest Agent not running**), it's possible that the VM has not been setup or `qemu-guest-agent` is not installed.

5. If you know the IP of the virtual machine, you may simply access it through [SSH]() from your personal machine for the best convenience and experience.

6. **Alternatively**, if you need to access it directly through Proxmox, navigate to the **Console** section and use the VM that way.

### VM Configuration

This details some generic configuration recommendations for a virtual machine after it's been [created and installed](#create-vm):

- Perform a system update on the VM
- Install the [`qemu-guest-agent`](https://pve.proxmox.com/wiki/Qemu-guest-agent) package on the VM to allow useful Proxmox VM management features
- Enable SSH on the VM
- **(Optional)** Change the default SSH port on the VM
- **(Optional)** Disable root login for SSH on the VM
- **(Optional)** Disable password authentication for SSH on the VM
- **(Optional)** Set up a firewall on the VM (allow traffic to the new SSH port if it's been updated)

For more detailed instructions on configuring a Linux based virtual machine, please refer to the [Linux configuration](linux.md#configuration) topic according to the guest OS.

### Create VM Template

This details how to create a VM Template out of a virtual machine.

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. Select the target virtual machine (illustrated with an icon of a display) you wish to turn to a VM Template. [Create the virtual machine](#create-vm) if you have not already.

3. In the virtual machine view, expand the **More** dropdown located at the top right corner and select the **Convert to template** option.

### Create VM From VM Template

This details how to create a virtual machine from a [VM Template](#create-vm-template).

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target VM Template (illustrated with an icon of a file and display) you wish to use as a base for your new virtual machine.

3. In the VM Template view, expand the **More** dropdown located at the top right corner and select the **Clone** option.

4. In the **Clone VM Template** window, configure the following options:

   - Target node: Set this to your Proxmox node (i.e. `proxmox`)
   - VM ID: Set this to an unused index (i.e. `101`)
   - Name: Set a suitable name for the VM (i.e. `my-vm.example.com`)
   - Mode: Expand the dropdown and select the `Full Clone` option
   - Target Storage: Expand the dropdown and select the storage where you wish to install the VM on (Defaults to `Same as source`)

   Click the **Clone** button.

5. Wait for the cloning process to complete (i.e. when the lock icon on the newly created VM disappears).

6. Once the VM has been created, [enter the VM](#enter-the-vm) or [remote into it using SSH](ssh.md#remotely-access-using-ssh) and [configure the VM](#vm-configuration) as necessary.

### Editing VM Parameters

This details how to edit an existing virtual machine or VM Template.

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target virtual machine (illustrated with an icon of a display) or VM Template (illustrated with an icon of a file and display) you wish to edit.

3. In the virtual machine view, go through any of the available sections such as **Hardware**, **Cloud-Init**, or **Options** and find parameters you wish to update.

4. To update a VM parameter, select to highlight the particular parameter (i.e. **Memory**) and click the **Edit** button.

5. In the parameter's **Edit** window, make any changes to the parameter value. Some parameters may have an **Advanced** option box you are able to check to enable.

6. Click the **OK** button to save the changes.

### Adding a Device to a VM

This details how to add a device to an existing virtual machine:

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target virtual machine (illustrated with an icon of a display) you wish to add a device to.

3. In the virtual machine view, navigate to the **Hardware** section.

4. Click the **Add** button to expand the list of device categories you wish to add to the virtual machine such as **Hard Disk**, **CD/DVD Drive**, and **Network Device**.

5. Select the type of device you wish to add (i.e. `PCI Device`).

6. In the prompted form, configure the device accordingly, and click the **Add** button.

---

## Linux Containers (LXC)

### Description

This details topics pertaining to Linux Containers (LXC) on Proxmox.

### References

- [Linux Container](https://pve.proxmox.com/wiki/Linux_Container)
- [Proxmox Container Toolkit](https://pve.proxmox.com/pve-docs/chapter-pct.html)
- [Guest Operating System Configuration](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_guest_operating_system_configuration)
- [Proxmox LXC - How To Guide - Better Than A VM?](https://youtu.be/xKhWRMj5Nrc)
- [Major and Minor Numbers](https://www.oreilly.com/library/view/linux-device-drivers/0596000081/ch03s02.html)
- [LXC On Proxmox](https://jellyfin.org/docs/general/administration/hardware-acceleration/intel/#lxc-on-proxmox)
- [Split A GPU Between Multiple Computers - Proxmox LXC (Unprivileged)](https://youtu.be/0ZDr5h52OOE)
- [Split A GPU Between Multiple Computers (Written Steps)](https://github.com/JamesTurland/JimsGarage/tree/main/LXC/Jellyfin)
- [NAS Shares On LXC - Unprivileged - Jellyfin Example](https://youtu.be/DMPetY4mX-c)
- [NAS Shares On LXC - Unprivileged (Written Steps)](https://github.com/JamesTurland/JimsGarage/tree/main/LXC/NAS)
- [New LXC Add Option - Device Passthrough](https://github.com/tteck/Proxmox/discussions/2850)
- [Tutorial: Unprivileged LXCs - Mount CIFS shares](https://forum.proxmox.com/threads/tutorial-unprivileged-lxcs-mount-cifs-shares.101795)


### Create LXC Container

This details how to create an LXC Container.

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. If no Container Image (Template) has been added to the Proxmox node, do the following steps:

   - On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

   - Select the target storage which is of type `Directory` (i.e. `local`).

   - In the target storage view, click the **CT Templates** menu option.

   - In the **CT Templates** view, click the **Templates** button.

   - In the list of available **Templates**, select the template you wish to download and use as the base of the new LXC Container Template (i.e. `debian-12-standard`).

   - Click the **Download** button.

   - In the **Task viewer** dialog, wait for the process to complete. Once it has, close it by clicking its corresponding **X** (**Close dialog**) button.

3. Click the **Create CT** button found on the top right corner of the web interface.

4. In the **Create: LXC Container** form, configure as such:

   - Select the **Advanced** checkbox at the bottom of the form to show additional options.

   - **General**:

     - Node: Expand the dropdown and select your Proxmox node (i.e. `proxmox`)
     - CT ID: Set this to an unused index (i.e. `101`)
     - Hostname: Set a suitable, unique hostname for the Container (i.e. `my-container.example.com`)
     - Password: Set a secure password for the Container user
     - Confirm password: Confirm the password you have set for the Container user
     - SSH public key(s): Copy over your public SSH key(s), each separated by a new line
     - Unprivileged container: Leaving this option checked and enabled is recommended as it keeps the Proxmox environment more secure in case of compromises
     - Nesting: Leaving this option checked and enabled in an unprivileged environment exposes procfs and sysfs contents of the host to the guest

      Click the **Next** button.

   - **Template**:

     - Storage: Expand the dropdown and select the storage which contains the Container Image you have added to the Proxmox node (i.e. `local`)
     - Template: Expand the dropdown and select the Container Template you wish to use (i.e. `debian-12-standard.tar.zst`)

      Click the **Next** button.

   - **Disks**:

     - Storage: Expand the dropdown and select the storage where you wish to install the disk on (i.e. `local-zfs`)
     - Disk size (GiB): Set the size of the disk (i.e. `8`)

      To add more disk to the Container, simply click the **Add** button and configure it as needed. Once done, click the **Next** button.

   - **CPU**:

     - Cores: Set the number of CPU cores as required by the container (i.e. `1`)
     - CPU limit: Set how much CPU time the container is allowed to use across all cores (i.e. `1`)
     - CPU units: Set the relative weight or priority for CPU access for the container (i.e. `100`)

      Click the **Next** button.

   - **Memory**:

     - Memory (MiB): Set the RAM capacity to allocate (i.e. `512`)
     - Swap (MiB): Set additional swap memory from the host swap space to allocate (i.e. `512`)

      Click the **Next** button.

   - **Network**:

     - Name: Set an appropriate name for the network or leave it as default (i.e. `eth0`)
     - Bridge: Expand the dropdown and select the network bridge you wish to use (i.e. `vmbr0`)
     - Firewall: Leave the corresponding checkbox checked to enable the firewall
     - IPv4: Select the `DHCP` option, or optionally, the `Static` option instead
     - IPv4/CIDR: Set a unique, static IPv4 address for the Container if you have chosen the `Static` option (i.e. `192.168.0.106/24`)
     - Gateway (IPv4): Set the local gateway of your network if you have chosen the `Static` option (i.e. `192.168.0.1`)

      Click the **Next** button.

   - **DNS**:

     - DNS domain: Leave empty to use the default host settings
     - DNS servers: Leave empty to use the default host settings

      Click the **Next** button.

   - Under the **Confirm** tab, review the Container configuration, optionally enable the **Start after created** option, and click the **Finish** button.

5.  In the **Task viewer** dialog, wait for the process to complete. Once it has, close it by clicking its corresponding **X** (**Close dialog**) button.

6. **TODO**: Once the Container has been created, [enter the Container](#enter-the-vm) or [remote into it using SSH](ssh.md#remotely-access-using-ssh) and [configure the Container](#vm-configuration) as necessary.

### Create LXC Container Template

This details how to create a Container Template out of an LXC Container.

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. Select the target Container (illustrated with an icon of a cube) you wish to turn to a Container Template. [Create the LXC Container](#create-lxc-container) if you have not already.

3. In the Container view, expand the **More** dropdown located at the top right corner and select the **Convert to template** option.

### Create LXC Container from Container Template

This details how to create an LXC Container from a Container Template.

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target Container Template (illustrated with an icon of a file and cube) you wish to use as a base for your new LXC Container.

3. In the Container Template view, expand the **More** dropdown located at the top right corner and select the **Clone** option.

4. In the **Clone CT Template** form, configure the following options:

   - Target node: Set this to your Proxmox node (i.e. `proxmox`)
   - CT ID: Set this to an unused index (i.e. `101`)
   - Hostname: Set a suitable, unique hostname for the Container (i.e. `my-container.example.com`)
   - Mode: Expand the dropdown and select the `Full Clone` option
   - Target Storage: Expand the dropdown and select the storage where you wish to install the Container on (Defaults to `Same as source`)

   Click the **Clone** button.

5. Wait for the cloning process to complete (i.e. when the lock icon on the newly created Container disappears).

6. **TODO**: Once the Container has been created, [enter the Container](#enter-the-vm) or [remote into it using SSH](ssh.md#remotely-access-using-ssh) and [configure the Container](#vm-configuration) as necessary.

### Editing LXC Container Parameters

This details how to edit an existing LXC Container or Container Template.

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target LXC Container (illustrated with an icon of a cube) or Container Template (illustrated with an icon of a file and cube) you wish to edit.

3. In the Container view, go through any of the available sections such as **Resources**, **Network**, **DNS**, or **Options** and find parameters you wish to update.

4. To update a Container parameter, select to highlight the particular parameter (i.e. **Memory**) and click the **Edit** button.

5. In the parameter's **Edit** form, make any changes to the parameter value. Some parameters may have an **Advanced** checkbox you could enable to show additional, hidden options.

6. Click the **OK** button to save the changes.

### Adding a Device to an LXC Container

This details how to add a device to an existing LXC Container:

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target LXC Container (illustrated with an icon of a cube) you wish to add a device to.

3. In the Container view, navigate to the section related to the device you wish to add (i.e. **Resources** or **Network**).

4. Click the **Add** button.

5. If provided with a selection of device types, select the type of device you wish to add (i.e. `Mount Point`).

6. In the prompted form, enable the **Advanced** checkbox to show additional, hidden options if necessary, and configure the device accordingly.

7. Click the **Add** or **Create** button.

### GPU Passthrough to LXC Container

This details two methods, **Easy** **(Recommended)** and **Advanced**, on how to passthrough and share a video device (i.e. GPU or iGPU) from a Proxmox node to one or more LXC Container(s):

1. On the Proxmox node host, identify the video device that you wish to passthrough and share to the LXC Container(s):

   - Run the following command to list all available video device(s) on the Proxmox node host:

      ```sh
      ls -l /dev/dri/by-path
      ```

      Sample output:

      ```
         lrwxrwxrwx 1 root root  8 Feb  7 23:18 pci-0000:07:00.0-card -> ../card0
         lrwxrwxrwx 1 root root 13 Feb  7 23:18 pci-0000:07:00.0-render -> ../renderD128
      ```

   - If you have multiple video devices (i.e. `renderD128` and `renderD129`), take note of each `renderD` device's corresponding "target":

      ```
         pci-0000:07:00.0-render -> ../renderD128
      ```

      Based on the "target" of the device (i.e. `pci-0000:07:00.0-render`), cross-check with the list of available PCI devices on the system using only the `XX:XX.X` portion of the value (i.e. `07:00.0`):

      ```sh
      lspci | grep '07:00.0'
      ```

      Sample output:

      ```
         07:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Cezanne [Radeon Vega Series / Radeon Vega Mobile Series] (rev c9)
      ```

   - Once you have identified the right video device that you wish to share, use the `XX:XX.X` value of the target device (i.e. `07:00.0`) to get its corresponding values specifically:

      ```sh
      ls -l /dev/dri/by-path | grep '07:00.0'
      ```

      Sample output:

      ```
         lrwxrwxrwx 1 root root  8 Feb  7 23:18 pci-0000:07:00.0-card -> ../card0
         lrwxrwxrwx 1 root root 13 Feb  7 23:18 pci-0000:07:00.0-render -> ../renderD128
      ```

   - Take note of the following values of the video device:

     - Numbered `renderD` value of the video device (i.e. `128`)
     - **(Easy)** Numbered `card` value of the video device (i.e. `0`)

2. **(Easy)** On the LXC Container, find out the User ID (UID) of the user that needs access to the video device:

   - Replace `username` with the username of the user that needs access to the video device:

      ```sh
      id -u <username>
      ```

      For example, if the user's username is `root`:

      ```sh
      id -u root
      ```

      Sample output:

      ```
         0
      ```

   - Take note of the following values:

     - Numbered user UID on the LXC Container (i.e. `0`)

3. **(Easy)** Find the Group ID (GID) of groups; `render` and `video` on the LXC Container:

   - On the LXC Container, run the following command to find the `render` and `video` groups:

      ```sh
      grep -E '^(render|video):' /etc/group
      ```

      Sample output:

      ```
         video:x:44:
         render:x:104:
      ```

   - Take note of the following values:

     - Numbered `render` group GID on the LXC Container (i.e. `104`)
     - Numbered `video` group GID on the LXC Container (i.e. `44`)

4. **(Advanced)** Find the GID of groups; `render` and `video` on the Proxmox node host:

   - On the Proxmox node host, run the following command to find the `render` and `video` groups:

      ```sh
      grep -E '^(render|video):' /etc/group
      ```

      Sample output:

      ```
         video:x:44:
         render:x:104:
      ```

   - Take note of the following values:

     - Numbered `render` group GID on the Proxmox node host (i.e. `104`)
     - Numbered `video` group GID on the Proxmox node host (i.e. `44`)

5. **(Advanced)** On the Proxmox node host, get the video device's Major and Minor device numbers:

   - Replace `<renderd-number>` with the `renderD` value of the video device you wish to share:

      ```sh
      ls -l /dev/dri/renderD<renderd-number>
      ```

      For example, if the `renderD` value of the video device is `128`:

      ```sh
      ls -l /dev/dri/renderD128
      ```

      Sample output:

      ```
         crw-rw---- 1 root render 226, 128 Feb  7 23:18 /dev/dri/renderD128
      ```

   - Take note of the following values of the video device:

     - Major device number (i.e. `226`)
     - Minor device number (i.e. `128`)

**(Easy)** This details how to perform the GPU passthrough graphically:

> [!IMPORTANT]  
> This method is only supported on Proxmox PVE Manager version 8.2+

1. Please ensure that you have the following values before proceeding with the rest of the steps:

   - `<card-number>`: Numbered `card` value of the video device (i.e. `0`)
   - `<renderd-number>`: Numbered `renderD` value of the video device (i.e. `128`)
   - `<lxc-uid>`: Numbered user UID on the LXC Container (i.e. `0`)
   - `<lxc-video-gid>`: Numbered `video` group GID on the LXC Container (i.e. `44`)
   - `<lxc-render-gid>`: Numbered `render` group GID on the LXC Container (i.e. `104`)

2. [Add a `Device Passthrough` device](#adding-a-device-to-an-lxc-container) to the LXC Container with the following configuration:

   - Device Path: `/dev/dri/card<card-number>` (i.e. `/dev/dri/card0`)
   - UID in CT: `<lxc-uid>` (i.e. `0`)
   - GID in CT: `<lxc-video-gid>` (i.e. `44`)

3. [Add another `Device Passthrough` device](#adding-a-device-to-an-lxc-container) to the LXC Container with the following configuration:

   - Device Path: `/dev/dri/renderD<renderd-number>` (i.e. `/dev/dri/renderD128`)
   - UID in CT: `<lxc-uid>` (i.e. `0`)
   - GID in CT: `<lxc-render-gid>` (i.e. `104`)

4. Reboot the LXC Container if it was running, to apply the changes.

**(Advanced)** This details how to perform the GPU passthrough manually:

1. On the Proxmox node host, update the GID mapping to grant the root user the permission to map these groups inside the LXC Container later:

   - Update the `/etc/subgid` file:

      ```sh
      sudo nano /etc/subgid
      ```

   - Allow the following groups to be mapped inside the LXC Container by adding the following lines to the file:

      ```diff
        root:100000:65536
      + root:<video-gid>:1
      + root:<render-gid>:1
      ```

      Replace `<video-gid>` and `<render-gid>` with the GID values for the `video` (i.e. `44`) and `render` (i.e. `104`) groups on the Proxmox node host, respectively.

   - Sample resulting file:

      ```
      root:100000:65536
      root:44:1
      root:104:1
      ```

2. On the Proxmox node host, [update the LXC Container's configuration file](#update-lxc-container-configuration):

   - Add each of the following lines to the end of the file:

      ```diff
        arch: amd64
        cores: 2
        features: nesting=1
        hostname: my-container.example.com
        memory: 2048
        net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:06:18:78,ip=dhcp,type=veth
        ostype: debian
        rootfs: local-lvm:vm-101-disk-0,size=8G
        swap: 512
        unprivileged: 1
      + lxc.cgroup2.devices.allow: c <major-device-number>:0 rwm
      + lxc.cgroup2.devices.allow: c <major-device-number>:<minor-device-number> rwm
      ```

      - Replace `<major-device-number>` with the video device's major device number (i.e. `226`).
      - Replace `<minor-device-number>` with the video device's minor device number (i.e. `128`).

   - Add the following line to the end of the file:

      ```diff
        lxc.cgroup2.devices.allow: c 226:0 rwm
        lxc.cgroup2.devices.allow: c 226:128 rwm
      + lxc.mount.entry: /dev/dri/renderD<renderd-number> dev/dri/renderD<renderd-number> none bind,optional,create=file
      ```

      - Replace `<renderd-number>` with the video device's corresponding `renderD` value (i.e. `128`).

   - Add the following line to the end of the file:

      ```diff
        lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
      + lxc.idmap: u 0 100000 65536
      ```

      - This maps the LXC Container's UIDs; `0:65535` to the Proxmox node host's UIDs; `100000:165535`.
      - This standard unprivileged UID mapping allows the full UID range inside the LXC Container, while remaining isolated from the Proxmox node host.

   - Add the following line to the end of the file:

      ```diff
        lxc.idmap: u 0 100000 65536
      + lxc.idmap: g 0 100000 <video-gid>
      ```

      - Replace `<video-gid>` with the GID value for the `video` group on the Proxmox node host (i.e. `44`).
      - This maps the LXC Container's GIDs; `0:(video-gid - 1)` to the Proxmox node host's GIDs; `100000:(100000 + (video-gid - 1))`.
      - According to the sample value, this maps the LXC Container's GIDs; `0:43` to the Proxmox node host's GIDs; `100000:100043`.
      - This mapping is done purposefully to make way for the `video` group (i.e. `44`) to be mapped accordingly next.

   - Add the following line to the end of the file:

      ```diff
        lxc.idmap: g 0 100000 44
      + lxc.idmap: g <video-gid> <video-gid> 1
      ```

      - Replace `<video-gid>` with the GID value for the `video` group on the Proxmox node host (i.e. `44`).
      - According to the sample value, this directly maps the LXC Container's GID, `44` to the Proxmox node host's GID, `44`.
      - This mapping grants the LXC Container access to the Proxmox node host's `video` group, for GPU device sharing.

   - Add the following line to the end of the file:

      ```diff
        lxc.idmap: g 44 44 1
      + lxc.idmap: g (<video-gid> + 1) (100000 + (<video-gid> + 1)) (<lxc-render-gid> - (<render-gid> + 1))
      ```

      - Replace `(<video-gid> + 1)` with the sum of the GID value for the `video` group on the Proxmox node host (i.e. `44`) and `1` (i.e. `45`).
      - Replace `(100000 + (<video-gid> + 1))` with the sum of `100000`, the GID value for the `video` group on the Proxmox node host (i.e. `44`), and `1` (i.e. `100045`).
      - Replace `(<lxc-render-gid> - (<render-gid> + 1))` with the difference between the intended `render` group GID on the LXC Container (i.e. `107`) and the sum of the GID value for the `video` group on the Proxmox node host (i.e. `44`) and `1` (i.e. `62`).
      - According to the sample value, this maps the LXC Container's GIDs; `45:106` to the Proxmox node host's GIDs; `100045:100106`.
      - This mapping resumes the previous unprivileged group mapping for the LXC Container, while making way for the intended `render` group GID on the LXC Container (i.e. `107`) to be mapped to the `render` group (i.e. `104`) on the Proxmox node host.

   - Add the following line to the end of the file:

      ```diff
        lxc.idmap: g 45 100045 62
      + lxc.idmap: g <lxc-render-gid> <render-gid> 1
      ```

      - Replace `<lxc-render-gid>` with the intended GID value for the `render` group on the LXC Container (i.e. `107`).
      - Replace `<render-gid>` with the GID value for the `render` group on the Proxmox node host (i.e. `104`).
      - According to the sample value, this maps the LXC Container's GID, `107` to the Proxmox node host's GID, `104`.
      - This mapping grants the LXC Container access to the Proxmox node host's `render` group, for GPU rendering.

   - Add the following line to the end of the file:

      ```diff
        lxc.idmap: g 107 104 1
      + lxc.idmap: g (<lxc-render-gid> + 1) (100000 + (<lxc-render-gid> + 1)) (65536 - (<lxc-render-gid> + 1))
      ```

      - Replace `(<lxc-render-gid> + 1)` with the sum of the intended `render` group GID on the LXC Container (i.e. `107`) and `1` (i.e. `108`).
      - Replace `(100000 + (<lxc-render-gid> + 1))` with the sum of `100000`, the intended `render` group GID on the LXC Container (i.e. `107`), and `1` (i.e. `100108`).
      - Replace `(65536 - (<lxc-render-gid> + 1))` with the difference between `65536` and the sum of the intended `render` group GID on the LXC Container (i.e. `107`) and `1` (i.e. `65428`).
      - According to the sample value, this maps the LXC Container's GIDs; `108:65535` to the Proxmox node host's GIDs; `100108:165535`.
      - This mapping completes the rest of the unprivileged group mapping for the LXC Container to the Proxmox node host.

   - Sample resulting file:

      ```
      arch: amd64
      cores: 2
      features: nesting=1
      hostname: my-container.example.com
      memory: 2048
      net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:06:18:78,ip=dhcp,type=veth
      ostype: debian
      rootfs: local-lvm:vm-101-disk-0,size=8G
      swap: 512
      unprivileged: 1
      lxc.cgroup2.devices.allow: c 226:0 rwm
      lxc.cgroup2.devices.allow: c 226:128 rwm
      lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
      lxc.idmap: u 0 100000 65536
      lxc.idmap: g 0 100000 44
      lxc.idmap: g 44 44 1
      lxc.idmap: g 45 100045 62
      lxc.idmap: g 107 104 1
      lxc.idmap: g 108 100108 65428
      ```

3. On the Proxmox node host, [add the `root` user to groups](linux.md#add-user-to-group); `render` and `video`.

### Mount SMB share on LXC Container

This details two methods, **Cluster** **(Recommended)** and **Node**, to mount an SMB share on an unprivileged LXC Container:

- **Cluster**: This method mounts the SMB/CIFS share as a Proxmox storage on the entire Proxmox cluster.
- **Node**: This method mounts the SMB/CIFS share individually on a single Proxmox node host.

Choose for yourself one of the aforementioned methods and follow the instructions below accordingly.

1. Add the SMB/CIFS share as storage to the Proxmox cluster or node host individually:

   - **(Cluster)** [Add the SMB/CIFS share as storage](#adding-smbcifs-storage) to the Proxmox cluster with the following considerations:

     - When adding the SMB/CIFS share as storage to the Proxmox cluster, uncheck the **Enable** option to disable and prevent it from being mounted.

     - On any one of your Proxmox node host(s), edit the Proxmox storage configuration file:

         ```sh
         nano /etc/pve/storage.cfg
         ```

         Locate the configuration section of your SMB/CIFS storage, for example:

         ```
         cifs: smb
                  path /mnt/pve/smb
                  server 192.168.0.106
                  share smbshare
                  content snippets
                  prune-backups keep-all=1
                  username smbuser
         ```

     - Add the following mount options to the end of the SMB/CIFS share section:

         ```diff
                  username smbuser
         +        options uid=100000,gid=110000,dir_mode=0770,file_mode=0770
         ```

         Sample resulting file:

         ```
         cifs: smb
                  path /mnt/pve/smb
                  server 192.168.0.106
                  share smbshare
                  content snippets
                  prune-backups keep-all=1
                  username smbuser
                  options uid=100000,gid=110000,dir_mode=0770,file_mode=0770
         ```

     - Navigate back to the [SMB/CIFS storage you have added](#adding-smbcifs-storage) and edit it by selecting the storage and clicking the **Edit** button.

     - Enable the storage to be mounted on all Proxmox node host(s) by checking the **Enable** option.

   - **(Node)** For each Proxmox node host (as necessary), mount the SMB share as you would normally do, including:

     - Creating a directory for the share to be mounted to (i.e. `/mnt/smb`).

     - Adding a mount for the share to the `/etc/fstab` file, including the following options:

       - `uid=100000,gid=110000,dir_mode=0770,file_mode=0770`: Limit read-write access to the share for the UID/GID

     - Mounting the share on the Proxmox node host.

      You may refer to the [Linux Wiki](https://github.com/irfanhakim-as/linux-wiki/blob/master/topics/samba.md#mounting-remote-directory) guide for more details on how to do this.

   - **(Optional)** In either of these methods, if you have trouble with allowing read access for a user in another layer of virtualisation/containerisation (i.e. Proxmox node host -> LXC Container -> Docker Container), you may need to update the `dir_mode` and `file_mode` mount options to `0775` accordingly.

2. Grant access to the SMB/CIFS share for your user in the LXC Container:

   - On the LXC Container, [create a group](linux.md#create-group) (i.e. `lxc-shares`) with GID of `10000` that will grant the permission to mount the SMB share.

   - If you have any user(s) other than `root` that needs access to the share inside the LXC Container, [add them to the group](linux.md#add-user-to-group) you created (i.e. `lxc-shares`).

3. On the Proxmox node host, [update the LXC Container's configuration file](#update-lxc-container-configuration):

   - Add the following line to the end of the file:

      ```diff
        arch: amd64
        cores: 2
        features: nesting=1
        hostname: my-container.example.com
        memory: 2048
        net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:06:18:78,ip=dhcp,type=veth
        ostype: debian
        rootfs: local-lvm:vm-101-disk-0,size=8G
        swap: 512
        unprivileged: 1
      + mp0: <host-mount-path>,mp=<container-mount-path>,ro=1
      ```

      - Replace `<host-mount-path>` with the path to the directory where you have mounted the share on the Proxmox node host (i.e. `/mnt/smb/`).
      - Replace `<container-mount-path>` with the path to where the share should be mounted inside the LXC Container (i.e. `/mnt/data`).
      - **(Optional)** Include the `ro=1` flag to make the share read-only inside the LXC Container.
      - **(Optional)** Adding the `shared=1` flag may be needed to allow migrating the LXC Container between Proxmox nodes.

   - Sample resulting configuration file:

      ```
        arch: amd64
        cores: 2
        features: nesting=1
        hostname: my-container.example.com
        memory: 2048
        net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:06:18:78,ip=dhcp,type=veth
        ostype: debian
        rootfs: local-lvm:vm-101-disk-0,size=8G
        swap: 512
        unprivileged: 1
        mp0: /mnt/smb/,mp=/mnt/data,ro=1
      ```

### Update LXC Container Configuration

This details how to update the configuration of an LXC Container:

1. Ensure that the LXC Container has been shut down.

2. On the Proxmox node host, update the LXC Container's configuration file:

   - Replace `<ct-id>` with the LXC Container's actual CT ID value:

        ```sh
        sudo nano /etc/pve/lxc/<ct-id>.conf
        ```

        For example, assuming that the LXC Container's CT ID is `101`:

        ```sh
        sudo nano /etc/pve/lxc/101.conf
        ```

   - Make your changes to the configuration file accordingly.

3. Save the changes made to the file and start the LXC Container.

### Persist Configurations in LXC

Some configurations inside LXC Containers are managed by Proxmox itself which will override your own changes across reboots. This details how to persist a configuration file inside the Container:

1. In the LXC Container, make your changes to a configuration file accordingly:

   ```sh
   sudo nano /path/to/<configuration-file>
   ```

   For example, if the `/path/to/<configuration-file>` is `/etc/network/interfaces`:

   ```sh
   sudo nano /etc/network/interfaces
   ```

2. After you have completed your configuration changes, make the configuration file persist by creating a `.pve-ignore` file for it:

   ```sh
   sudo touch /path/to/.pve-ignore.<configuration-file>
   ```

   For example, if the `/path/to/<configuration-file>` is `/etc/network/interfaces`:

   ```sh
   sudo touch /etc/network/.pve-ignore.interfaces
   ```

   Doing so will tell Proxmox not to manage or interfere with the configuration file that corresponds to the `.pve-ignore` file.

---

## Backups

### Description

This details topics pertaining backups on Proxmox.

### References

- [ProxMox - Migration, Backup, and Restoration Tutorial](https://youtu.be/BkVi2vRB75Q)

### Creating Backup

This details the process of creating backups of a virtual machine or VM Template on Proxmox:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the target virtual machine (illustrated with an icon of a display) or VM Template (illustrated with an icon of a file and display) you wish to create a backup of.

3. In the virtual machine view, click the **Backup** menu option.

4. Click the **Backup now** button.

5. In the **Backup** window, configure the following:

   - Storage: Expand the dropdown and select the storage where you wish to store the backup to (i.e. `local`)
   - Mode: Expand the dropdown and select the backup mode you wish to use (i.e. `Snapshot`)
   - Protected: Leave this box unchecked as default
   - Compression: Expand the dropdown and select the compression algorithm you wish to use (i.e. `ZSTD`)
   - Notification mode: Expand the dropdown and select the notification mode you wish to use (i.e. `Auto`)
   - Send email to: Add an email address to receive the backup notification or leave it empty as default
   - Notes: Customise the note corresponding to the backup or leave it as default (i.e. `{{guestname}}`)

    Click the **Backup** button.

6. In the **Task viewer** window, wait for the backup to complete. Once it has, close the window by clicking its **X** button.

### Restoring Backup

This details how to restore a virtual machine or VM Template from a backup:

1. If you wish to create a new virtual machine or VM Template from a backup:

   1. On the left-hand side of the web interface, select the target storage which contains the backup you wish to use to restore (i.e. `local`).
   2. In the storage view, click the **Backups** menu option.

2. **Alternatively**, if you wish to overwrite an existing virtual machine or VM Template from a backup:

   1. On the left-hand side of the web interface, select the target virtual machine you wish to overwrite from a backup.
   2. In the storage view, click the **Backup** menu option.

3. Select to highlight the backup you wish to restore from and click the **Restore** button.

4. In the **Restore: VM** window, configure the following:

   - Storage: Leave it as default (i.e. `From backup configuration`) or expand the dropdown and select the storage where you wish to install the VM on (i.e. `local-zfs`)
   - VM: If you are creating a new VM, set the VM ID to an unused index (i.e. `100`)
   - **(Optional)** Unique: Check the box to have Proxmox autogenerate unique properties
   - **(Optional)** Start after restore: Check the box to have Proxmox start the VM after it has been restored (created)
   - **(Optional)** Override Settings: Use this section to override any hardware configurations such as the **Name**, **Memory** capacity, number of **Cores**, or **Sockets**

    Click the **Restore** button.

5. In the **Task viewer** window, monitor the **Output** and wait for the task to finish. Once the task has finished successfully, close it by clicking the corresponding **X** button.

### Exporting Backup

This details how to export a backup of a virtual machine or VM Template:

1. On the left-hand side of the web interface, select the target storage which contains the backup you wish to export (i.e. `local`).

2. In the storage view, click the **Backups** menu option.

3. From the list of available backups, take note the **Name** of the backup you wish to export.

4. From your client system, export the backup to the downloads folder:

    ```sh
    scp root@<proxmox-host>:/var/lib/vz/dump/<backup-name>* ~/Downloads
    ```

   - Replace `<proxmox-host>` with the IP address or hostname of the Proxmox server (i.e. `192.168.0.106`)
   - Replace `<backup-name>` with the name of the backup you wish to export (i.e. `vzdump-qemu-100-2024_09_24-11_15_57.vma.zst`)

---

## Migrating to Proxmox

### Description

This details the process of migrating a virtual machine from an existing hypervisor to Proxmox.

### References

- [New Proxmox Import Wizard for Migrating VMware ESXi VMs](https://youtu.be/H1t6hxCoiZw)
- [Migrate to Proxmox VE](https://pve.proxmox.com/wiki/Migrate_to_Proxmox_VE)

### Manual

This details how to migrate a virtual machine manually to Proxmox:

1. Export the source virtual machine's `*.vmdk` and `*-flat.vmdk` files to your client machine.

2. Send the files to a location accessible to the destination Proxmox node (i.e. `~`):

    ```sh
    scp *.vmdk root@<proxmox-host>:~
    ```

    Replace `<proxmox-host>` with the IP address or hostname of the Proxmox server (i.e. `192.168.0.106`)

3. On the Proxmox node, [create a new VM](#create-vm) with the specifications as close to the source virtual machine as possible, not using any media, and have its default disk **REMOVED**. Take note of the virtual machine's VM ID (i.e. `100`).

4. On the Proxmox server shell, import the `*.vmdk` file to the new target virtual machine:

   - Navigate to where the `*.vmdk` and `*-flat.vmdk` files are located (i.e. `~`):

      ```sh
      cd ~
      ```

   - Import the `*.vmdk` file to the target virtual machine:

      ```sh
      qm disk import <vmid> <vmdk-file> <storage>
      ```

     - Replace `<vmid>` with the ID of the target virtual machine (i.e. `100`)
     - Replace `<vmdk-file>` with the name of the target `*.vmdk` (**NOT** `*-flat.vmdk`) file (i.e. `my-server.vmdk`)
     - Replace `<storage>` with the name of the target storage (i.e. `local-zfs`)

   - If the target storage expects or stores disk images in a certain format, specify the format by adding the `--format` option to the same command:

      ```sh
      qm disk import <vmid> <vmdk-file> <storage> --format <format>
      ```

     - Replace `<format>` with the disk format the target storage expects (i.e. `raw`)

   - The full sample command should look something like this:

      ```sh
      qm disk import 100 my-server.vmdk local-zfs --format raw
      ```

   - If the import was successful, it should return an output similar to the following:

      ```sh
      Successfully imported disk as 'unused0:local-zfs:vm-100-disk-0'
      ```

   - If the source virtual machine has multiple disks, repeat the import process for each disk's `*.vmdk` file. For example:

      ```sh
      qm disk import 100 my-server_1.vmdk local-zfs --format raw
      ```

5. Once the import is done, [update](#editing-vm-parameters) each imported disk (i.e. `Unused Disk 0`) in the **Hardware** section of the target machine:

   - Make any configuration to the **Add: Unused Disk** form if necessary.
   - Click the **Add** button to attach the disk to the target machine.

6. [Update](#editing-vm-parameters) the **Boot Order** in the **Options** section of the target machine and ensure the boot or primary storage device (i.e. `scsi0`) is enabled and first in order.

7. Carry out the [Post-Migration](#post-migration) process to ensure the VM is properly configured.

8. Once the imported VM is up and running, you may delete the imported `*.vmdk` and `*-flat.vmdk` files from the Proxmox node:

    ```sh
    rm -f ~/*.vmdk
    ```

### ESXi

This process details how to migrate a virtual machine from an ESXi hypervisor to Proxmox:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the **Datacenter** menu item.

3. In the Datacenter view, click the **Storage** menu option.

4. In the Storage section, click the **Add** button to expand the dropdown and select the **ESXi** option.

5. In the **Add: ESXi** form, configure the following:

   - ID: Add a unique, descriptive ID for the ESXi storage (i.e. `esxi`)
   - Server: Add the IP address of the ESXi node (i.e. `192.168.0.106`)
   - Username: Add the administrator username of the ESXi node (i.e. `root`)
   - Password: Add the administrator password of the ESXi node
   - Skip Certificate Verification: Check the box to avoid issues with the possibly self-signed certificate

    Click the **Add** button.

6. On the left-hand side of the web interface, select the added ESXi storage (i.e. `esxi`).

7. In the list of **Virtual Guests**, select to highlight the virtual machine you wish to import denoted by their `vmx` file (i.e. `my-server.vmx`), and click the **Import** button.

    > [!WARNING]  
    > Please ensure that the VM you are importing is not in a running state (on the ESXi server)!

8. In the **Import Guest** form, configure the following:

   - **General**:

     - VM ID: Set this to an unused index (i.e. `101`)
     - Name: Set a suitable name for the VM (i.e. `my-vm.example.com`)
     - CPU Type: Expand the dropdown and select the CPU type you wish for the VM to use or emulate depending on your CPU and requirements (i.e. `x86-64-v3`)
     - OS Type: Expand the dropdown and select the type of OS the VM runs (i.e. `Linux`)

   - **Advanced**:

     - SCSI Controller: Expand the dropdown and select the SCSI controller you wish to use (i.e. `VirtIO SCSI single`)
     - Network Interface Model: Expand the dropdown and select the network model you wish to use (i.e. `VirtIO (paravirtualized)`)

   - Configure the rest of the settings as you see fit or leave them as default. Click the **Import** button once done.

9. In the **Task viewer** window, wait for the import to complete. Once it has, close the window by clicking its **X** button.

10. Once the VM has been imported, you will find the VM listed on the left-hand side of the web interface, under the group denoted by the name of your Proxmox server (i.e. `proxmox`). Select the VM.

11. [Edit the VM](#editing-vm-parameters)'s following configurations found in the **Options** menu:

    - Boot Order: Ensure the boot storage device (i.e. `scsi0`) is enabled and first in order
    - QEMU Guest Agent: Check the box to enable the QEMU Guest Agent

12. Carry out the [Post-Migration](#post-migration) process to ensure the VM is properly configured.

### Post-Migration

This process details some configuration options to a virtual machine that has just been imported to Proxmox:

1. [Enter the VM](#enter-the-vm) to do some [configuration](linux.md#configuration) according to the target guest OS.

2. In some cases, you may need to configure the existing network configuration which still expects its old network interface (from ESXi) to be available:

   - Run the following command to determine its new network interface:

      ```sh
      ip link
      ```

   - In this sample output, the right network interface is `enp6s18`:

      ```
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
          link/ether fg:LA:Og:mQ:LQ:7Q brd ff:ff:ff:ff:ff:ff
      ```

      Replace any possible mention of the old network interface (i.e. `ens192`) in the VM's network configuration with the new one.

3. [Install](package-manager.md#install-software) packages required by Proxmox using the system package manager (i.e. `apt`):

   - `qemu-guest-agent`

4. Additionally, you may also [uninstall](package-manager.md#remove-software) packages that are no longer required using the system package manager (i.e. `apt`):

   - `open-vm-tools`

---

## PCIe Passthrough

### Description

This details the process of passing through PCIe devices to a virtual machine in Proxmox.

### References

- [Proxmox 8.0 - PCIe Passthrough Tutorial](https://youtu.be/_hOBAGKLQkI)
- [Written guide](https://drive.google.com/file/d/1rPTKi_b7EFqKTMylH64b3Dg9W0N_XIhO/view)
- [PCI(e) Passthrough](https://pve.proxmox.com/wiki/PCI(e)_Passthrough)
- [Determine which Bootloader is Used](https://pve.proxmox.com/wiki/Host_Bootloader#sysboot_determine_bootloader_used)

### Enablement

This process details how to enable PCIe passthrough in Proxmox:

1. On the Proxmox node, determine the bootloader used:

    ```sh
    efibootmgr -v
    ```

   - If the output shows something like the following:

      ```
      BootCurrent: 0002
      ```

      ```
      Boot0002* Linux Boot Manager    HD(2,GPT,aqxk1iaf-t8fe-2hoz-iz2i-ycetxczlsftm,0x800,0x200000)/File(\EFI\SYSTEMD\SYSTEMD-BOOTX64.EFI)
      ```

      It means that the bootloader is Systemd-boot. Otherwise, assume it is GRUB.

2. Update the kernel command line according to the bootloader used. To do so:

   - Systemd-boot:

      ```sh
      nano /etc/kernel/cmdline
      ```

     - Append the required argument(s) to the end of the first line. For example:

        ```diff
        - root=ZFS=rpool/ROOT/pve-1 boot=zfs
        + root=ZFS=rpool/ROOT/pve-1 boot=zfs sample_arg=sample_value
        ```

     - Save and apply the changes after updating the kernel command line:

        ```sh
        proxmox-boot-tool refresh
        ```

   - GRUB:

      ```sh
      nano /etc/default/grub
      ```

     - Append the required argument(s) to the end of the value of the `GRUB_CMDLINE_LINUX_DEFAULT` variable. For example:

        ```diff
        - GRUB_CMDLINE_LINUX_DEFAULT="quiet"
        + GRUB_CMDLINE_LINUX_DEFAULT="quiet sample_arg=sample_value"
        ```

     - Save and apply the changes after updating the kernel command line:

        ```sh
        update-grub
        ```

3. Enable IOMMU according to your CPU by adding the following value to the kernel command line:

   - Intel:

      ```sh
      intel_iommu=on
      ```

   - AMD:

      ```sh
      amd_iommu=on
      ```

4. Enable IOMMU Passthrough mode for increased performance for supported hardware:

    Add the following option to the kernel command line:

    ```sh
    iommu=pt
    ```

5. Load the following kernel modules to enable support for hardware passthrough:

    ```sh
    nano /etc/modules
    ```

   - Add the following kernel modules to the file:

      ```diff
        # /etc/modules: kernel modules to load at boot time.
        #
        # This file contains the names of kernel modules that should be loaded
        # at boot time, one per line. Lines beginning with "#" are ignored.
        # Parameters can be specified after the module name.
      + vfio
      + vfio_iommu_type1
      + vfio_pci
      + # vfio_virqfd # not needed if on kernel 6.2 or newer
      ```

      Save the changes made to the updated file:

      ```sh
      # /etc/modules: kernel modules to load at boot time.
      #
      # This file contains the names of kernel modules that should be loaded
      # at boot time, one per line. Lines beginning with "#" are ignored.
      # Parameters can be specified after the module name.
      vfio
      vfio_iommu_type1
      vfio_pci
      # vfio_virqfd # not needed if on kernel 6.2 or newer
      ```

   - Refresh the `initramfs` after adding the new kernel modules:

      ```sh
      update-initramfs -u -k all
      ```

   - Verify that the modules we have added are loaded:

      ```sh
      lsmod | grep vfio
      ```

      Sample output:

      ```sh
      vfio_pci               16384  0
      vfio_pci_core          86016  1 vfio_pci
      irqbypass              12288  2 vfio_pci_core,kvm
      vfio_iommu_type1       49152  0
      vfio                   65536  3 vfio_pci_core,vfio_iommu_type1,vfio_pci
      iommufd                94208  1 vfio
      ```

6. Reboot the Proxmox node for all changes to take effect. After rebooting, verify that all the required features have been enabled:

      ```sh
      dmesg | grep -e DMAR -e IOMMU -e AMD-Vi
      ```

      Sample output:

      ```sh
      [    0.000000] AMD-Vi: Unknown option - 'on'
      [    0.000000] AMD-Vi: Using global IVHD EFR:0xf77ef22294ada, EFR2:0x0
      [    0.325658] pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported
      [    0.327225] AMD-Vi: Extended features (0xf77ef22294ada, 0x0): PPR NX GT IA GA PC GA_vAPIC
      [    0.327244] AMD-Vi: Interrupt remapping enabled
      [    0.327388] AMD-Vi: Virtual APIC enabled
      [    0.327499] perf/amd_iommu: Detected AMD IOMMU #0 (2 banks, 4 counters/bank).
      ```

### Adding to VM

> [!NOTE]  
> This part of the guide assumes that you have [enabled PCIE passthrough](#enablement) on your Proxmox node.

This details how to configure a virtual machine to pass through a PCIe device:

1. First and foremost, identify the PCI device that you wish to pass through, and verify that it is "safe" to do so:

   - Run the following command on the Proxmox node to list all PCI devices together with their IOMMU groups:

      ```sh
      for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%%/*}; printf 'IOMMU group %s ' "${n}"; lspci -nns "${d##*/}"; done
      ```

      For example, the PCI device we wish to pass through is the following SATA controller:

      ```
         IOMMU group 2 07:00.0 SATA controller [0106]: JMicron Technology Corp. JMB58x AHCI SATA controller [197b:0585]
      ```

   - Based on our example, the PCI device we wish to share belongs to IOMMU group `2`.

     - Check to see if the PCI device shares the same IOMMU group with any other (critical) devices:

         ```sh
         for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%%/*}; [[ ${n} -eq <group> ]] && printf 'IOMMU group %s ' "${n}" && lspci -nns "${d##*/}"; done
         ```

         Replace `<group>` with the IOMMU group of the PCI device we wish to share (i.e. `2`).

     - Sample output:

         ```
            IOMMU group 2 00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
            IOMMU group 2 00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
            IOMMU group 2 07:00.0 SATA controller [0106]: JMicron Technology Corp. JMB58x AHCI SATA controller [197b:0585]
         ```

   - Once we have managed to establish that the PCI device is safe to pass through (along with other devices in the same IOMMU group), proceed with the rest of the guide.

2. [Create a virtual machine](#create-vm-from-vm-template) or [update](#editing-vm-parameters) an existing one with the following considerations:

   - **System**:

     - Machine: Expand the dropdown and select the machine type (chipset) that matches your Proxmox node host (i.e. `q35`)
     - BIOS: Expand the dropdown and select the right BIOS type according to your selected **Machine** (i.e. `OVMF (UEFI)`)

   - **Memory**:

     - Ballooning Device: Uncheck the box to disable it as required by hardware passthrough which is memory address based

3. Once the virtual machine has been created or updated, [add a PCI Device to the virtual machine](#adding-a-device-to-a-vm) with the following configurations:

   - Mapped Device: If you are passing through a device that has shareable resources like certain video cards and network cards, check this box to enable it
   - Raw Device: If you are passing through a device as a whole, check this box to enable it
   - Device: Expand the dropdown and select the PCI device you wish to add

4. [Enter the virtual machine](#enter-the-vm) and verify that the device has been passed through:

    ```sh
    lspci
    ```

    From the output, locate the PCI device you have added.

---

## Updating Configuration

### Description

This details the process of updating the configuration of a Proxmox node.

### References

- [Datacenter Configuration](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_options)

### Datacenter Configuration

1. Create a backup of the datacenter configuration file:

   ```sh
   cp /etc/pve/datacenter.cfg /etc/pve/datacenter.cfg.bak
   ```

2. Update the datacenter configuration file:

   ```sh
   nano /etc/pve/datacenter.cfg
   ```

   For example, to update the default GUI language, add the following property:

   ```diff
   + language: en
   ```

   Please refer to the [official documentation](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_options) for the full list of available options.

3. Save the changes made to the file and reboot the Proxmox node.

---

## Clustering

### Description

This details the process of clustering multiple Proxmox nodes together, including a QDevice.

### References

- [ProxMox High Availability Cluster!](https://youtu.be/08b9DDJ_yf4)
- [SMALL Proxmox Cluster Tips | Quorum and QDevices, Oh My! (+ Installing a QDevice on a RasPi)](https://youtu.be/TXFYTQKYlno)
- [QDevice-Net Setup](https://pve.proxmox.com/wiki/Cluster_Manager#_qdevice_net_setup)
- [Default firewall rules](https://pve.proxmox.com/wiki/Firewall#pve_firewall_default_rules)
- [Can a QDevice be outside the local network](https://www.reddit.com/r/Proxmox/comments/1aolhng/can_a_qdevice_be_outside_the_local_network)

### Creating Cluster

TODO

### Joining Node

TODO

### Recommended Configuration

This details some recommended configurations for the Proxmox cluster depending on your setup.

1. If the cluster is using the same NIC for your VMs and the cluster link, [update the datacenter configuration](#datacenter-configuration) with the following property and value:

   ```
   bwlimit: migration=768000
   ```

### High Availability (HA)

This details the steps to create a HA group on a Proxmox cluster:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the **Datacenter** menu item.

3. In the Datacenter view, expand the **HA** group and select the **Groups** menu option.

4. Click the **Create** button to create a new HA group.

5. In the **Create: HA Group** form, configure the following:

   - ID: Add a unique, descriptive ID for the HA group (i.e. `proxmox-HA`)
   - Restricted: Leave unchecked to prevent resources from being restricted to nodes defined by the group
   - Nofailback: Leave unchecked to allow services to migrate to a node with a higher priority
   - Comment: Add a descriptive comment or leave empty as default
   - Nodes: From the list of nodes, check the corresponding boxes of each node you wish to add to the HA group

    Click the **Create** button.

This details how to make a VM HA:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the target virtual machine (illustrated with an icon of a display) you wish to make HA.

3. In the virtual machine view, expand the **More** dropdown on the top right side and click the **Manage HA** option.

4. In the **Add: Resource: Container/Virtual Machine** form, configure the following:

   - Group: Expand the dropdown and select the HA group you wish to add the virtual machine to (i.e. `proxmox-HA`)
   - Request State: Expand the dropdown and select the `started` option
   - Comment: Add a descriptive comment or leave empty as default

    Click the **Add** button.

In order for VM HA to work, either a shared storage or a storage replication is required. To setup storage replication:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the target virtual machine (illustrated with an icon of a display) you wish to be replicated.

3. In the virtual machine view, select the **Replication** menu option.

4. Click the **Add** button to setup replication for the VM.

5. In the **Create: Replication Job** form, configure the following:

   - Target: Expand the dropdown and select the node(s) you wish to replicate the VM storage to (i.e. `proxmox-2`)
   - Schedule: Expand the dropdown and select the desired schedule or leave it as default (i.e. `*/15 - Every 15 minutes`)
   - Comment: Add a descriptive comment or leave empty as default
   - Enabled: Check this box to enable replication for the VM

    Click the **Create** button.

6. Either wait for the replication to be scheduled, or select the replication job and click the **Schedule now** button.

### Adding QDevice

1. Prepare an external device (i.e. Raspberry Pi) that will serve as the QDevice. The following are some recommendations and considerations for the QDevice:

   - Any Linux distribution will do, but [Debian 12](debian.md) or [Ubuntu 20.04 LTS](ubuntu.md) is recommended. [Install](linux.md#installation) and [configure](linux.md#configuration) the device accordingly.
   - The QDevice requires a static IPv4 address. This should already be the case if you have configured the QDevice correctly.
   - [Enable SSH](ssh.md#enable-remote-access) on the QDevice.
   - [Update the SSH configuration](ssh.md#configuration) and ensure the following things are in order:

     - `Port`: The SSH port has to be set to its default, `22`.
     - `PermitRootLogin`: Allow root login by setting it to `yes`.
     - `PasswordAuthentication`: Enable password authentication by setting it to `yes`.

   - [Enable the Firewall](firewall.md#enablement) on the QDevice.
   - [Add and apply the following allow rules](firewall.md#adding-allow-rule) on the QDevice:

     - `22/tcp` (SSH)
     - `5403/tcp` (Corosync)

2. On the QDevice itself, [install](package-manager.md#install-software) the `corosync-qnetd` package using the system's package manager (i.e. `apt`).

3. On each Proxmox node on the cluster you wish to add the QDevice to, [install](package-manager.md#install-software) the `corosync-qdevice` package using the system's package manager (i.e. `apt`).

4. Ensure all Proxmox nodes on the cluster are up and running. On only one of the Proxmox nodes, add the QDevice to the cluster by running the following command:

   ```sh
   pvecm qdevice setup <qdevice-ip>
   ```

   Replace `<qdevice-ip>` with the static IPv4 address of the QDevice (i.e. `192.168.0.108`).

5. If the command was successful, check the status of the cluster by running the following on the same Proxmox node:

   ```sh
   pvecm status
   ```

   Sample output:

   ```
   Cluster information
   -------------------
   Name:             my-proxmox-cluster
   Config Version:   7
   Transport:        knet
   Secure auth:      on

   Quorum information
   ------------------
   Date:             Wed Dec 25 16:13:02 2024
   Quorum provider:  corosync_votequorum
   Nodes:            2
   Node ID:          0x00000001
   Ring ID:          1.1b
   Quorate:          Yes

   Votequorum information
   ----------------------
   Expected votes:   3
   Highest expected: 3
   Total votes:      3
   Quorum:           2
   Flags:            Quorate Qdevice

   Membership information
   ----------------------
      Nodeid      Votes    Qdevice Name
   0x00000001          1    A,V,NMW 192.168.0.106 (local)
   0x00000002          1    A,V,NMW 192.168.0.107
   0x00000000          1            Qdevice
   ```

   Ensure that the `Qdevice` is part of the cluster's `Membership information` section and that it has exactly `1` vote.

6. If all is done correctly, as long as not more than one node (including the QDevice) is down, the cluster should remain quorate.

---

## Monitoring

### Description

This details the setup process of a monitoring system on Proxmox using Netdata.

### References

- [Install Netdata with kickstart.sh](https://learn.netdata.cloud/docs/netdata-agent/installation/linux)
- [Boost Proxmox with NetData: Real-Time Monitoring](https://youtu.be/auvVz1D8RGk)
- [Deployment Examples: Parent – Child](https://learn.netdata.cloud/docs/deployment-guides/deployment-examples#parent--child)
- [Configuration steps for deploying Netdata with Observability Centralization Points](https://learn.netdata.cloud/docs/deployment-guides/deployment-with-centralization-points#configuration-steps-for-deploying-netdata-with-observability-centralization-points)
- [Edit a configuration file using edit-config](https://learn.netdata.cloud/docs/netdata-agent/configuration#edit-a-configuration-file-using-edit-config)
- [Binding Netdata to multiple ports](https://learn.netdata.cloud/docs/netdata-agent/configuration/securing-agents/web-server-reference#binding-netdata-to-multiple-ports)
- [Service Control](https://learn.netdata.cloud/docs/netdata-agent/maintenance/service-control)
- [How do I install uuidgen](https://stackoverflow.com/questions/17710958/how-do-i-install-uuidgen)
- [Daemon Configuration Reference](https://learn.netdata.cloud/docs/netdata-agent/configuration/daemon)
- [Email](https://learn.netdata.cloud/docs/alerts-&-notifications/notifications/agent-dispatched-notifications/email)
- [SSMTP](https://wiki.archlinux.org/title/SSMTP)

### Install Netdata Agent

This details the steps to install Netdata Agent on a particular system:

1. On said system, run the following command to install Netdata Agent:

   ```sh
   curl https://get.netdata.cloud/kickstart.sh > /tmp/netdata-kickstart.sh && sh /tmp/netdata-kickstart.sh --stable-channel --disable-telemetry
   ```

### Configuring Netdata

This details the steps to edit a Netdata configuration file on a system with Netdata Agent installed:

1. On said system, run the following to get to your Netdata config directory:

   ```sh
   cd /etc/netdata 2>/dev/null || cd /opt/netdata/etc/netdata
   ```

2. Configure the file you wish to update:

   ```sh
   sudo ./edit-config <config-file>
   ```

   Replace `<config-file>` with the name of the Netdata configuration file you wish to update (i.e. `netdata.conf`). For example:

   ```sh
   sudo ./edit-config netdata.conf
   ```

3. Make your changes to the file with the editor and save it.

4. To apply your changes, [restart](systemd.md#restart-service) the `netdata.service` service.

### Netdata Parent

1. [Create a virtual machine](#create-vm-from-vm-template) or [update](#editing-vm-parameters) an existing one to be designated as the Parent node with the following considerations:

   - **Disks**:
     - Disk size (GiB): `25` (Recommended)
   - **Memory**:
     - Memory (MiB): `3584` (Recommended)

   As for the OS, we recommend [installing](linux.md#installation) (and [configuring](linux.md#configuration)) [Debian](debian.md).

2. Configure the firewall to [allow connections](firewall.md#adding-allow-rule) to the following port(s):

   - Netdata: `19999/tcp`

3. [Install Netdata Agent](#install-netdata-agent) on the Parent node.

4. Add the following [configurations](#configuring-netdata) to the `netdata.conf` config file:

   ```ini
   [db]
      mode = dbengine
      dbengine tier backfill = new
      storage tiers = 3
      dbengine page cache size = 1.4GiB
      # storage tier 0
      update every = 1
      dbengine tier 0 retention size = 12GiB
      # storage tier 1
      dbengine tier 1 update every iterations = 60
      dbengine tier 1 retention size = 4GiB
      # storage tier 2
      dbengine tier 2 update every iterations = 60
      dbengine tier 2 retention size = 2GiB
   [ml]
      # Enabled by default
      # enabled = yes
   [health]
      # Enabled by default
      # enabled = yes
   [web]
      # Enabled by default
      # bind to = *
   ```

   - This configuration is tailored for multiple [tiers of metrics storage](https://learn.netdata.cloud/docs/netdata-agent/database#tiers), for 10 Children nodes:

     - Tier 0: 1s granularity for 1 week
     - Tier 1: 1m granularity for 1 month
     - Tier 2: 1h granularity for 1 year

   - **Optionally**, you could lower resource usage by lowering the data granularity by making these changes:

      ```diff
         # storage tier 0
      -  update every = 1
      +  update every = 30 # update every 30s
         # storage tier 1
      -  dbengine tier 1 update every iterations = 60
      +  dbengine tier 1 update every iterations = 10 # update every 10 tier 0 iterations of 30s = 5m
         # storage tier 2
      -  dbengine tier 2 update every iterations = 60
      +  dbengine tier 2 update every iterations = 12 # update every 12 tier 1 iterations of 5m = 1h
      ```

      This configuration sets the data granularity of tiers; 0, 1, and 2 to `30s`, `5m`, and `1h` respectively.

   - You may also lower the amount of storage space to consume if you are monitoring a lesser number of Children node(s):

      ```diff
         # storage tier 0
      -  dbengine tier 0 retention size = 12GiB
      +  dbengine tier 0 retention size = 2.5GiB
         # storage tier 1
      -  dbengine tier 1 retention size = 4GiB
      +  dbengine tier 1 retention size = 1GiB
         # storage tier 2
      -  dbengine tier 2 retention size = 2GiB
      +  dbengine tier 2 retention size = 0.5GiB
      ```

      Use this configuration for a cluster of only 2 Children nodes.

   - Additionally, you may also reduce the amount of memory to consume by reducing the page cache size:

      ```diff
      -  dbengine page cache size = 1.4GiB
      +  dbengine page cache size = 0.3GiB
      ```

      This amount should be sufficient for a cluster of only 2 Children nodes.

   - Make your own adjustments accordingly based on the number of Children node(s) you have and the data granularity you require.

### Netdata Child

1. First and foremost, ensure that the node has a unique [hostname](linux.md#update-hostname) configured for itself before proceeding with the Netdata Agent installation.

2. [Install Netdata Agent](#install-netdata-agent) on the node.

3. Add the following [configurations](#configuring-netdata) to the `netdata.conf` config file:

   ```ini
   [db]
      # https://github.com/netdata/netdata/blob/master/src/database/README.md
      # none = no retention, ram = some retention in ram
      mode = ram
      # The retention in seconds.
      # This provides some tolerance to the time the child has to find a parent 
      # to transfer the data. For IoT, this can be lowered to 120.
      retention = 1200
      # The granularity of metrics, in seconds.
      # You may increase this to lower CPU resources.
      update every = 1
   [ml]
      # Disable Machine Learning
      enabled = no
   [health]
      # Disable Health Checks (Alerting)
      enabled = no
   [web]
      # Disable the Web Server
      mode = none
   [plugins]
      # Uncomment the following line to disable all external plugins on extreme IoT cases by default.
      # enable running new plugins = no
   ```

   **Optionally**, if you have lowered the (tier 0) data granularity on the Parent node, you may reflect the same changes on the Child node (i.e. `30`):

   ```diff
   -  update every = 1
   +  update every = 30
   ```

      This configuration sets the data granularity of tier 0 to `30s` like you may have configured on the Parent node.

### Streaming Data

This details the steps to stream data from a Child node to the Parent node:

1. After setting up both the [Parent node](#netdata-parent) and [Child node(s)](#netdata-child), on **each** Child node, generate a unique API key:

   ```sh
   uuidgen
   ```

   Sample output:

   ```
   759eff6f-cfc2-45cb-b477-edb130abb413
   ```

   If the command is not installed, [install](package-manager.md#install-software) the `uuid-runtime` package using the system's package manager (i.e. `apt`) accordingly.

2. On **each** Child node, add the following [configurations](#configuring-netdata) to the end of the `stream.conf` config file:

   ```ini
   [stream]
      # Stream metrics to another Netdata
      enabled = yes
      # The IP and PORT of the parent
      destination = <parent-node>:19999
      # The shared API key, generated by uuidgen
      api key = <api-key>
   ```

   Replace `<parent-node>` with the static IP address of the Parent node (i.e. `192.168.0.106`) and `<api-key>` with the API key you generated (i.e. `759eff6f-cfc2-45cb-b477-edb130abb413`) accordingly. For example:

   ```diff
      # The IP and PORT of the parent
   -  destination = <parent-node>:19999
   +  destination = 192.168.0.106:19999
      # The shared API key, generated by uuidgen
   -  api key = <api-key>
   +  api key = 759eff6f-cfc2-45cb-b477-edb130abb413
   ```

3. On the Parent node, add the following [configurations](#configuring-netdata) to the end of the `stream.conf` config file for **each** Child node:

   ```ini
   [<child-api-key>]
     # Accept metrics streaming from other Agents with the specified API key
     enabled = yes
   ```

   Replace `<child-api-key>` with the API key of the Child node (i.e. `759eff6f-cfc2-45cb-b477-edb130abb413`) accordingly. For example:

   ```diff
   - [<child-api-key>]
   + [759eff6f-cfc2-45cb-b477-edb130abb413]
       # Accept metrics streaming from other Agents with the specified API key
       enabled = yes
   ```

   As instructed, do this for each Child node that you have in your cluster. For example, if you have another Child node with an API key of its own (i.e. `09319cf6-2283-48d1-b113-a6589348d216`), your configuration should look like this:

   ```ini
   [759eff6f-cfc2-45cb-b477-edb130abb413]
     # Accept metrics streaming from other Agents with the specified API key
     enabled = yes

   [09319cf6-2283-48d1-b113-a6589348d216]
     # Accept metrics streaming from other Agents with the specified API key
     enabled = yes
   ```

### Netdata Email Alerts

This details the steps to set up email alerts on the Parent node:

1. On the Parent node, [install](package-manager.md#install-software) the `ssmtp` package using the system's package manager (i.e. `apt`).

2. Backup the existing SSMTP configuration file:

   ```sh
   sudo cp /etc/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf.bak
   ```

3. Update the network configuration file:

   ```sh
   sudo nano /etc/ssmtp/ssmtp.conf
   ```

   Add the following configuration to the end of the file:

   ```sh
   root=<email-address>

   mailhub=smtp.gmail.com:465

   rewriteDomain=gmail.com

   TLS_CA_FILE=/etc/ssl/certs/ca-certificates.crt
   UseTLS=Yes
   UseSTARTTLS=No

   AuthUser=<email-address>
   AuthPass=<email-password>
   AuthMethod=LOGIN
   ```

   Replace `<email-address>` and `<email-password>` with your own email address (i.e. `netdata@example.com`) and [app password](https://support.google.com/mail/answer/185833) respectively. You may need to configure these a little differently according to your own email provider.

4. Test to see whether the SSMTP configuration works by sending a test email:

   ```sh
   echo -e "Subject: Test Email\n\nThis is a test email" | sudo sendmail <receiver-email>
   ```

   Replace `<receiver-email>` with the email address you wish to send the test email to (i.e. `admin@example.com`).

5. If the test email was sent successfully, add the following [configurations](#configuring-netdata) to the end of the `health_alarm_notify.conf` config file:

   ```sh
   EMAIL_SENDER="<sender-email>"
   SEND_EMAIL="YES"
   DEFAULT_RECIPIENT_EMAIL="<receiver-email>"
   ```

   Replace `<sender-email>` with your email address (i.e. `netdata@example.com`) and `<receiver-email>` with the email address you wish to send the email alerts to (i.e. `admin@example.com`) accordingly. For example:

   ```sh
   EMAIL_SENDER="netdata@example.com"
   SEND_EMAIL="YES"
   DEFAULT_RECIPIENT_EMAIL="admin@example.com"
   ```

   You may also add multiple recipient emails by separating them with spaces:

   ```sh
   DEFAULT_RECIPIENT_EMAIL="admin@example.com user@example.com"
   ```

6. Verify that the Netdata email alerts are working by sending test alerts as user `netdata`:

   ```sh
   sudo -u netdata /usr/libexec/netdata/plugins.d/alarm-notify.sh test
   ```

---

## Storage

### Description

This details topics pertaining storage including adding a storage pool to the Proxmox cluster.

### References

- [Proxmox VE Storage](https://pve.proxmox.com/pve-docs/chapter-pvesm.html)

### Adding Storage

This details the process of adding a storage pool to the Proxmox cluster:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the **Datacenter** menu item.

3. In the Datacenter view, select the **Storage** menu option.

4. Click the **Add** button to expand it and select the type of storage you wish to add (i.e. `Directory`).

5. In the **Add** form, enable the **Advanced** checkbox to show additional, hidden options if necessary, and configure the storage accordingly.

6. Click the form's corresponding **Add** button to add or create the storage pool.

### Adding SMB/CIFS Storage

This details how to add an SMB/CIFS share as storage to the Proxmox cluster:

1. [Add the `SMB/CIFS` storage](#adding-storage) to the Proxmox cluster, configuring it as follows:

   - ID: Add an appropriate, unique name as the identifier for the storage (i.e. `smb`)
   - Server: Add the static IP address of the SMB/CIFS server (i.e. `192.168.0.106`)
   - Username: Add the username of the SMB/CIFS share user (i.e. `smbuser`)
   - Password: Add the password of the SMB/CIFS share user (i.e. `smbuserpassword`)
   - Share: After filling in the earlier fields, expand the dropdown and select the SMB/CIFS share location (i.e. `smbshare`)
   - Nodes: Expand the dropdown and select the nodes which will have access to the storage, or leave it as default (i.e. `All`)
   - Enable: Check the corresponding checkbox to enable the storage
   - Content: Expand the dropdown and select each option that applies to the kind of files that will be stored on the storage (i.e. `Snippets`)

2. After adding the SMB/CIFS storage, the share should be available on all Proxmox node(s) you have selected, at the following mount point:

   ```
   /mnt/pve/<storage-id>
   ```

   Replace `<storage-id>` with the ID of the SMB/CIFS storage (i.e. `smb`).
