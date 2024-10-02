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
    - [Create Container Template](#create-container-template)
    - [Create VM From Container Template](#create-vm-from-container-template)
    - [Editing VM Parameters](#editing-vm-parameters)
  - [Backups](#backups)
    - [Description](#description-5)
    - [References](#references-4)
    - [Creating Backup](#creating-backup)
    - [Restoring Backup](#restoring-backup)
    - [Exporting Backup](#exporting-backup)

## References

- [Proxmox](https://www.proxmox.com/en)

---

## Setup

### Description

This details the process of setting up Proxmox including installation and configuration.

### References

- [Let's Install Proxmox 8.0!](https://youtu.be/sZcOlW-DwrU)

### Installation

1. Boot into the Proxmox Virtual Environment installer ISO off of a USB stick.

2. In the Proxmox Virtual Environment welcome page, select the **Install Proxmox VE (Graphical)** option.

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

8. In the **Summary** page, review your installation options, check the **Automatically reboot after successful installation** box, and click the **Install** button.

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
- [Enable SSH](ssh.md#enable-remote-access) on the VM
- **(Optional)** Change the default SSH port on the VM
- **(Optional)** Disable root login for SSH on the VM
- **(Optional)** Disable password authentication for SSH on the VM
- **(Optional)** Set up a firewall on the VM (allow traffic to the new SSH port if it's been updated)

For more detailed instructions on configuring a Linux based virtual machine, please refer to the [Linux configuration](linux.md#configuration) topic according to the guest OS.

### Create Container Template

This details how to create a Container Template out of a virtual machine.

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. Select the target virtual machine (illustrated with an icon of a display) you wish to turn to a Container Template. [Create](#create-vm) the virtual machine if you have not already.

3. In the virtual machine view, expand the **More** dropdown located at the top right corner and select the **Convert to template** option.

### Create VM From Container Template

This details how to create a virtual machine or Container Template from a backup.

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target Container Template (illustrated with an icon of a file and display) you wish to use as a base for your new virtual machine.

3. In the Container Template view, expand the **More** dropdown located at the top right corner and select the **Clone** option.

4. In the **Clone VM Template** window, configure the following options:

   - Target node: Set this to your Proxmox node (i.e. `proxmox`)
   - VM ID: Set this to an unused index (i.e. `101`)
   - Name: Set a suitable name for the VM (i.e. `my-vm.example.com`)
   - Mode: Expand the dropdown and select the `Full Clone` option
   - **(Target Storage)**: Expand the dropdown and select the storage where you wish to install the VM on (Defaults to `Same as source`)

   Click the **Clone** button.

5. Wait for the cloning process to complete (i.e. when the lock icon on the newly created VM disappears).

6. Once the VM has been created, you may start using the VM by selecting the VM entry under the Proxmox node and click the **Start** button in its corresponding view.

### Editing VM Parameters

This details how to edit an existing virtual machine or Container Template.

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target virtual machine (illustrated with an icon of a display) or Container Template (illustrated with an icon of a file and display) you wish to edit.

3. In the virtual machine view, go through any of the available sections such as **Hardware**, **Cloud-Init**, or **Options** and find parameters you wish to update.

4. To update a VM parameter, select to highlight the particular parameter (i.e. **Memory**) and click the **Edit** button.

5. In the parameter's **Edit** window, make any changes to the parameter value. Some parameters may have an **Advanced** option box you are able to check to enable.

6. Click the **OK** button to save the changes.

---

## Backups

### Description

This details topics pertaining backups on Proxmox.

### References

- [ProxMox - Migration, Backup, and Restoration Tutorial](https://youtu.be/BkVi2vRB75Q)

### Creating Backup

This details the process of creating backups of a virtual machine or Container Template on Proxmox:

1. Launch the Proxmox Virtual Environment web interface on a web browser.

2. On the left-hand side of the web interface, select the target virtual machine (illustrated with an icon of a display) or Container Template (illustrated with an icon of a file and display) you wish to create a backup of.

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

This details how to restore a virtual machine or Container Template from a backup.

1. If you wish to create a new virtual machine or Container Template from a backup:

   1. On the left-hand side of the web interface, select the target storage which contains the backup you wish to use to restore (i.e. `local`).
   2. In the storage view, click the **Backups** menu option.

2. **Alternatively**, if you wish to overwrite an existing virtual machine or Container Template from a backup:

   1. On the left-hand side of the web interface, select the target storage which contains the backup you wish to use to restore (i.e. `local`).
   2. In the storage view, click the **Backups** menu option.

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

TODO
