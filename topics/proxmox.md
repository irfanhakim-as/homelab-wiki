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
    - [Adding a Device](#adding-a-device)
  - [Backups](#backups)
    - [Description](#description-5)
    - [References](#references-4)
    - [Creating Backup](#creating-backup)
    - [Restoring Backup](#restoring-backup)
    - [Exporting Backup](#exporting-backup)
  - [Migrating to Proxmox](#migrating-to-proxmox)
    - [Description](#description-6)
    - [References](#references-5)
    - [Manual](#manual)
    - [ESXi](#esxi)
    - [Post-Migration](#post-migration)
  - [PCIe Passthrough](#pcie-passthrough)
    - [Description](#description-7)
    - [References](#references-6)
    - [Enablement](#enablement)
    - [Adding to VM](#adding-to-vm)

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

2. Select the target virtual machine (illustrated with an icon of a display) you wish to turn to a Container Template. [Create the virtual machine](#create-vm) if you have not already.

3. In the virtual machine view, expand the **More** dropdown located at the top right corner and select the **Convert to template** option.

### Create VM From Container Template

This details how to create a virtual machine from a [Container Template](#create-container-template).

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

### Adding a Device

This details how to add a device to an existing virtual machine:

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the target virtual machine (illustrated with an icon of a display) you wish to add a device to.

3. In the virtual machine view, navigate to the **Hardware** section.

4. Click the **Add** button to expand the list of device categories you wish to add to the virtual machine such as **Hard Disk**, **CD/DVD Drive**, and **Network Device**.

5. Select the type of device you wish to add (i.e. `PCI Device`).

6. In the prompted form, configure the device accordingly, and click the **Add** button.

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

This details how to restore a virtual machine or Container Template from a backup:

1. If you wish to create a new virtual machine or Container Template from a backup:

   1. On the left-hand side of the web interface, select the target storage which contains the backup you wish to use to restore (i.e. `local`).
   2. In the storage view, click the **Backups** menu option.

2. **Alternatively**, if you wish to overwrite an existing virtual machine or Container Template from a backup:

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

This details how to export a backup of a virtual machine or Container Template:

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

This details how to configure a virtual machine to pass through a PCIe device:

1. [Create a virtual machine](#create-vm-from-container-template) or [update](#editing-vm-parameters) an existing one with the following considerations:

   - **System**:
     - Machine: Expand the dropdown and select the machine type (chipset) that matches your Proxmox node (i.e. `q35`)
     - BIOS: Expand the dropdown and select the right BIOS type according to your selected **Machine** (i.e. `OVMF (UEFI)`)
   - **Memory**:
     - Ballooning Device: Uncheck the box to disable it as required by hardware passthrough which is memory address based

2. Once the virtual machine has been created or updated, [add a PCI Device](#adding-a-device) to the virtual machine with the following configurations:

   - Mapped Device: If you are passing through a device that has shareable resources like certain video cards and network cards, check this box to enable it
   - Raw Device: If you are passing through a device as a whole, check this box to enable it
   - Device: Expand the dropdown and select the PCI device you wish to add

3. [Enter the virtual machine](#enter-the-vm) and verify that the device has been passed through:

    ```sh
    lspci
    ```

    From the output, locate the PCI device you have added.
