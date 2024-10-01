# Apt

## Description

Advanced Package Tool (or APT), the main command-line package manager for Debian and its derivatives. It provides command-line tools for searching, managing and querying information about packages, as well as low-level access to all features provided by the libapt-pkg and libapt-inst libraries which higher-level package managers can depend upon.

## Directory

- [Apt](#apt)
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

- [Apt](https://wiki.debian.org/Apt)
- [APT (software)](https://en.wikipedia.org/wiki/APT_(software))

---

## Install Software

### Description

This details the process of installing software.

### References

- [Installing, removing and upgrading packages](https://wiki.debian.org/AptCLI#Installing.2C_removing_and_upgrading_packages)

### Steps

1. To install a single package:

    ```sh
    sudo apt install <package>
    ```

    As an example, to install the `vim` package:

    ```sh
    sudo apt install vim
    ```

2. **Optionally**, use the `-y` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo apt install -y <package>
    ```

3. To install multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo apt install <package1> <package2> <package3>
    ```

---

## Update Software

### Description

This details the process of updating a software or the system.

### References

- [Installing, removing and upgrading packages](https://wiki.debian.org/AptCLI#Installing.2C_removing_and_upgrading_packages)

### Steps

1. Update all software repositories installed on the system **before** performing any of the following operations:

    ```sh
    sudo apt update
    ```

2. To update a single package:

    ```sh
    sudo apt install --only-upgrade <package>
    ```

    As an example, to upgrade the `vim` package:

    ```sh
    sudo apt install --only-upgrade vim
    ```

3. To update all installed packages:

    ```sh
    sudo apt upgrade
    ```

4. In some cases however, a simple upgrade is not enough. To perform a full system upgrade, run the following command:

    ```sh
    sudo apt dist-upgrade
    ```

---

## Remove Software

### Description

This details the process of uninstalling software.

### References

- [Installing, removing and upgrading packages](https://wiki.debian.org/AptCLI#Installing.2C_removing_and_upgrading_packages)

### Steps

1. To uninstall a single package:

    ```sh
    sudo apt remove <package>
    ```

    As an example, to uninstall the `vim` package:

    ```sh
    sudo apt remove vim
    ```

2. **Optionally**, use the `-y` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo apt remove -y <package>
    ```

3. To uninstall multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo apt remove <package1> <package2> <package3>
    ```

---

## Search Software

### Description

This details the process of searching software.

### References

- [Search for packages](https://wiki.debian.org/AptCLI#Search_for_packages)

### Steps

1. To search for a package:

    ```sh
    apt search <package>
    ```

    As an example, to search for the `vim` package:

    ```sh
    apt search vim
    ```

---

## Clean Up

### Description

This details the process of cleaning up the system.

### References

- [Delete cached package files](https://wiki.debian.org/AptCLI#Delete_cached_package_files)
- [Sudo apt autoremove](https://discourse.lubuntu.me/t/sudo-apt-autoremove/3699)

### Steps

1. To remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed:

    ```sh
    sudo apt autoremove
    ```

2. To delete cached package files that have been installed:

    ```sh
    sudo apt clean
    ```
