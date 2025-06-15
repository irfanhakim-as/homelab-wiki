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
  - [Adding Kernel Parameters](#adding-kernel-parameters)
    - [Description](#description-2)
    - [References](#references-2)
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
