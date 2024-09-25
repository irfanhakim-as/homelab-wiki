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
  - [Virtual Machine](#virtual-machine)
    - [Description](#description-3)
    - [Restore VM or Container Template From Backup](#restore-vm-or-container-template-from-backup)
    - [Create VM From Container Template](#create-vm-from-container-template)
    - [Editing VM Parameters](#editing-vm-parameters)

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

## Virtual Machine

### Description

This details matters pertaining to virtual machines on Proxmox.

### Restore VM or Container Template From Backup

This details how to restore a virtual machine or Container Template from a backup.

1. On the left-hand side of the web interface, under **Datacenter**, expand the section belonging to your Proxmox node (i.e. `proxmox`).

2. Select the storage which contains the backup you wish to use (i.e. `local`).

3. In said storage's menu options, click the **Backups** section.

4. Select the backup you wish to restore from (create VM out of) and click the **Restore** button.

5. In the **Restore: VM** window, configure the following:

   - Expand the **Storage** dropdown and select the storage where you wish to install the VM on (i.e. `local-zfs`)
   - Set the **VM** ID to an unused index (i.e. `100`)
   - **(Optional)** Check the **Unique** box to have Proxmox autogenerate unique properties.
   - **(Optional)** Check the **Start after restore** box to have Proxmox start the VM after it has been restored (created).
   - **(Optional)** Use the **Override Settings** section to override any hardware configurations such as the **Name**, **Memory** capacity, number of **Cores**, or **Sockets**.
   - Click the **Restore** button.

6. In the **Restore** console, monitor the **Output** and wait for the task to finish. Once the task has finished successfully, close the console by clicking the corresponding **X** button.

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
