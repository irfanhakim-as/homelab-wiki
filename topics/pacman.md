# Pacman

## Description

The pacman package manager is one of the major distinguishing features of Arch Linux. It combines a simple binary package format with an easy-to-use Arch build system. The goal of pacman is to make it possible to easily manage packages, whether they are from the official repositories or the user's own builds.

## Directory

- [Pacman](#pacman)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Install Software](#install-software)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Update Software](#update-software)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)
  - [Remove Software](#remove-software)
    - [Description](#description-3)
    - [References](#references-3)
    - [Steps](#steps-2)
  - [Search Software](#search-software)
    - [Description](#description-4)
    - [References](#references-4)
    - [Steps](#steps-3)
  - [Clean Up](#clean-up)
    - [Description](#description-5)
    - [References](#references-5)
    - [Steps](#steps-4)

## References

- [Pacman](https://wiki.archlinux.org/title/Pacman)

---

## Install Software

### Description

This details the process of installing software.

### References

- [Installing packages](https://wiki.archlinux.org/title/Pacman#Installing_packages)

### Steps

1. To install a single package:

    ```sh
    sudo pacman -S <package>
    ```

    As an example, to install the `vim` package:

    ```sh
    sudo pacman -S vim
    ```

2. **Optionally**, use the `--noconfirm` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo pacman -S --noconfirm <package>
    ```

3. To install multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo pacman -S <package1> <package2> <package3>
    ```

---

## Update Software

### Description

This details the process of updating a software or the system.

### References

- [Upgrading packages](https://wiki.archlinux.org/title/Pacman#Upgrading_packages)

### Steps

1. To update a particular package or multiple of them, simply [reinstall](#install-software) the package(s).

2. To update the entire system:

    ```sh
    sudo pacman -Syu
    ```

---

## Remove Software

### Description

This details the process of uninstalling software.

### References

- [Removing packages](https://wiki.archlinux.org/title/Pacman#Removing_packages)

### Steps

1. To uninstall a single package along with its dependencies:

    ```sh
    sudo pacman -Rns <package>
    ```

    As an example, to uninstall the `vim` package:

    ```sh
    sudo pacman -Rns vim
    ```

2. **Optionally**, use the `--noconfirm` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo pacman -Rns --noconfirm <package>
    ```

3. To uninstall multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo pacman -Rns <package1> <package2> <package3>
    ```

---

## Search Software

### Description

This details the process of searching software.

### References

- [Querying package databases](https://wiki.archlinux.org/title/Pacman#Querying_package_databases)

### Steps

1. To search for a package:

    ```sh
    pacman -Ss <package>
    ```

    As an example, to search for the `vim` package:

    ```sh
    pacman -Ss vim
    ```

---

## Clean Up

### Description

This details the process of cleaning up the system.

### References

- [Removing unused packages (orphans)](https://wiki.archlinux.org/title/Pacman/Tips_and_tricks#Removing_unused_packages_(orphans))
- [Cleaning the package cache](https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache)

### Steps

1. To remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed:

    ```sh
    sudo pacman -Rns $(pacman -Qdtq)
    ```

2. To delete cached package files of packages that have been installed or removed:

    ```sh
    sudo pacman -Scc
    ```
