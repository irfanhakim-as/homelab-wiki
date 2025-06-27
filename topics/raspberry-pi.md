# Raspberry Pi

## Description

Raspberry Pi is a series of small single-board computers (SBCs) developed in the United Kingdom by the Raspberry Pi Foundation in collaboration with Broadcom.

## Directory

- [Raspberry Pi](#raspberry-pi)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Raspberry Pi Imager](#raspberry-pi-imager)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Flash OS Image](#flash-os-image)
  - [RaspiBackup](#raspibackup)
    - [Description](#description-2)
    - [References](#references-2)
    - [Installing RaspiBackup](#installing-raspibackup)
    - [Configuring RaspiBackup](#configuring-raspibackup)
  - [Log2Ram](#log2ram)
    - [Description](#description-3)
    - [References](#references-3)
    - [Installing Log2Ram](#installing-log2ram)
    - [Configuring Log2Ram](#configuring-log2ram)
  - [Adding Kernel Parameters](#adding-kernel-parameters)
    - [Description](#description-4)
    - [References](#references-4)
    - [Steps](#steps)

## References

- [Raspberry Pi](https://www.raspberrypi.com)
- [Wikipedia](https://en.wikipedia.org/wiki/Raspberry_Pi)

---

## Raspberry Pi Imager

### Description

This details how to install and use the Raspberry Pi Imager application to flash an OS image to a storage medium for the Raspberry Pi.

### References

- [Install using Imager](https://www.raspberrypi.com/documentation/computers/getting-started.html#raspberry-pi-imager)

### Installation

This details how to install the Raspberry Pi Imager application:

1. On Linux, the Raspberry Pi Imager can be easily installed as a [Flatpak](https://flathub.org/apps/org.raspberrypi.rpi-imager).

2. **Alternatively**, download and install Raspberry Pi Imager from the [Raspberry Pi website](https://www.raspberrypi.com/software).

### Flash OS Image

This details how to flash an OS image to a storage medium for Raspberry Pi:

1. Launch the **Raspberry Pi Imager** desktop application.

2. In the landing page of the app, click the **CHOOSE DEVICE** button.

3. In the **Raspberry Pi Device** window, locate and select the device you wish to flash the OS image to (i.e. `Raspberry Pi 4`).

4. In the landing page of the app, click the **CHOOSE OS** button.

5. In the **Operating System** window, scroll through the list of options and select your pick accordingly:

   - In the main menu, there may be a small selection of **Raspberry Pi OS** options you could choose from such as: Raspberry Pi OS 64-bit, 32-bit, and Legacy (32-bit).
   - Options under the **Raspberry Pi OS (other)** group: Raspberry Pi OS Lite (64-bit and 32-bit), Raspberry Pi OS Full (64-bit and 32-bit), and others.
   - Options under the **Other general-purpose OS** group: Ubuntu, Alpine Linux, Armbian, and others.
   - Options under the **Media player OS** group: LibreELEC, OSMC, Volumio, and others.
   - Options under the **Emulation and game OS** group: Recalbox, RetroPie, and others.
   - Options under the **Other specific-purpose OS** group: Kali Linux, Home Assistant OS, OctoPi, and others.
   - Options under the **Freemium and paid-for OS** group: 3DPrinterOS, Yodeck Digital Signage, TLXOS, and others.
   - Options under the **Misc utility images** group: Bootloader, PINN, and others.
   - If no options here suit your needs, or if you would like to add your own image you have created or downloaded: Select the **Use custom** option, and supply your own image file accordingly.

6. In the landing page of the app, click the **CHOOSE STORAGE** button.

7. In the **Storage** window, locate and select the storage device (i.e. SD card or USB drive) you wish to flash the OS image to (i.e. `Generic STORAGE DEVICE Media - X GiB`).

8. Click the **NEXT** button to proceed.

9. **(Recommended)** When prompted with the **Use OS customisation** window asking if you would like to apply OS customisation settings, click the **EDIT SETTINGS** button, and configure only the options that you require:

   - In the **GENERAL** tab:

     - Hostname: Set a unique, descriptive hostname for the Raspberry Pi (i.e. `pi-1`)
     - Username and password: Set a secure pair of Username (i.e. `user`) and Password for the Raspberry Pi user
     - Wireless LAN: Set the SSID (i.e. `MY-WIFI-5G`), Password, and Country (i.e. `MY`) of the WiFi the Raspberry Pi should use
     - Locale settings: Set the locale Time zone (i.e. `Asia/Kuala_Lumpur`) and Keyboard layout (i.e. `us`)

   - In the **SERVICES** tab:

     - **(Recommended)** Enable SSH: Check the box to enable remote SSH connection to the Raspberry Pi
     - Use password authentication: Select this option if you would like to use password authentication for SSH
     - **(Recommended)** Allow public-key authentication only: Select this option if you would like to only allow public-key authentication for SSH - paste your public SSH key(s) to the provided field accordingly (separated by newline if you have multiple)

   - In the **OPTIONS** tab:

     - Play sound when finished: Check the box if you wish to be alerted when the app has finished flashing your storage device
     - Eject media when finished: Check the box so the storage device is automatically ejected when the flashing process has completed
     - Enable telemetry: Check the box if you would like to enable telemetry to help improve the app

    Click the **SAVE** button once you are done configuring. **Otherwise**, simply click the **NO** button to skip this step if you do not require these customisations.

10. When prompted with the **Warning** window warning you that the chosen storage device will be erased, click the **YES** button to proceed with the flashing process.

11. Wait for the flashing process to complete, and click the **CONTINUE** button in the **Write Successful** window once it's done.

12. Remove the flashed storage device from your computer and attach it to your Raspberry Pi device accordingly.

---

## RaspiBackup

### Description

This details how to set up a backup solution using RaspiBackup for the Raspberry Pi.

### References

- [raspiBackup](https://github.com/framps/raspiBackup)
- [raspiBackup - Installation and configuration in 5 minutes](https://www.linux-tips-and-tricks.de/en/installation)
- [Backup types](https://www.linux-tips-and-tricks.de/en/backup#butypes)
- [Restorescenario for Linuxusers](https://www.linux-tips-and-tricks.de/en/restore#linxrestore)

### Installing RaspiBackup

> [!NOTE]  
> A separate storage device, local or remote, is required to store the backup data.

1. Ensure that you have [prepared a secondary storage](linux.md#partition-storage) on the Raspberry Pi device to be used as the backup storage.

2. Create and designate a directory (i.e. `/mnt/data/raspi-backup`) on the intended backup storage device to store the RaspiBackup backup data:

    ```sh
    mkdir -p <backup-directory>
    ```

    For example:

    ```sh
    mkdir -p /mnt/data/raspi-backup
    ```

3. [Set up an email sender](email.md#email-sender-setup) system (i.e. sSMTP) if there is not an existing one on the Raspberry Pi device.

4. Download and start the RaspiBackup installation script using the following command:

    ```sh
    curl -o install -L https://raw.githubusercontent.com/framps/raspiBackup/master/installation/install.sh; sudo bash ./install
    ```

5. Go through the installation procedure:

   - In the **Navigation** section, select the **Ok** option to get to the _landing page_.
   - From the _landing page_ of the **raspiBackup Installation and Configuration Tool**, select the **Install components** menu option.
   - In the **Install components** section:
     - Select the **Install raspiBackup using a default configuration** option.
     - Select the **Back** option to return to the _landing page_ once the process has completed.

6. Go through the [configuration process](#configuring-raspibackup) as part of the installation.

7. If presented with the **First steps** section informing you that RaspiBackup has been installed successfully, select the **Ok** option.

8. If presented with the **Help** section offering some helpful links, select the **Ok** option to exit the script.

### Configuring RaspiBackup

1. If you are configuring RaspiBackup for the first time as part of the installation process, proceed to the next step accordingly. If you are configuring RaspiBackup after the fact, run the following command first to start the configuration process:

    ```sh
    sudo raspiBackupInstallUI
    ```

2. From the _landing page_ of the **raspiBackup Installation and Configuration Tool**, select the **Configure major options** menu option to start the configuration process.

3. In the **Configure major options** section, select and configure the following options:

   - Backup path:
     - Backup path: Set the path to the backup data directory you have created (i.e. `/mnt/data/raspi-backup`)
   - Backup versions:
     - Select the `Keep a maximum number of backups` option
     - Number of backups to keep: Set the intended number of backups to keep (i.e. `3`)
   - Backup type:
     - Select the intended backup method (i.e. `Backup with tar`)
   - Backup mode:
     - Select the intended backup mode (i.e. `Backup the two standard partitions`)
   - **(Optional)** Services to stop and start:
     - If there are any services you need or wish to stop during the backup process and restart once it's finished (i.e. `docker`), select them accordingly
   - Message verbosity:
     - Select the intended verbosity of log messages (i.e. `Display all messages`)
   - eMail notification:
     - eMail address to send notifications to: Set the intended receiver email address (i.e. `admin@example.com`)
     - Mail program: Select the configured email sender on the system (i.e. `ssmtp`) or leave as default (`mail`)
   - Regular backup:
     - Select the `Enable regular backup` option to perform periodical backups
     - Weekday of regular backup: Select the intended day of the week to perform backups (i.e. `Daily`)
     - Time of regular backup: Select the intended time of day to perform backups (i.e. `01:00`)
   - Compression:
     - Select the `Compress tar backup` option to reduce the size of `tar` backups

4. Once you have finished configuring all of the intended settings, go **Back** to exit the **Configure major options** section and submit the configuration options.

5. When prompted to save the updated configuration, select the **Yes** option.

6. If prompted to save the updated `systemd` settings, select the **Yes** option.

7. Once you have been redirected to the **raspiBackup Installation and Configuration Tool** _landing page_, select the **Finish** option.

8. You may be presented with a warning of _inconsistent backups_ if you have opted not to stop any services before initiating backups. If so, select the **Yes** option to ignore the warning.

9. If presented with the **Help** section offering some helpful links, select the **Ok** option to exit the script.

---

## Log2Ram

### Description

This details the setup process of Log2Ram to delegate some writing operations to RAM instead of the storage medium (i.e. microSD card).

### References

- [Log2Ram](https://github.com/azlux/log2ram)
- [Using Log2RAM on the Raspberry Pi](https://pimylifeup.com/raspberry-pi-log2ram)
- [Track down additional folders that are seeing many writes](https://forum.openmediavault.org/index.php?thread/6438-tutorial-experimental-third-party-plugin-available-reducing-omv-s-disk-writes-al)

### Installing Log2Ram

This details how to install and set up Log2Ram on the system:

1. Add the appropriate software repository to the system:

    ```sh
    echo "deb [signed-by=/usr/share/keyrings/azlux-archive-keyring.gpg] http://packages.azlux.fr/debian/ bookworm main" | sudo tee /etc/apt/sources.list.d/azlux.list
    sudo wget -O /usr/share/keyrings/azlux-archive-keyring.gpg  https://azlux.fr/repo.gpg
    ```

2. [Update all software repositories](package-manager.md#update-software) installed on the system using `apt`.

3. [Install](package-manager.md#install-software) the `log2ram` package using `apt`.

4. [Configure Log2Ram](#configuring-log2ram) to finish the setup.

5. After a reboot, verify that Log2Ram has been set up correctly:

    ```sh
    df -hT | grep log2ram | awk '{print " Name: " $1 "\nMount: " $7 "\n Type: " $2 "\nUsage: " $6 "\n Size: " $3 "\n Used: " $4 "\n Free: " $5}'
    ```

    Sample output:

    ```
       Name: log2ram
      Mount: /var/log
       Type: tmpfs
      Usage: 1%
       Size: 256M
       Used: 52K
       Free: 256M
       Name: log2ram
      Mount: /var/tmp
       Type: tmpfs
      Usage: 0%
       Size: 256M
       Used: 0
       Free: 256M
       Name: log2ram
      Mount: /var/spool
       Type: tmpfs
      Usage: 0%
       Size: 256M
       Used: 0
       Free: 256M
    ```

### Configuring Log2Ram

This details the recommended configuration options for Log2Ram:

1. Create a backup of the default Log2Ram configuration file:

    ```sh
    sudo cp /etc/log2ram.conf /etc/log2ram.conf.bak
    ```

2. Update the Log2Ram configuration:

   - Update the configuration file:

      ```sh
      sudo nano /etc/log2ram.conf
      ```

   - Increase the amount of RAM reserved for Log2Ram (i.e. `256M`):

      ```diff
      - SIZE=128M
      + #SIZE=128M
      + SIZE=256M
      ```

      You may check the amount of (RAM) storage needed for all the directories you wish to store in RAM, separated by a single whitespace (i.e. `/var/log`, `/var/tmp`, and `/var/spool`):

      ```sh
      sudo du -sh /var/log /var/tmp /var/spool
      ```

      Sample output:

      ```
        72K     /var/log
        36K     /var/tmp
        12K     /var/spool
      ```

   - Append to the list of directories to store in RAM, separated by a semicolon (i.e. `/var/log`, `/var/tmp`, and `/var/spool`):

      ```diff
      - PATH_DISK="/var/log"
      + #PATH_DISK="/var/log"
      + PATH_DISK="/var/log;/var/tmp;/var/spool"
      ```

   - Save changes made to the configuration file.

3. You may need to reboot the system for the changes to take effect.

---

## Adding Kernel Parameters

### Description

This details how to add command line parameters to the Linux kernel on a Raspberry Pi.

### References

- [Kernel command line (cmdline.txt)](https://www.raspberrypi.com/documentation/computers/configuration.html#kernel-command-line-cmdline-txt)

### Steps

1. Create a backup of the default `cmdline.txt` file:

    ```sh
    sudo cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.bak
    ```

2. Add the desired kernel parameters to the `cmdline.txt` file:

   - Update the `cmdline.txt` file:

      ```sh
      sudo nano /boot/firmware/cmdline.txt
      ```

      Sample default content of the file:

      ```
        console=serial0,215110 console=tty1 root=PARTUUID=00dde0b8-05 rootfstype=ext4 fsck.repair=yes rootwait
      ```

   - On the same line in that file, add the desired kernel parameters to the start of the line:

      ```
        <new-kernel-parameters> console=serial0,115200 console=tty1 root=PARTUUID=00dde0b8-05 rootfstype=ext4 fsck.repair=yes rootwait
      ```

      or to the end of the line:

      ```
        console=serial0,115200 console=tty1 root=PARTUUID=00dde0b8-05 rootfstype=ext4 fsck.repair=yes rootwait <new-kernel-parameters>
      ```

      Each parameter should be separated from each other by a single space.

   - Save the changes made to the `cmdline.txt` file.

3. Reboot the system to apply all of the kernel parameters on the next boot:

    ```sh
    sudo reboot now
    ```

4. Verify that the new kernel parameters have been applied:

    ```sh
    sudo cat /proc/cmdline
    ```

    Sample output:

    ```
      console=serial0,115200 console=tty1 root=PARTUUID=00dde0b8-05 rootfstype=ext4 fsck.repair=yes rootwait <new-kernel-parameters>
    ```
