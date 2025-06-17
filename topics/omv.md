# OpenMediaVault

## Description

OpenMediaVault is the next generation network attached storage (NAS) solution based on Debian Linux. It contains services like SSH, (S)FTP, SMB/CIFS, RSync and many more ready to use.

## Directory

- [OpenMediaVault](#openmediavault)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Install OMV](#install-omv)
    - [Install OMV-Extras](#install-omv-extras)
    - [Install FlashMemory Plugin](#install-flashmemory-plugin)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [References](#references-2)
    - [General Configuration](#general-configuration)
    - [Storage Configuration](#storage-configuration)
  - [Storage Setup](#storage-setup)
    - [Description](#description-3)
    - [References](#references-3)
    - [Set Up Storage Disks](#set-up-storage-disks)
    - [SnapRAID](#snapraid)
    - [MergerFS](#mergerfs)
    - [SMB Share](#smb-share)
  - [Docker](#docker)
    - [Description](#description-4)
    - [References](#references-4)
    - [Installing Docker](#installing-docker)
    - [Deploying Docker Container](#deploying-docker-container)
  - [User Management](#user-management)
    - [Description](#description-5)
    - [User Home Directory](#user-home-directory)
    - [Create User](#create-user)
  - [Troubleshooting](#troubleshooting)
    - [Description](#description-6)
    - [References](#references-5)
    - [Storage Disk Not Showing on OMV](#storage-disk-not-showing-on-omv)
    - [Unable to Get SMART Data from Storage Disk](#unable-to-get-smart-data-from-storage-disk)
  - [Usage](#usage)
    - [Description](#description-7)
    - [References](#references-6)
    - [Web Interface](#web-interface)
    - [Installing Plugin](#installing-plugin)
    - [Create or Edit Scheduled Task](#create-or-edit-scheduled-task)
    - [Create Shared Folder](#create-shared-folder)

## References

- [OpenMediaVault](https://www.openmediavault.org)
- [Wikipedia](https://en.wikipedia.org/wiki/OpenMediaVault)

---

## Setup

### Description

This details the installation steps for setting up an OpenMediaVault (OMV) server.

### References

- [Installation](https://docs.openmediavault.org/en/latest/installation)
- [Installing OMV7 on a Raspberry PI](https://wiki.omv-extras.org/doku.php?id=omv7:raspberry_pi_install)
- [Installing OMV7 on i386 32-bit PC's](https://wiki.omv-extras.org/doku.php?id=omv7:i386_32-bit_install)
- [Installing OMV-Extras](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#installing_omv-extras)
- [FlashMemory Plugin For OMV7](https://wiki.omv-extras.org/doku.php?id=omv7:omv7_plugins:flashmemory)

### Install OMV

> [!NOTE]  
> This part of the guide focuses on the installation of OMV on SBCs like the [Raspberry Pi](raspberry-pi.md).

1. Prepare a system (i.e. virtual machine or bare metal) with a basic setup of [Debian](debian.md) Linux (64-bit):

   - If deploying OMV on a Raspberry Pi, [flashing Raspberry Pi OS Lite (64-bit)](raspberry-pi.md#flash-os-image) is highly recommended.

   - Unlike our usual, pretty extensive [Linux configuration](linux.md#configuration) recommendations, a very minimal configuration should suffice in this case as anything more may be conflicting with OMV's own configuration.

   - Your system is required to be connected to the internet via a wired connection before proceeding with the installation.

2. Run OMV's pre-install script on the system to ensure it is ready for the OMV installation:

    ```sh
    wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/preinstall | sudo bash
    ```

3. Reboot the system to apply any changes made by the pre-install script:

    ```sh
    sudo reboot now
    ```

4. Run OMV's installation script on the system to install and set it up as an OMV server:

    ```sh
    wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/install | sudo bash
    ```

      Note that you may lose remote SSH connection to the OMV server at a certain point during the installation process. Should this happen, simply reconnect to the server's latest IP address via SSH or the web interface.

### Install OMV-Extras

> [!NOTE]  
> OMV [installations intended for SBCs](#install-omv) (i.e. Raspberry Pi) already include OMV-Extras by default.

1. To install and enable OMV-Extras, run the following installation script on the OMV server:

   ```sh
   wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | sudo bash
   ```

### Install FlashMemory Plugin

> [!IMPORTANT]  
> OMV servers that rely on flash media such as SD cards or USB drives require the FlashMemory plugin for longevity.

> [!NOTE]  
> OMV [installations intended for SBCs](#install-omv) (i.e. Raspberry Pi) already include the FlashMemory plugin by default.

1. From the web interface, [install the FlashMemory plugin](#installing-plugin), `openmediavault-flashmemory` graphically.

2. **Alternatively**, [install](package-manager.md#install-software) the `openmediavault-flashmemory` package using the package manager (i.e. `apt`).

---

## Configuration

### Description

This details several configuration options for an OpenMediaVault (OMV) NAS.

### References

- [Openmediavault 7 New User Guide](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide)
- [Initial Configuration](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#initial_configuration)
- [SBC users should consider leaving their wired network interface set to DHCP, until Docker is installed](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#network_interfaces_sbc_users)
- [Network Interfaces – i386/amd64 Users](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#network_interfaces_sbc_users)
- [Hard Drive Health and SMART](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#hard_drive_health_and_smart)
- [Drive Self-Tests](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#drive_self-tests)

### General Configuration

> [!IMPORTANT]  
> Most changes made to the OMV configuration through the web interface requires additional confirmation in order to apply submitted changes - make sure to wait for the prompt and apply them accordingly.

This details recommended configuration options for OMV in general:

1. Set a secure password for the OMV admin user:

   - From the web interface, click the **User Settings** button on the top right corner, and select the **Change Password** menu option.

   - In the **Change Password** form, configure the following:

     - New password: Enter a secure password for the OMV admin user
     - Confirm password: Confirm the password you have set for the OMV admin user

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

2. Configure automatic logout for the web interface:

   - From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **Workbench** menu option.

   - In the **Workbench** form, configure the following:

     - Automatic logout: Expand the dropdown and select a duration (i.e. `60 minutes`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

3. Update the time zone of the OMV server:

   - From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **Date & Time** menu option.

   - In the **Date & Time** form, configure the following:

     - Time zone: Expand the dropdown and select the intended server time zone (i.e. `Asia/Kuala_Lumpur`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

4. Enable email notifications for the OMV server:

   - From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **Notification** menu option.

   - In the **Notification** page, click the **Settings** menu option.

   - In the Notification **Settings** form, configure the following:

     - Enabled: Check the box to enable email notifications
     - SMTP server: Set the SMTP server address of your email account (i.e. `smtp.gmail.com`)
     - SMTP port: Set the SMTP mail server port (i.e. `465`)
     - Encryption mode: Expand the dropdown and select the intended encryption mode (i.e. `SSL/TLS`)
     - Sender email: Set the sender email address (i.e. `omv@example.com`)
     - Authentication required: Check the box to enable requiring authentication for the sender email account
     - User name: Set the username of the sender email account (i.e. `omv@example.com`)
     - Password: Set the (app) password of the sender email account
     - Recipient:

       - Primary email: Set the primary email address to receive email notifications (i.e. `omv@example.com`)
       - **(Optional)** Secondary email: Set the secondary email address on the receiving end for email notifications

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

   - Click the **Test** button to send a test email as per our configuration to verify that email notifications are working.

5. Configure email notification events after enabling email notifications:

   - From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **Notification** menu option.

   - In the **Notification** page, click the **Events** menu option.

   - In the Notification **Events** form, configure the following:

     - Click the **Select All** button twice to deselect all events as a start.
     - Check (at least) the following events' corresponding checkboxes to enable them:

       - File systems
       - S.M.A.R.T.

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

6. Update the hostname of the OMV server:

   - From the web interface dashboard, click the **Network** group on the left-hand side to expand it, and click the **General** menu option.

   - In the Network **General** form, configure the following:

     - Hostname: Set a unique, descriptive hostname for the OMV server (i.e. `omv-1`)
     - **(Optional)** Domain name: Set the domain name of the OMV server (i.e. `example.com`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

7. Set a static IP address for the OMV server:

   - **(Important)** If you are running the OMV server on an SBC (i.e. Raspberry Pi), either set a static IP address from your network router, or only proceed with the following steps **AFTER** you have [installed Docker](#installing-docker) on the server.

   - From the web interface dashboard, click the **Network** group on the left-hand side to expand it, and click the **Interfaces** menu option.

   - In the **Interfaces** page, select to highlight the network interface (i.e. `eth0`), and click the **Edit** button.

   - In the Network interface **Edit** form, configure the following:

     - IPV4:

       - Method: Expand the dropdown and select the intended method (i.e. `Static`)
       - Address: Set the intended static IP address of the OMV server (i.e. `192.168.0.106`)
       - Netmask: Set the appropriate netmask for your network (i.e. `255.255.255.0`)
       - Gateway: Set the local gateway of your network (i.e. `192.168.0.1`)

     - IPV6:

       - Method: Expand the dropdown and select the intended method (i.e. `DHCP`)

     - Advanced Settings:

       - DNS servers: Set the intended DNS server(s) for the OMV server (i.e. `1.1.1.1,8.8.8.8`)
       - Wake-on-LAN: Check the box to enable wake-on-LAN for the OMV server (this feature may need to be enabled in the BIOS of the OMV server)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

   - If the static IP address you have set is different from its current IP address, you will lose connection to the OMV server and will need to reconnect using the new IP address.

8. Customise the dashboard of the OMV server:

   - From the web interface, click the **User Settings** button on the top right corner, and select the **Dashboard** menu option.

   - In the Dashboard **Settings** form, check to enable the following widgets:

     - CPU Utilization
     - Disk Temperatures
     - File Systems (grid)
     - Load Average
     - Memory
     - Network Interfaces (grid)
     - S.M.A.R.T. Status
     - System Information
     - Uptime

      Submit the form by clicking the **Save** button.

9. **(Optional)** TODO: Set up Timeshift for OMV server snapshots

### Storage Configuration

> [!NOTE]  
> This part of the guide assumes that you have enabled email notifications on the OMV server for S.M.A.R.T. events.

> [!NOTE]  
> Ensure all storage disks have been attached to the OMV server before proceeding.

This details recommended configuration options for OMV in terms of storage:

1. Enable SMART monitoring on the server:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **S.M.A.R.T.** menu option.

   - In the Storage **S.M.A.R.T.** page, click the **Settings** menu option.

   - In the S.M.A.R.T. **Settings** form, configure the following:

     - Enabled: Check the box to enable S.M.A.R.T. monitoring on the OMV server

     - Temperature monitoring:

       - Difference: Expand the dropdown and select the intended temp difference to report (i.e. `5 °C`)
       - Maximum: Expand the dropdown and select the intended maximum temp to report (i.e. `65 °C`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

2. For each storage disk attached to the OMV server, enable SMART monitoring for the disk:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **S.M.A.R.T.** menu option.

   - In the Storage **S.M.A.R.T.** page, click the **Devices** menu option.

   - In the S.M.A.R.T. **Devices** page, select to highlight the storage disk, and click the **Edit** button.

   - In the Devices **Edit** form, configure the following:

     - Monitoring enabled: Check the box to enable S.M.A.R.T. monitoring for the storage disk

      Submit the form by clicking the **Save** button.

   - Once you have done the above for each storage disk(s), if prompted to confirm and apply configuration changes made, click the **Apply** button.

3. For each storage disk attached to the OMV server, set up periodic drive self-tests:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **S.M.A.R.T.** menu option.

   - In the Storage **S.M.A.R.T.** page, click the **Scheduled Tasks** menu option.

   - In the S.M.A.R.T. **Scheduled Tasks** page, click the **Create** button.

   - In the Scheduled Tasks **Create** form, configure the following:

     - Enabled: Check the box to enable the self-test scheduled task
     - Device: Expand the dropdown and select the storage disk (i.e. `SATA SSD [/dev/sda, X GiB]`)
     - Type: Expand the dropdown and select the intended self-test option (i.e. `Short self-test`)
     - Hour: Expand the dropdown and select the intended hour (i.e. `3`)
     - Day of month: Expand the dropdown and select the intended day of month (i.e. `*`)
     - Month: Expand the dropdown and select the intended month (i.e. `*`)
     - Day of week: Expand the dropdown and select the intended day of week (i.e. `Sunday`)

      This sets the short self-test on the drive to run every Sunday at 3:00 AM. Submit the form by clicking the **Save** button.

   - Once you have done the above for each storage disk(s), if prompted to confirm and apply configuration changes made, click the **Apply** button.

4. Check and verify SMART test result(s) for each storage disk:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **S.M.A.R.T.** menu option.

   - In the Storage **S.M.A.R.T.** page, click the **Devices** menu option.

   - From the list of S.M.A.R.T. enabled storage device(s), select to highlight the storage disk, and click the **Show details** button.

   - In the device **Details** page, click the **Extended Information** tab.

   - In the device **Extended Information** tab, inspect the SMART data of the storage device. If there is no SMART data being returned, check out this [troubleshooting](#unable-to-get-smart-data-from-storage-disk) section for a possible fix.

---

## Storage Setup

### Description

This details how to setup the storage solution on the OMV server.

### References

- [Data Drive Set Up](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#data_drive_set_up)
- [SnapRAID Initialization](https://wiki.omv-extras.org/doku.php?id=omv7:omv7_plugins:snapraid#snapraid_initialization)
- [Creating a SMB/CIF “Samba” Network Share](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#creating_a_smb_cif_samba_network_share)

### Set Up Storage Disks

> [!NOTE]  
> Ensure all storage disks have been attached to the OMV server before proceeding.

This details how to prepare and setup the storage disks on the OMV server:

1. Ensure all storage disks have been attached and show up on the OMV server:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **Disks** menu option.

   - In the Storage **Disks** page, check and ensure that all of your storage disks appear here accordingly.

   - If they are attached but fail to appear on OMV, check out this [troubleshooting](#storage-disk-not-showing-on-omv) section for a possible fix.

2. **(Recommended)** Wipe any existing data on the storage disks before proceeding:

   - In the same Storage **Disks** page, select to highlight each storage disk, and click the **Wipe** button.

   - When prompted with the **Wipe** confirmation dialog, select the **Confirm** checkbox, and click the **Yes** button.

   - When prompted with the **Wipe** dialog asking for the method to wipe the disk, select the **Quick** option.

   - Wait for the process to complete, and click the console dialog's corresponding **Close** button.

3. Set up the file system on the storage disks:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **File Systems** menu option.

   - In the Storage **File Systems** page, click the **Create and mount a file system** button, and select the intended file system (i.e. `EXT4`).

   - In the File Systems **Create** form, configure the following:

     - Device: Expand the dropdown and select the intended storage disk (i.e. `/dev/sda [X GiB]`)

      Submit the form by clicking the **Save** button.

   - Wait for the **Create file system** process to complete, and click the console dialog's corresponding **Close** button.

   - In the File Systems **Mount** form, configure the following:

     - File system: Expand the dropdown and select the file system you had just created (i.e. `/dev/sda1 [EXT4, X GiB]`)

      Submit the form by clicking the **Save** button.

   - Once you have done the above for each storage disk(s), if prompted to confirm and apply configuration changes made, click the **Apply** button.

### SnapRAID

> [!NOTE]  
> This part of the guide assumes that you have at least a single storage disk designated for data storage, and another designated for parity storage. The parity storage disk should be at least the same size as the largest data storage disk in the array.

This details how to install and set up SnapRAID to have parity for our OMV server storage setup:

1. From the web interface, [install the SnapRAID plugin](#installing-plugin), `openmediavault-snapraid`.

2. Create a SnapRAID array:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **SnapRAID** menu option.

   - In the **SnapRAID** page, click the **Arrays** menu option.

   - In the SnapRaid **Arrays** page, click the **Create** button to create a new array.

   - In the Arrays **Create** form, configure the following:

     - Name: Set a unique, descriptive name for the SnapRAID array (i.e. `SnapR-A1`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

3. Populate the SnapRAID array with storage disks:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **SnapRAID** menu option.

   - In the **SnapRAID** page, click the **Drives** menu option.

   - In the SnapRaid **Drives** page, click the **Create** button to assign a drive to a SnapRAID array.

   - In the Drives **Create** form, configure the following:

     - Array: Expand the dropdown and select the SnapRAID array (i.e. `SnapR-A1`)
     - Drive: Expand the dropdown and select the storage drive (i.e. `/dev/sda1 [EXT4, X GiB]`)
     - Name: Set a unique, descriptive name for the drive based on its role (i.e. `data1`)
     - Content: Check the box to enable it if the storage drive is designated for **Data**
     - Data: Check the box to enable it if the storage drive is designated for **Data**
     - Parity: Check the box to enable it if the storage drive is designated for **Parity**

      Submit the form by clicking the **Save** button.

   - Once you have done the above for each storage disk(s), if prompted to confirm and apply configuration changes made, click the **Apply** button.

4. Configure SnapRAID settings:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **SnapRAID** menu option.

   - In the **SnapRAID** page, click the **Settings** menu option.

   - In the SnapRaid **Settings** form, configure the following:

     - Scrub frequency: `1`
     - Scrub percentage: `5`

      These settings are set to complete a full scrub every 20 days. Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

5. Scheduled tasks on OMV aren't named, [edit the existing task](#create-or-edit-scheduled-task) that has `/usr/sbin/omv-snapraid-diff` in the command field to enable it and make it run daily at a specified time (i.e. 4.30 AM):

   - Enabled: Check the box to enable the scheduled task
   - Time of execution: Expand the dropdown and select the `Certain date` option
   - Minute: Expand the dropdown and select the checkbox corresponding to the intended minute (i.e. `30`)
   - Hour: Expand the dropdown and select the checkbox corresponding to the intended hour (i.e. `4`)
   - Day of month: Expand the dropdown and select the checkbox corresponding to the daily option (i.e. `*`)
   - Month: Expand the dropdown and select the checkbox corresponding to the every month option (i.e. `*`)
   - Day of week: Expand the dropdown and select the checkbox corresponding to the daily option (i.e. `*`)
   - Send command output via email: Check the box to send command output via email upon task completion

    If you use or have installed [Docker](#docker) on the OMV server, it's a good idea to stop the Docker service before running the SnapRAID task and restarting it afterwards. To do so, update the tasks's `Command` field to the following:

    ```sh
    systemctl stop docker && while systemctl is-active --quiet docker; do sleep 1; done && for conf in /etc/snapraid/omv-snapraid-*.conf; do /usr/sbin/omv-snapraid-diff "${conf}"; done && systemctl start docker
    ```

6. Initialise the SnapRAID array by running a Sync operation:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **SnapRAID** menu option.

   - In the **SnapRAID** page, click the **Arrays** menu option.

   - In the SnapRaid **Arrays** page, select to highlight the SnapRAID array (i.e. `SnapR-A1`).

   - Click the **Tools** button, and select the **Sync** option.

   - Wait for the process to complete, and click the console dialog's corresponding **Close** button.

### MergerFS

> [!NOTE]  
> This part of the guide assumes that you have at least two data storage disks that you intend to pool together.

This details how to install and set up MergerFS to have a shared storage pool for our OMV server storage disks:

1. From the web interface, [install the MergerFS plugin](#installing-plugin), `openmediavault-mergerfs`.

2. For each data storage disk you wish to pool, [create a shared folder](#create-shared-folder) for it:

   - Name: Set a unique, descriptive name for the shared folder (i.e. `mfshare1`)
   - File system: Expand the dropdown and select the file system of the data storage disk (i.e. `/dev/sda1 [EXT4, X GiB]`)
   - Relative path: Leave as default (i.e. `mfshare1/`)
   - Permissions: `Everyone: read/write`

3. Create a MergerFS storage pool:

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **mergerfs** menu option.

   - In the Storage **mergerfs** page, click the **Create** button to create a new MergeFS storage pool.

   - In the MergerFS **Create** form, configure the following:

     - Name: Set a unique, descriptive name for the MergerFS storage pool (i.e. `Merger1`)
     - Shared folders: Expand the dropdown and check the corresponding box for each shared folder you have created earlier (i.e. `mfshare1` and `mfshare2`)
     - Create policy: Expand the dropdown and select the `Most free space` option
     - Minimum free space: Set a minimum free space for each file system in the pool (i.e. `10` `Gigabytes`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

4. **(Optional)** Find the newly created MergerFS storage pool (file system):

   - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **File Systems** menu option.

   - In the Storage **File Systems** page, click the **Show/Hide columns** button and check the **Label** option.

   - From the list of file systems, locate the MergerFS storage pool you have created from its label (i.e. `Merger1`).

5. [Create a shared folder](#create-shared-folder) using the MergerFS storage pool:

   - Name: Set a unique, descriptive name for the shared folder (i.e. `merger1.share`)
   - File system: Expand the dropdown and select the MergerFS storage pool file system (i.e. `Merger1 [X GiB]`)
   - Relative path: Leave as default (i.e. `merger1.share/`)
   - Permissions: `Administrator: read/write, Users: read/write, Others: no access`

### SMB Share

> [!NOTE]  
> This part of the guide assumes that you have prepared a [shared folder](#create-shared-folder) to be used for the SMB share.

This details how to enable SMB (CIFS) and set up an SMB share on the OMV server:

1. Enable and configure the SMB service on the server:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **SMB/CIFS** menu option.

   - In the **SMB/CIFS** page, click the **Settings** menu option.

   - In the SMB/CIFS **Settings** form, configure the following:

     - Enabled: Check the box to enable the SMB service

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

2. Create an SMB share:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **SMB/CIFS** menu option.

   - In the **SMB/CIFS** page, click the **Shares** menu option.

   - In the SMB/CIFS **Shares** page, click the **Create** button to create a new SMB share.

   - In the Shares **Create** form, configure the following:

     - Enabled: Check the box to enable the SMB share
     - Shared folder: Expand the dropdown and select the shared folder you wish to share (i.e. `merger1.share`)
     - Inherit ACLs: Check the box to inherit default ACLs from parent directories
     - Inherit permissions: Check the box to override permissions governed by create mask and directory mask
     - Extended attributes: Check the box to allow clients to attempt to access extended attributes
     - Store DOS attributes: Check the box to enable DOS attributes on the share

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

3. **(Optional)** [Create a service user](#create-user) dedicated to the purpose of accessing the SMB share:

   - Name: Set a unique, descriptive name for the user (i.e. `smbuser`)
   - Groups: Expand the dropdown and select the `users` group in addition to any others you may wish to add (i.e. `_ssh`)

---

## Docker

### Description

This details how to install and use Docker on the OMV server.

### References

- [Docker in OMV 7](https://wiki.omv-extras.org/doku.php?id=omv7:docker_in_omv)
- [(Docker) Compose Plugin For OMV7](https://wiki.omv-extras.org/doku.php?id=omv7:omv7_plugins:docker_compose)
- [Install and configure Docker](https://wiki.omv-extras.org/doku.php?id=omv7:docker_in_omv#install_and_configure_docker)
- [Plugin Settings](https://wiki.omv-extras.org/doku.php?id=omv7:docker_in_omv#plugin_settings)
- [CONFIGURATION](https://wiki.omv-extras.org/doku.php?id=omv7:docker_in_omv#configuration)
- [OpenMediaVault 7: Step-by-Step Docker Deployment with Docker Compose Plugin - Episode 2](https://youtu.be/D_d0DNUQcwM)

### Installing Docker

> [!NOTE]  
> This part of the guide assumes that you have finished configuring your [storage setup](#storage-setup).

This details how to install and setup Docker using the Compose plugin on OMV:

1. Enable the Docker repository on OMV-Extras (which you should have already [installed](#install-omv-extras)):

   - From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **omv-extras** menu option.

   - In the System **omv-extras** page, check the box corresponding to `Docker repo` to enable it.

   - Click the **Save** button.

2. From the web interface, [install the Compose plugin](#installing-plugin), `openmediavault-compose`. Note that this will also install the `openmediavault-sharerootfs` plugin as a dependency.

3. At this point, we should determine the intended storage structure for Docker on the OMV server. This guide focuses on a _variation_ of OMV-Extras' documented `ADVANCED OMV NAS SYSTEM` storage setup:

   - On at least one of your data storage disks, [create a shared folder](#create-shared-folder) designated for hosted apps (including Docker):

     - Name: Set a unique, descriptive name for the shared folder (i.e. `omv.app1`)
     - File system: Expand the dropdown and select the file system of the data storage disk (i.e. `/dev/sda1 [EXT4, X GiB]`)
     - Relative path: Set an appropriate path relative to the root of the file system (i.e. `omv.app/`)
     - Permissions: `Administrator: read/write, Users: read/write, Others: no access`

   - On one data storage disk, [create a shared folder](#create-shared-folder) designated for Docker Compose files:

     - Name: Set a unique, descriptive name for the shared folder (i.e. `omv.appdata`)
     - File system: Expand the dropdown and select the file system of the data storage disk (i.e. `/dev/sda1 [EXT4, X GiB]`)
     - Relative path: Leave as default (i.e. `omv.appdata/`)
     - Permissions: `Administrator: read/write, Users: read/write, Others: no access`

   - On one data storage disk, [create a shared folder](#create-shared-folder) designated for Docker Compose backups:

     - Name: Set a unique, descriptive name for the shared folder (i.e. `omv.backup_compose`)
     - File system: Expand the dropdown and select the file system of the data storage disk (i.e. `/dev/sdb1 [EXT4, X GiB]`)
     - Relative path: Leave as default (i.e. `omv.backup_compose/`)
     - Permissions: `Administrator: read/write, Users: read/write, Others: no access`

   - [Create a shared folder](#create-shared-folder) on your main storage pool or disk, if you do not have one already:

     - Name: Set a unique, descriptive name for the shared folder (i.e. `merger1.share`)
     - File system: Expand the dropdown and select the file system of the data storage pool or disk (i.e. `Merger1 [X GiB]`)
     - Relative path: Leave as default (i.e. `merger1.share`)
     - Permissions: `Administrator: read/write, Users: read/write, Others: no access`

      If one already exists, use said shared folder instead for storing persistent data for your Docker containers.

4. Configure and install Docker on the OMV server:

   - From the web interface dashboard, click the **Services** group on the left-hand side to expand it, and click the **Compose** menu option.

   - In the **Compose** service page, click the **Settings** menu option.

   - In the Compose **Settings** form, configure the following:

     - Compose Files:
       - Shared folder: Expand the dropdown and select the shared folder designated for Docker Compose files (i.e. `omv.appdata`)
     - Data:
       - Shared folder: Expand the dropdown and select the shared folder designated for storing persistent data (i.e. `merger1.share`)
     - Backup:
       - Shared folder: Expand the dropdown and select the shared folder designated for Docker Compose backups (i.e. `omv.backup_compose`)
     - Docker:
       - Docker storage: Expand the dropdown and select the shared folder designated for hosted apps (i.e. `omv.app1`)

      Submit the form by clicking the **Save** button.

   - If prompted to confirm and apply configuration changes made, click the **Apply** button.

   - In the same Compose **Settings** form:

     - Click the **Reinstall Docker** button to install Docker on the OMV server.
     - When prompted with the confirmation dialog, click the **Yes** button to confirm.
     - Wait for the process to complete, and click the console dialog's corresponding **Close** button.

### Deploying Docker Container

TODO

---

## User Management

### Description

This details topics pertaining user management on an OpenMediaVault (OMV) server.

### User Home Directory

> [!NOTE]  
> This part of the guide assumes that you have finished configuring your [storage setup](#storage-setup).

This details how to enable users created and managed by OMV to have their own home directories:

1. [Create a shared folder](#create-shared-folder) for the user home directory:

   - Name: Set a unique, descriptive name for the shared folder (i.e. `home`)
   - File system: Expand the dropdown and select the intended file system (i.e. `Merger1 [X GiB]`)
   - Relative path: Leave as default (i.e. `home/`)
   - Permissions: `Administrator: read/write, Users: read/write, Others: no access`

2. From the web interface dashboard, click the **Users** group on the left-hand side to expand it, and click the **Settings** menu option.

3. In the User Management **Settings** form, configure the following:

   - Enabled: Check the box to enable user home directories on the OMV server
   - Location: Expand the dropdown and select the shared folder for the user home directory (i.e. `home [on Merger1, home/]`)

    Submit the form by clicking the **Save** button.

4. If prompted to confirm and apply configuration changes made, click the **Apply** button.

### Create User

This details how to create a user account on the OMV server:

1. **(Optional)** Ensure you have enabled and configured [user home directories](#user-home-directory) on the OMV server.

2. From the web interface dashboard, click the **Users** group on the left-hand side to expand it, and click the **Users** menu option.

3. In the User Management **Users** page, click the **Create | Import** button, and select the **Create** option.

4. In the Users **Create** form, configure the following:

   - Name: Set a unique, descriptive name for the user (i.e. `user1`)
   - Password: Set a distinct, secure password for the user
   - Confirm password: Repeat the user password for confirmation
   - Shell: Leave as default or expand the dropdown and select a different default shell (i.e. `/usr/bin/bash`)
   - Groups: Expand the dropdown and select the intended group(s) for the user (i.e. `users` and `_ssh`)
   - SSH public keys: For every SSH public key you wish to add, click the **Add** button, and add it to the form accordingly

    Submit the form by clicking the **Save** button.

5. If prompted to confirm and apply configuration changes made, click the **Apply** button.

---

## Troubleshooting

### Description

This details several known issues and workarounds you may encounter on an OpenMediaVault (OMV) NAS.

### References

- [STICKY: If you have a Raspberry Pi 4 and are getting bad speeds transferring data to/from USB3.0 SSDs, read this](https://forums.raspberrypi.com/viewtopic.php?t=245931)
- [SMART statuses are not being checked or detected by OMV anymore](https://forum.openmediavault.org/index.php?thread/50404-smart-statuses-are-not-being-checked-or-detected-by-omv-anymore/&postID=373629#post373629)
- [smartctl(8) - Linux man page](https://linux.die.net/man/8/smartctl)

### Storage Disk Not Showing on OMV

> [!NOTE]  
> This is likely only an issue with using storage devices using a USB-to-SATA interface.

> [!NOTE]  
> This part of the guide is tailored for the [Raspberry Pi](raspberry-pi.md), and may or may not apply to your specific setup.

This details a potential fix for (USB) storage devices not being able to load or appear on OMV, potentially crashing the entire USB stack on the server:

1. After ensuring that all of your required storage device(s) have been physically attached, reboot the server to a _clean slate_ before proceeding to the following steps.

2. First, to identify that this issue is even happening on your server:

   - Check that the server itself (i.e. not OMV) can _see_ all of your storage devices without issue:

     - Check that the storage devices appear using `lsblk`:

         ```sh
         sudo lsblk
         ```

         Sample output showing all three storage devices attached to the server:

         ```
           NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
           sda           8:16   0 931.5G  0 disk
           sdb           8:32   0 931.5G  0 disk
           sdc           8:48   0 931.5G  0 disk
         ```

     - Find the (USB) storage interface(s) (i.e. `jmicron` in this sample) used to connect the storage devices to the server:

         ```sh
         sudo lsusb | grep -i jmicron
         ```

         While making sure that the relevant storage interface(s) show up as expected, take note of their `<VID>:<PID>` value(s) as well:

         ```
           Bus 001 Device 007: ID 152d:0583 JMicron Technology Corp. / JMicron USA Technology Corp. JMS583Gen 2 to PCIe Gen3x2 Bridge
           Bus 001 Device 006: ID 152d:0583 JMicron Technology Corp. / JMicron USA Technology Corp. JMS583Gen 2 to PCIe Gen3x2 Bridge
           Bus 001 Device 005: ID 152d:0583 JMicron Technology Corp. / JMicron USA Technology Corp. JMS583Gen 2 to PCIe Gen3x2 Bridge
         ```

         From this sample output, the `<VID>:<PID>` value of the storage interface is `152d:0583`.

   - Check if the same storage devices are visible on the OMV server:

     - From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **Disks** menu option.

     - In the Storage **Disks** page, check and ensure that all of the same storage disks appear here accordingly.

     - If they do appear without issue, it's likely that this issue does not apply to your setup.

   - If the disks fail to show up on OMV, check again with `lsblk` and `lsusb` to see if they've _disappeared_:

     - Check if the storage devices are no longer attached using `lsblk`:

         ```sh
         sudo lsblk
         ```

     - Find the (USB) storage interface(s) (i.e. `jmicron` in this sample) used to connect the storage devices to the server:

         ```sh
         sudo lsusb | grep -i jmicron
         ```

      If the earlier found or attached storage disks and interfaces are no longer visible from the output, proceed to the following steps.

3. This issue, in cases of using storage devices using a USB-to-SATA interface, is likely caused by the use of the USB Attached SCSI (UAS) driver. Verify that this is the case:

   ```sh
   sudo dmesg | grep -i uas
   ```

   Sample output:

   ```
     [    0.899109] usbcore: registered new interface driver uas
     [    1.430272] scsi host0: uas
     [    2.488903] scsi host1: uas
     [    2.741227] scsi host2: uas
   ```

   While the UAS driver is often preferable and more performant, some storage setup may not be fully compatible or compliant with the UAS specification. In such cases, it is recommended to use the `usb-storage` driver instead.

4. We need to tell our server to use the `usb-storage` driver, specifically for the affected storage device(s). On the Raspberry Pi, this can be done by [adding the following kernel parameter](raspberry-pi.md#adding-kernel-parameters):

   ```
   usb-storage.quirks=<VID>:<PID>:u
   ```

   Replace `<VID>:<PID>` with the `<VID>:<PID>` value(s) of the storage interface(s) you took note of earlier. For example:

   ```
   usb-storage.quirks=152d:0583:u
   ```

   If there are multiple storage interfaces with different `<VID>:<PID>` values, add them to the same kernel parameter accordingly, separated by a comma:

   ```
   usb-storage.quirks=<VID1>:<PID1>:u,<VID2>:<PID2>:u
   ```

5. After a reboot, verify that your storage device(s) are now using the `usb-storage` driver:

   ```sh
   sudo dmesg | grep -i uas
   ```

   Sample output:

   ```
     [    0.918739] usbcore: registered new interface driver uas
     [    1.439226] usb 2-1: UAS is ignored for this device, using usb-storage instead
     [    1.439267] usb 2-1: UAS is ignored for this device, using usb-storage instead
     [    2.510028] usb 1-1.1.1: UAS is ignored for this device, using usb-storage instead
     [    2.510066] usb 1-1.1.1: UAS is ignored for this device, using usb-storage instead
     [    2.766095] usb 1-1.1.4: UAS is ignored for this device, using usb-storage instead
     [    2.766132] usb 1-1.1.4: UAS is ignored for this device, using usb-storage instead
   ```

6. Check again on the OMV server web interface if the storage disks are now showing as expected.

### Unable to Get SMART Data from Storage Disk

This details a potential fix for OMV's inability to get SMART data from your storage disk(s):

1. First of all, ensure that `smartctl` is indeed unable to retrieve SMART data from your storage device:

   ```sh
   sudo smartctl -a <storage-device>
   ```

   For example, if your storage device is `/dev/sda`, run the following command:

   ```sh
   sudo smartctl -a /dev/sda
   ```

   If there is indeed an issue, it should return a _non-result_ output.

2. In some cases, an additional option may be needed for `smartctl` to be able to know the type of storage device you have attached, and return the intended SMART results for it:

   - In this example setup, the `--device` option is required on top of the same command:

      ```sh
      sudo smartctl -a --device <device-type> <storage-device>
      ```

      Replace `<device-type>` with the type of storage device or interface you are using to attach it to the server (i.e. `ata`, `scsi`, `sat`).

   - In this example setup, `sat` is the right device type to successfully get SMART data from the storage device (i.e. `/dev/sda`):

      ```sh
      sudo smartctl -a --device sat /dev/sda
      ```

      If this solves the issue, it should return the expected SMART data for the storage device.

3. After determining the exact option(s) needed for `smartctl` to acquire the SMART data from the storage device, we will need a solution that _informs_ `smartctl` on how to acquire the SMART data from that **specific** storage device or interface going forward:

   - In order to narrow the scope of our custom solution to the specific storage device or interface, we will need to get its `<VID>:<PID>` value:

      In this example setup, the storage devices are attached to the server using a USB-to-SATA interface made by `JMicron`:

      ```sh
      sudo lsusb | grep -i jmicron
      ```

      From the following sample output, the `<VID>:<PID>` value of the affected storage device or interface is `152d:0583`:

      ```
        Bus 001 Device 007: ID 152d:0583 JMicron Technology Corp. / JMicron USA Technology Corp. JMS583Gen 2 to PCIe Gen3x2 Bridge
        Bus 001 Device 006: ID 152d:0583 JMicron Technology Corp. / JMicron USA Technology Corp. JMS583Gen 2 to PCIe Gen3x2 Bridge
        Bus 001 Device 005: ID 152d:0583 JMicron Technology Corp. / JMicron USA Technology Corp. JMS583Gen 2 to PCIe Gen3x2 Bridge
      ```

   - Create a custom `smartmontools` drive database file with configuration targeting each unique storage device or interface that requires the additional option(s):

      ```sh
      sudo nano /etc/smart_drivedb.h
      ```

      For each unique storage device or interface (i.e. different `<VID>:<PID>` value), add the following configuration to the file:

      ```
      {
         "<name>",
         "0x<VID>:0x<PID>",
         "",
         "",
         "<additional-options>"
      },
      ```

      In this example setup, the additional `smartctl` option(s) are only needed for the storage devices or interfaces with the same `<VID>:<PID>` value of `152d:0583`:

      ```
      {
         "USB: JMicron JMS583 with SAT",
         "0x152d:0x0583",
         "",
         "",
         "--device sat"
      },
      ```

      As per the example, replace `<name>` with a unique, descriptive name, and the `<VID>` (i.e. `152d`), `<PID>` (i.e. `0583`), as well as `<additional-options>` (i.e. `--device sat`) placeholders accordingly.

   - Verify that the custom configuration we had set up is working as intended by getting the SMART data from the affected storage device (i.e. `/dev/sda`) **without** specifying the additional option(s) manually:

      ```sh
      sudo smartctl -a /dev/sda
      ```

      If all works correctly, this should now return the expected SMART data output - including in the OMV interface.

---

## Usage

### Description

This details some common usage steps for an OpenMediaVault (OMV) server.

### References

- [Web console login](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#web_console_login)
- [Setting up a Shared Folder](https://wiki.omv-extras.org/doku.php?id=omv7:new_user_guide#setting_up_a_shared_folder)

### Web Interface

Most of the configuration steps laid out in following section(s) must be done through the OMV server web interface.

1. To get to the OMV web interface, navigate to the following URL on a web browser:

    ```
      http://<omv-server-ip>
    ```

    Replace `<omv-server-ip>` with the IP address of the OMV server (i.e. `192.168.0.106`).

2. Once taken to the OMV web interface, access with the default login credentials if it is your first time logging in:

   - User name: `admin`
   - Password: `openmediavault`

    It is highly recommended that you change the default password to a more secure one.

3. After a successful login, you should be taken to the default OMV **Dashboard** page.

### Installing Plugin

> [!NOTE]  
> This part of the guide assumes that you have [OMV-Extras installed](#install-omv-extras) on the server.

This details how to install additional plugins to extend OMV's functionality:

1. From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **Plugins** menu option.

2. In the **Plugins** list, browse through the available plugins or use the search bar to find a specific plugin.

3. Select to highlight the plugin you wish to install, and click the **Install** button.

4. When prompted with the confirmation dialog, select the **Confirm** checkbox, and click the **Yes** button.

5. Wait for the process to complete, and click the console dialog's corresponding **Close** button.

### Create or Edit Scheduled Task

This details how to create or edit an existing scheduled task on the OMV server:

1. From the web interface dashboard, click the **System** group on the left-hand side to expand it, and click the **Scheduled Tasks** menu option.

2. In the **Scheduled Tasks** page:

   - To create a new scheduled task: Click the **Create** button.
   - To update an existing scheduled task: Select to highlight the task you wish to edit, and click the **Edit** button.

3. In the Scheduled Tasks **Create** or **Edit** form, configure the following:

   - Enabled: Check the box to enable the scheduled task or otherwise disable it
   - User: Set the user account to run the scheduled task as (i.e. `root`)
   - Command: Set the command to run as per the task
   - Send command output via email: Check the box to send command output via email upon task completion

    Configure the schedule for running the task accordingly. Submit the form by clicking the **Save** button.

4. If prompted to confirm and apply configuration changes made, click the **Apply** button.

### Create Shared Folder

This details how to create a shared folder on the OMV server:

1. From the web interface dashboard, click the **Storage** group on the left-hand side to expand it, and click the **Shared Folders** menu option.

2. In the Storage **Shared Folders** page, click the **Create** button.

3. In the Shared Folders **Create** form, configure the following:

   - Name: Set a unique, descriptive name for the shared folder (i.e. `share1`)
   - File system: Expand the dropdown and select the file system to create the shared folder on (i.e. `/dev/sda1 [EXT4, X GiB]`)
   - Relative path: Leave as default or set an appropriate path relative to the root of the file system (i.e. `share1/`)
   - Permissions: Expand the dropdown and select the intended permissions (i.e. `Admin: r/w, Users: r/w, Others: no access`)

    Submit the form by clicking the **Save** button.

4. If prompted to confirm and apply configuration changes made, click the **Apply** button.
