# ESXi

## Description

VMware ESXi (formerly ESX) is an enterprise-class, type-1 hypervisor developed by VMware for deploying and serving virtual computers.

## Directory

- [ESXi](#esxi)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Acquire License](#acquire-license)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Custom ISO](#custom-iso)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)
  - [Installation](#installation)
    - [Description](#description-3)
    - [References](#references-3)
    - [Steps](#steps-2)
    - [Notes](#notes)
  - [Periodic Config Backups](#periodic-config-backups)
    - [Description](#description-4)
    - [References](#references-4)
    - [Steps](#steps-3)
  - [Virtual Machine](#virtual-machine)
    - [Description](#description-5)
    - [Creating a Virtual Machine Template](#creating-a-virtual-machine-template)
    - [Creating a Virtual Machine From a Template](#creating-a-virtual-machine-from-a-template)
    - [Editing a Virtual Machine](#editing-a-virtual-machine)
    - [Add Virtual Disk](#add-virtual-disk)
    - [Expand Virtual Disk](#expand-virtual-disk)
  - [Networking](#networking)
    - [Description](#description-6)
    - [References](#references-5)
    - [Steps](#steps-4)
  - [Storage](#storage)
    - [Description](#description-7)
    - [References](#references-6)
    - [Datastore](#datastore)
    - [Disk Pass-through](#disk-pass-through)

## References

- [VMware ESXi](https://www.vmware.com/products/esxi-and-esx.html)

---

## Acquire License

> [!IMPORTANT]  
> We recommend acquiring a free **VMware vSphere Hypervisor 6 License** to use with ESXi 6.7, but as of the time of writing, the free license is only available for ESXi 7.0 and 8.0.

### Description

This details how to acquire a free license from VMware for ESXi.

### References

- [Free ESXi 8.0 - How to Download and get License Keys](https://www.virten.net/2022/11/free-esxi-8-0-how-to-download-and-get-license-keys)

### Steps

1. Visit the [**Customer Connect Login**](https://customerconnect.vmware.com/login) page.

2. Click the **Sign up now** link under the **SIGN IN** button.

3. In the **Register** page, fill in the required details appropriately, and agree to the terms and conditions.

4. Click the **Register** button.

5. Receive the **Account Activation** email and take note of the 6 digit **Authentication Code**.

6. Back in your browser, you should be redirected to the **Activate your profile** page. Type in your **Authentication Code** in the **Activation Code** field.

7. Click the **VERIFY CODE** button.

8. You will be taken to the **Profile Activated** page after a successful activation, then sent back to the **Customer Connect Login** page.

9. Enter your account credentials and click the **SIGN IN** button.

10. Go to the [**Product Evaluation Center for VMware vSphere Hypervisor 8**](https://customerconnect.vmware.com/en/evalcenter?p=free-esxi8) page.

11. Navigate to the **License & Download** tab.

12. Click the **Register** button.

13. Fill up the **Accept End-User License Agreement** form appropriately, and agree to the terms and conditions.

14. Click the **START FREE TRIAL** button.

15. You will be redirected to the **Product Evaluation Center for VMware vSphere Hypervisor 8** page.

16. Under **License Information** in the **License & Download** tab, copy the **License Key** for **VMware vSphere Hypervisor 8 License**.

17. Repeat steps 10 to 16 for the [**VMware vSphere Hypervisor 7 License**](https://customerconnect.vmware.com/evalcenter?p=free-esxi7) license.

---

## Custom ISO

> [!NOTE]  
> This is only required if you require additional drivers for your hardware, which would be the case for the _recommended_ [Hardware Specification](../courses/hardware.md#hardware-specification).

> [!NOTE]  
> This part of the guide requires using a Windows machine, or a Linux machine with PowerShell installed.

> [!IMPORTANT]  
> We recommend doing this with the ESXi 6.7 ISO, but as of the time of writing, the free license for ESXi 6 is no longer obtainable.

### Description

This details how to create a custom ESXi ISO with additional drivers.

### References

- [Adding Realtek 8111 driver to vSphere 6.7 image](https://www.sysadminstories.com/2018/08/adding-realtek-8111-driver-to-vsphere.html)
- [V-Front Online Depot for VMware ESXi](https://vibsdepot.v-front.de/wiki/index.php/List_of_currently_available_ESXi_packages)

### Steps

1. Download the [VMware vSphere Hypervisor (ESXi) Offline Bundle](https://customerconnect.vmware.com/downloads/details?downloadGroup=ESXI67U3&productId=742) for **VMware vSphere Hypervisor (ESXi) 6.7U3**.

2. Download any required drivers you would need from the **V-Front Online Depot for VMware ESXi** website.

    In this example, we will be downloading and adding the following drivers:

    - [sata-xahci](https://vibsdepot.v-front.de/wiki/index.php/Sata-xahci)
    - [Net55-r8168](https://vibsdepot.v-front.de/wiki/index.php/Net55-r8168)

3. Get into a **PowerShell** session using your system **Terminal** application as an Administrator.

4. Move the **VMware vSphere Hypervisor (ESXi) Offline Bundle 6.7U3**, and drivers; i.e. **sata-xahci** and **Net55-r8168** to the working directory.

5. Install `PowerCli`:

    ```powershell
    Install-Module -Name VMware.PowerCLI -AllowClobber
    Set-PowerCLIConfiguration -InvalidCertificateAction Ignore
    ```

    Verify that `PowerCli` is installed:

    ```powershell
    Get-PowerCLIVersion
    ```

6. Add the **VMware vSphere Hypervisor (ESXi) Offline Bundle 6.7U3** to the `EsxSoftwareDepot`:

    ```powershell
    Add-EsxSoftwareDepot .\update-from-esxi6.7-6.7_update03.zip
    ```

    Replace `.\update-from-esxi6.7-6.7_update03.zip` with the actual path to the **VMware vSphere Hypervisor (ESXi) Offline Bundle 6.7U3** file.

7. Add the drivers to the `EsxSoftwareDepot`:

    ```powershell
    Add-EsxSoftwareDepot .\net55-r8168-8.045a-napi-offline_bundle.zip, .\sata-xahci-1.42-1-offline_bundle.zip
    ```

    > [!TIP]  
    > Multiple files can be added at once by separating them with a comma.

    Replace the paths with the actual paths to each driver file(s).

8. List the name of available image profiles:

    ```powershell
    Get-EsxImageProfile | ft Name
    ```

9. Clone the **standard ESXi image** profile (i.e. `ESXi-6.7.0-20190802001-standard`) to a new **custom image** profile i.e. `ESXi-6.7.0-20190802001-custom` we'll be customising:

    ```powershell
    New-EsxImageProfile -CloneProfile "ESXi-6.7.0-20190802001-standard" -Name "ESXi-6.7.0-20190802001-custom" -Vendor "MyName"
    ```

    Replace `ESXi-6.7.0-20190802001-standard` with the name of the standard image profile.

    Replace `ESXi-6.7.0-20190802001-custom` with the name of the custom image profile.

    Replace `MyName` with the name of the vendor (i.e. your name).

10. Get the name of the driver packages:

    ```powershell
    Get-EsxSoftwarePackage | Where {$_.Vendor -eq "VendorName"} | ft Name
    ```

    Replace `VendorName` with the name of the vendor (i.e. `Realtek` or `VFrontDe`).


11. Add each driver package to the **custom image** profile:

    ```powershell
    Add-EsxSoftwarePackage -ImageProfile "ESXi-6.7.0-20190802001-custom" -SoftwarePackage "net55-r8168", "sata-xahci"
    ```

    Replace `ESXi-6.7.0-20190802001-custom` with the name of the custom image profile.

    Replace `net55-r8168` and `sata-xahci` with the name of the driver packages.

12. Set the acceptance level of the **custom image** profile to `CommunitySupported`:

    ```powershell
    Set-EsxImageProfile -ImageProfile "ESXi-6.7.0-20190802001-custom" -AcceptanceLevel CommunitySupported
    ```

    Replace `ESXi-6.7.0-20190802001-custom` with the name of the custom image profile.

13. Export the **custom image** profile to an ISO file:

    ```powershell
    Export-EsxImageProfile -ImageProfile "ESXi-6.7.0-20190802001-custom" -ExportToIso -FilePath ".\ESXi-6.7.0-20190802001-custom.iso"
    ```

    Replace `ESXi-6.7.0-20190802001-custom` with the name of the custom image profile.

    Replace `.\ESXi-6.7.0-20190802001-custom.iso` with the path to the new custom ISO file.

---

## Installation

> [!NOTE]  
> Correctly configure the Motherboard BIOS prior to installation. Refer to the [Motherboard BIOS](../courses/hardware.md#motherboard-bios) section for more details.

> [!NOTE]  
> It is highly recommended that you are using an SSD for the ESXi installation.

### Description

This details how to install and setup ESXi on a server.

### References

- [VMware ESXi Installation and configuration for AMD Ryzen Part 1](https://youtu.be/glu9Y4m6XpQ)
- [Home VMware ESXi Installation and configuration Part 2](https://youtu.be/mdsO4tRHWPg)

### Steps

Follow the steps detailed in the two videos referenced above.

### Notes

- Use the custom ESXi installer ISO created in the [Custom ISO](#custom-iso) section if you are using the _recommended_ [Hardware Specification](../courses/hardware.md#hardware-specification).

---

## Periodic Config Backups

> [!IMPORTANT]  
> The cron job will (sometimes) disappear after a reboot. In this case, you will need to re-add the cron job.

### Description

This details how to setup periodic configuration backups for ESXi.

### References

- [How to: A Complete Guide to Backing Up and Restoring VMware ESXi Host Configuration](https://community.spiceworks.com/how_to/174408-a-complete-guide-to-backing-up-and-restoring-vmware-esxi-host-configuration)

### Steps

1. Get into VMware ESXi web interface by navigating to the IP address of the ESXi server in your browser.

2. In the **Host** page, locate and click the **Actions** button with the gear cog icon.

3. Hover over the **Services** menu item, and click the **Enable Secure Shell (SSH)** option.

4. Launch a **Terminal** application on your local machine, and remote into the server using the following command:

    ```sh
    ssh root@<ip-address>
    ```

    Replace `<ip-address>` with the actual IP address of the ESXi server.

5. Enter the password for the `root` user when prompted.

6. Once logged in, sync configuration files to the datastore using the following command:

    ```sh
    vim-cmd hostsvc/firmware/sync_config
    ```

7. Perform a backup of the configuration files using the following command:

    ```sh
    vim-cmd hostsvc/firmware/backup_config
    ```

    This command should return a link to download a bundle of the configuration backup.

8. To automate the backup process, create an `esxi-backup` folder to the default datastore (i.e. `datastore1`), and a `backups` folder within it for storing the backups:

    ```sh
    mkdir -p /vmfs/volumes/datastore1/esxi-backup/backups
    ```

9. Create an `esxi-backup.sh` script inside the `esxi-backup` folder for use to perform the backup:

    ```sh
    vi /vmfs/volumes/datastore1/esxi-backup/esxi-backup.sh
    ```

    Content of the script:

    ```sh
    #!/bin/sh
    vim-cmd hostsvc/firmware/sync_config
    vim-cmd hostsvc/firmware/backup_config
    find /scratch/downloads/ -name \*.tgz -exec cp {} /vmfs/volumes/datastore1/esxi-backup/backups/MyEsxiServer_config_backup_$(date +'%Y%m%d_%H%M%S').tgz \;
    ```

    > [!TIP]
    > Replace `MyEsxiServer` with the name of your ESXi server (without whitespaces).

10. Make the script executable:

    ```sh
    chmod +x /vmfs/volumes/datastore1/esxi-backup/esxi-backup.sh
    ```

11. Test the backup script by running it:

    ```sh
    /vmfs/volumes/datastore1/esxi-backup/esxi-backup.sh
    ```

    Verify that the backup file is created and stored in the `backups` folder.

    ```sh
    ls -la /vmfs/volumes/datastore1/esxi-backup/backups
    ```

12. Create a cron job to run the backup script periodically:

    ```sh
    vi /var/spool/cron/crontabs/root
    ```

    Add the following line to the end of the file:

    ```sh
    0 0 * * * /vmfs/volumes/datastore1/esxi-backup/esxi-backup.sh
    ```

    > [!TIP]
    > This will run the backup script every day at midnight (according to the ESXi server's time).

    Save and exit the file using the following options:

    ```sh
    :wq!
    ```

13. Head back to the ESXi web interface, and disable SSH by clicking the **Actions** button with the gear cog icon, hovering over the **Services** menu item, and clicking the **Disable Secure Shell (SSH)** option.

---

## Virtual Machine

### Description

This details matters pertaining to virtual machines on ESXi.

### Creating a Virtual Machine Template

This details how to create a virtual machine to be used as a template or base for other virtual machines.

1. Get into VMware ESXi web interface by navigating to the IP address of the ESXi server in your browser.

2. In the **Virtual Machines** page, locate and click the **Create/Register VM** button.

3. In the **Select creation type** page, select the **Create a new virtual machine** option and click **Next**.

4. In the **Select a name and guest OS** page:

    - Enter a name for the virtual machine to the **Name** field i.e. `ubuntu-server.example.com`.
    - Select `ESXi {VERSION_NUMBER} virtual machine` for **Compatibility**.
    - Select the appropriate OS family for the VM **Guest OS Family** i.e. `Linux`.
    - Select the appropriate OS version for the VM **Guest OS Version** i.e. `Ubuntu Linux (64-bit)`.
    - Click the **Next** button.

5. In the **Select storage** page, select the datastore to be used i.e. `datastore1` and click **Next**.

    > [!TIP]  
    > The datastore will be used for the VM's configuration files and virtual OS disk.

6. In the **Customize settings** page, configure the virtual hardware as minimally as possible (according to the OS requirements):

    - CPU: `1`
    - Memory: `1024 MB`
    - Hard Disk 1: `10 GB`
    - SCSI Controller 0: `LSI Logic Parallel` (change appropriately if needed)
    - USB controller 1: `USB 2.0`
    - Network Adapter 1: `VM Network` (change appropriately if needed)
    - CD/DVD Drive 1:
        - `Datastore ISO File`, Connect: `Enabled`
        - Status: `Connect at power on`
        - CD/DVD Media: `ISO/ubuntu-20.04.6-live-server-amd64.iso` (select the actual ISO file stored in any of your datastores)
        - Controller location: `SATA controller 0`, `SATA (0:0)`
    - Video Card: `Default settings` (change appropriately if needed)

7. Click the **Next** button.

8. In the **Ready to complete** page, review the configuration and click the **Finish** button.

9. In the **Virtual Machines** page, locate and click the newly created virtual machine.

10. In the VM's page, click the **Power on** button.

11. Click the "screen" on the VM's page to open a browser console to the VM.

12. Go through the OS installation process. Refer to the [Linux](linux.md) topic for more installation details for Linux based VMs.

13. After the installation has completed, restart the VM, and in the VM's page, click the **Edit** button.

14. In the VM's **Edit Settings** page, navigate to the **CD/DVD Drive 1** section, and disable the **Connect at power on** option.

15. Click the **Save** button.

16. Back in the VM (through the browser console), press the <kbd>Enter</kbd> key to reboot into the OS (instead of the installer media).

17. On your personal machine, [copy](ssh.md#copy-ssh-keys) over your public SSH key to the VM.

18. Log into the VM with your provided credentials.

19. Perform any additional configuration steps required for the VM. Steps will vary depending on the OS, so again, refer to the [Linux](./linux.md) topic for more configuration details. Recommended configuration steps include:

    - Enabling SSH on the VM.
    - Change the default SSH port on the VM.
    - Disabling password authentication for SSH on the VM.
    - Setting up a firewall on the VM while also allowing traffic to the new SSH port.

20. Once you are done with all of the configurations that will be inherited by future VMs, shut down the VM.

21. In the VM's page, click the **Actions** button with the gear cog icon.

22. Click the **Unregister** option, and click the **Yes** button to confirm.

23. In the left panel of the VMware ESXi web interface, click the **Storage** menu item.

24. In the **Storage** page, click the **Datastores** tab.

25. Click the datastore where the VM's configuration files and virtual OS disk are stored i.e. `datastore1`.

26. Click the **Datastore browser** button.

27. Click the **Create directory** button.

28. Fill in the **Directory name** field with an appropriate name indicating that is a template for a VM of a particular OS i.e. `ubuntu-server-template`.

29. Click the **Create directory** button.

30. Locate the VM's folder which should be named after the VM's name i.e. `ubuntu-server.example.com`.

31. Select the `.vmx` file found in the VM's folder, and click the **Copy** button.

32. In the newly opened **Select destination** window, select the newly created directory i.e. `ubuntu-server-template`, and click the **Copy** button.

33. Repeat the last two steps but for the `.vmdk` file found in the VM's folder.

    > [!TIP]  
    > This might take a little longer depending on the size of the virtual disk (`.vmkd` file).

34. Once you have copied over both the `.vmx` and `.vmdk` files, you may delete the VM's folder by selecting it and clicking the **Delete** button in the **Datastore browser**.

35. Click the **Close** button.

### Creating a Virtual Machine From a Template

This details how to create a virtual machine from a template or base virtual machine.

1. Get into VMware ESXi web interface by navigating to the IP address of the ESXi server in your browser.

2. In the left panel of the VMware ESXi web interface, click the **Storage** menu item.

3. In the **Storage** page, click the **Datastores** tab.

4. Click the datastore where you wish to install the VM to i.e. `datastore1`.

5. Click the **Datastore browser** button.

6. Click the **Create directory** button.

7. Fill in the **Directory name** field with the name you wish to provide for the new VM i.e. `ubuntu-0.example.com`.

8. Click the **Create directory** button.

9. In the **Datastore browser**, locate the template VM's folder in the datastore it was stored to.

10. Select the template VM's folder, and select the `.vmx` file found inside it.

11. Click the **Copy** button.

12. In the newly opened **Select destination** window, select the newly created directory i.e. `ubuntu-0.example.com`, and click the **Copy** button.

13. Repeat the last three steps but for the `.vmdk` file found in the template VM's folder.

    > [!TIP]  
    > This might take a little longer depending on the size of the virtual disk (`.vmkd` file).


14. Once you have copied over both the `.vmx` and `.vmdk` files to the new VM's folder, close the **Datastore browser**.

15. In the **Datastores** tab of the **Storage** page, click the **Register a VM** button.

16. In the newly opened **Register VM** window, select the right datastore, and select the new VM's folder.

17. Select the `.vmx` file found in the new VM's folder, and click the **Register** button.

18. In the **Virtual Machines** page, locate and click the newly created virtual machine.

    > [!TIP]  
    > At this moment, the VM's name will still be the same as the template VM's name.

24. In the new VM's page, click the **Edit** button.

25. In the VM's **Edit Settings** page, navigate to the **VM Options** section, and edit the **VM Name** field to the name you wish to provide for the new VM i.e. `ubuntu-0.example.com`.

26. Make any other changes to the VM's hardware and options as needed, then click the **Save** button.

27. In the VM's page, click the **Power on** button.

28. When prompted with a question asking if you have copied or moved the VM, select the **I Copied It** option, and click the **Answer** button.

29. Click the "screen" to open a browser console to the VM.

30. Log into the VM with your provided credentials.

31. Perform any additional configuration steps required for the VM. Steps will vary depending on the OS, refer to the [Linux](./linux.md) topic for more details. Recommended configuration steps include:

    - Setting a static IP address for the VM.
    - Updating the system's hostname.
    - Getting the system up to date since the VM template was created.

### Editing a Virtual Machine

This details how to edit an existing virtual machine.

1. Get into VMware ESXi web interface by navigating to the IP address of the ESXi server in your browser.

2. Click the **Virtual Machines** menu item from the left panel.

3. In the **Virtual Machines** page, locate and click the virtual machine you wish to edit.

4. In the VM's page, click the **Edit** button.

5. In the **Edit Settings** page, make any changes to the VM's **Virtual Hardware** and **VM Options** as needed.

6. Click the **Save** button to apply the changes.

### Add Virtual Disk

This details how to add an additional virtual disk to a virtual machine.

TODO

### Expand Virtual Disk

This details how to expand the size of an existing disk or add an additional virtual disk to a virtual machine.

1. [Edit the virtual machine](#editing-a-virtual-machine) you wish to expand the disk for:

    - In the **Edit Settings** page, configure the VM's **Virtual Hardware**.
    - Locate the section for the virtual disk you wish to expand i.e. **Hard Disk 1**.
    - In the provided size field, enter the new size for the virtual disk i.e. `20 GB`.
    - Click the **Save** button to apply the changes.

2. Expand the partition on the virtual disk. There are different ways to do this depending on the OS and the partitioning scheme used. To determine the method of doing this on Linux, refer to the [Linux](./linux.md) topic.

---

## Networking

### Description

This details how to add and set up an additional network card to the ESXi server.

### References

- [How to add second nic to vswitch on esx host](https://youtu.be/wXwmDN10PnE)

### Steps

1. Connect the physical network interface card to the ESXi server by way of the onboard PCIe slot.

2. In the VMware ESXi web interface, navigate to the **Networking** menu item from the left panel.

3. In the **Networking** page, click the **Physical NICs** tab.

4. From the list of physical network interface cards, take note of the **Name** of the new network interface card i.e. `vmnic1`.

5. Navigate to the **Virtual switches** tab.

6. Click the **Add standard virtual switch** button.

7. In the **Add standard virtual switch** window:

    - vSwitch Name: Fill in with an appropriate name for the new virtual switch i.e. `vSwitch1`.
    - MTU: Leave as default i.e. `1500`.
    - Uplink 1: Expand the dropdown and select the new network interface card from the available list i.e. `vmnic1`.
    - Click the **Add** button.

8. Optionally, you may also add an uplink redundancy to the newly created virtual switch:

    - In the **Virtual switches** tab, select the newly created virtual switch i.e. `vSwitch1`.
    - Click the **Add uplink** button.
    - In the **Edit standard virtual switch** window, locate the newly added **Uplink 2** section, expand the dropdown, and select any available network interface card (port) from the available list i.e. `vmnic2`.
    - Click the **Save** button.

9. Navigate to the **Port groups** tab.

10. Click the **Add port group** button.

11. In the **Add port group** window:

    - Name: Fill in with an appropriate name for the new port group i.e. `vSwitch1 Network`.
    - VLAN ID: Leave as default i.e. `0`.
    - Virtual switch: Expand the dropdown and select the newly created virtual switch i.e. `vSwitch1`.
    - Click the **Add** button.

---

## Storage

### Description

This details how to add and set up additional storage to the ESXi server including pass-through of physical storage devices.

### References

- [Create a new Datastore in ESXi 6.7 (virtual environment)](https://youtu.be/-0e3jSjO1Ss)
- [ESXi - How to Enable Passthrough mode (VMDirectPath) in #VMware #vSphere #ESXi 6.7](https://youtu.be/zzp9WBCFaK0)

### Datastore

1. Connect the physical storage device to the ESXi server by way of the onboard SATA ports or PCIe M.2 slot.

2. In the VMware ESXi web interface, navigate to the **Storage** menu item from the left panel.

3. In the **Storage** page, click the **Datastores** tab.

4. Click the **New datastore** button.

5. In the **Select creation type** page of the **New datastore** window, select the **Create new VMFS datastore** option, and click the **Next** button.

6. In the **Select device** page:

    - Fill in the **Name** field with an appropriate name for the new datastore i.e. `SSD`.
    - Select the physical storage device you connected to the ESXi server from the available list.
    - Click the **Next** button.

7. In the **Select partitioning options** page:

    - Select the **Use full disk** option.
    - Select the **VMFS 6** version.
    - Click the **Next** button.

8.  In the **Ready to complete** page, review the configuration, and click the **Finish** button.

### Disk Pass-through

1. Connect the physical SATA adapter/card to the ESXi server by way of the onboard PCIe slot.

2. In the VMware ESXi web interface, expand the **Host** section on the left panel, and select the **Manage** menu item.

3. In the **Manage** page, click the **Hardware** tab.

4. Click the **PCI Devices** section.

5. From the list of PCI devices, select the corresponding checkbox to the SATA adapter/card you have installed to the ESXi server i.e. `JMicron Technology Corp. SATA controller`.

6. Click the **Toggle passthrough** button to update the device's **Passthrough** status from `Disabled` to `Enabled`.

7. Click the **Reboot host** button to reboot the ESXi server for the changes to apply.

8. After the ESXi server has rebooted, navigate back to the same **PCI Devices** section, and verify that the SATA adapter/card's **Passthrough** status is `Active`.

9. To use the SATA adapter/card on a VM, [create](#creating-a-virtual-machine-from-a-template) a new VM or [edit](#editing-a-virtual-machine) an existing VM, and configure the VM's **Virtual Hardware**:

    - In the **Virtual Hardware** tab, click the **Add other device** button.
    - Click the **PCI Device** option.
    - In the newly added **New PCI Device** section, expand the dropdown, and select the SATA adapter/card we had enabled for passthrough.

    > [!WARNING]  
    > You **MAY** also need to update the **SCSI Controller 0** selection to `LSI Logic SAS`. **ONLY** do this if it turns out that this is actually required.

    - Click the **Save** button.
