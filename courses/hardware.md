# Hardware

## Description

This details the hardware used in my homelab and any related configuration steps in setting them up.

## Directory

- [Hardware](#hardware)
  - [Description](#description)
  - [Directory](#directory)
  - [Hardware Specification](#hardware-specification)
    - [Description](#description-1)
    - [Main Server](#main-server)
    - [Networking](#networking)
    - [Storage](#storage)
  - [Hardware Configuration](#hardware-configuration)
    - [Description](#description-2)
    - [Motherboard BIOS](#motherboard-bios)
  - [Secondary NAS](#secondary-nas)
    - [Description](#description-3)
    - [Main Server](#main-server-1)
    - [Storage](#storage-1)

---

## Hardware Specification

### Description

This details the hardware used in my homelab.

### Main Server

- Motherboard: [ASRock B450M Pro4](https://www.asrock.com/mb/AMD/B450M%20Pro4/index.asp)

- CPU: [AMD Ryzen 7 1700](https://www.amd.com/en/support/cpu/amd-ryzen-processors/amd-ryzen-7-desktop-processors/amd-ryzen-7-1700)

- GPU: [NVIDIA GeForce GT 710 1GB](https://www.techpowerup.com/gpu-specs/geforce-gt-710.c1990)

  > [!NOTE]  
  > The GPU is only used for video output due to lack of integrated graphics on the CPU.

- RAM: 4x [ADATA Premier DDR4 2666](https://www.adata.com/en/specification/483) 16GB (64GB)

- PSU: [Cooler Master MWE Gold 750 Full Modular](https://www.coolermaster.com/catalog/power-supplies/mwe-series/mwe-gold-750-full-modular)

  > [!NOTE]  
  > A trusted 500W Gold-rated (or better) PSU should be more than sufficient for this build.

- Storage:

  - OS: [Silicon Power PCIe Gen 3x4 P34A80](https://www.silicon-power.com/web/us/product-P34A80) 512GB
  - Data (VM): [Crucial MX500 SATA SSD](https://www.crucial.com/products/ssd/crucial-mx500-ssd) 1TB

- Chassis: [Tecware Quad Cube](https://www.tecware.co/quad)

  > [!WARNING]  
  > The tempered glass side panels on the Tecware Quad Cube falls off from the chassis easily over time.

### Networking

- **(Optional)** Network Card: [WINYAO E575T2 (Intel 82575EB)](https://www.winyao.com/a/product_center/network_card__wangka/RJ45/2020/0420/114.html)

- Router: [D-Link DIR-X3060Z AX3000](https://www.dlink.com.my/product/ax3000-mesh-gigabit-wireless-router)

- Network Switch: [TL-SG105E](https://www.tp-link.com/my/business-networking/easy-smart-switch/tl-sg105e)

### Storage

- Storage Card: [ACASIS PCIe SATA Card AC-PE001 (JMB585)](https://shopee.com.my/acasisofficialshop.os/10403688152)

  > [!WARNING]  
  > This SATA card is reportedly not compatible with X470 and X570 motherboards from MSI.

- Storage Disk: 4x [Seagate BarraCuda 4TB ST4000DM004](https://www.seagate.com/as/en/products/hard-drives/barracuda-hard-drive) (16TB)

  > [!NOTE]  
  > These disks are attached to the storage card to be passed through to the NAS VM.

- SATA Cable: [SATA III 6Gbps SAS Cable](https://shopee.com.my/countless.my/22518192840) (6 SATA to 6 SATA, 1m)

---

## Hardware Configuration

> [!NOTE]  
> Parts of this section may be specific to the hardware specified in the [Hardware Specification](#hardware-specification) section.

### Description

This details the configuration steps for the hardware used in a homelab.

### Motherboard BIOS

> [!NOTE]  
> These steps are tailored to the ASRock B450M Pro4 motherboard, but should be applicable to most motherboards.

This details some recommended configuration options for the motherboard BIOS:

1. Turn on the computer and immediately spam the appropriate key to enter the BIOS (i.e. <kbd>Delete</kbd>).

2. Once in the BIOS, locate and configure the following settings accordingly:

   - Secure Boot: `Disabled`
   - Fast Boot: `Disabled`
   - IOMMU: `Enabled`
   - SVM: `Enabled`
   - SR-IOV: `Enabled`
   - Cool 'n' Quiet: `Disabled`

3. **(Recommended)** Save the BIOS settings to a profile on the motherboard or a USB drive.

4. Save changes made and exit the BIOS.

---

## Secondary NAS

### Description

This details the hardware used in my secondary NAS.

### Main Server

- Server: [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b) 4GB RAM

- PSU: [Official Power Adapter for Raspberry Pi 4 15W Black (DC 5V, 3A) UK Plug](https://shopee.com.my/autobotic/8383451404)

- Storage:

  - OS: [SanDisk High Endurance V30 U3 Class 10 microSD](https://shopee.com.my/all_it.os/2495770926) 64GB
  - Data (Backup): [Kingston Pendrive DataTraveler Exodia Onyx DTXON USB 3.2 Gen1 Flash Drive](https://shopee.com.my/all_it.os/7948053622) 64GB

- Chassis: [Black ABS Casing with Fan for Raspberry Pi 4 with GPIO Slot](https://shopee.com.my/autobotic/5036358937)

- Cooler: [Raspberry Pi 4 Heatsink](https://shopee.com.my/autobotic/2734588353)

### Storage

- USB 3.0 Powered Hub:

  - Hub: [USB HUB Super High Speed Multi usb Hub](https://shopee.com.my/widetalent/7939944334) (USB 3.0, 4 Ports)
  - PSU: [AC 100V-240V adapter DC 5V 4A 15V 2A 24V 1A Switching power supply DC plug 3.5mm x 1.35mm UK](https://shopee.com.my/wuzhihui2019.my/13537804309) (5V 4A)

- Storage Disk: 3x [PNY CS900 2.5" SATA SSD](https://shopee.com.my/brightstarcomputer/20244351359) 1TB

- SATA Cable: 3x [SATA TO USB Cable for 2.5 Inch Adapter](https://shopee.com.my/lumpsumstore/5767654064) (USB 3.0)
