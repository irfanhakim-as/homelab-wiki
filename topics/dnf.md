# Dandified YUM (DNF)

## Description

Dandified YUM (DNF) is the next upcoming major version of YUM. It does package management using RPM, libsolv and hawkey libraries. For metadata handling and package downloads it utilizes librepo. To process and effectively handle the comps data it uses libcomps.

## Directory

- [Dandified YUM (DNF)](#dandified-yum-dnf)
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

- [Dandified YUM](https://github.com/rpm-software-management/dnf)
- [DNF (software)](https://en.wikipedia.org/wiki/DNF_(software))

---

## Install Software

### Description

This details the process of installing software.

### References

- [dnf command](https://docs.rockylinux.org/books/admin_guide/13-softwares/#dnf-command)
- [Other useful dnf options](https://docs.rockylinux.org/books/admin_guide/13-softwares/#other-useful-dnf-options)
- [Group Command](https://dnf.readthedocs.io/en/latest/command_ref.html#group-command-label)

### Steps

1. To install a single package:

    ```sh
    sudo dnf install <package>
    ```

    As an example, to install the `vim` package:

    ```sh
    sudo dnf install vim
    ```

2. **Optionally**, use the `-y` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo dnf install -y <package>
    ```

3. To install multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo dnf install <package1> <package2> <package3>
    ```

4. To install any of the predefined groups of packages, use the `group install` command:

    ```sh
    sudo dnf group install '<group>'
    ```

    As an example, to install the `Development Tools` group:

    ```sh
    sudo dnf group install 'Development Tools'
    ```

---

## Update Software

### Description

This details the process of updating a software or the system.

### References

- [Upgrade Command](https://dnf.readthedocs.io/en/latest/command_ref.html#upgrade-command-label)

### Steps

1. To update a single package:

    ```sh
    sudo dnf upgrade <package>
    ```

    As an example, to upgrade the `vim` package:

    ```sh
    sudo dnf upgrade vim
    ```

2. To update the entire system:

    ```sh
    sudo dnf upgrade
    ```

---

## Remove Software

### Description

This details the process of uninstalling software.

### References

- [dnf command](https://docs.rockylinux.org/books/admin_guide/13-softwares/#dnf-command)

### Steps

1. To uninstall a single package:

    ```sh
    sudo dnf remove <package>
    ```

    As an example, to uninstall the `vim` package:

    ```sh
    sudo dnf remove vim
    ```

2. **Optionally**, use the `-y` flag to automatically answer "yes" to all prompts:

    ```sh
    sudo dnf remove -y <package>
    ```

3. To uninstall multiple packages in a single command, simply separate each package with a space:

    ```sh
    sudo dnf remove <package1> <package2> <package3>
    ```

---

## Search Software

### Description

This details the process of searching software.

### References

- [dnf command](https://docs.rockylinux.org/books/admin_guide/13-softwares/#dnf-command)
- [Group Command](https://dnf.readthedocs.io/en/latest/command_ref.html#group-command-label)

### Steps

1. To search for a package:

    ```sh
    dnf search <package>
    ```

    As an example, to search for the `vim` package:

    ```sh
    dnf search vim
    ```

2. To list down all available predefined groups of packages:

    ```sh
    dnf group list
    ```

---

## Clean Up

### Description

This details the process of cleaning up the system.

### References

- [dnf command](https://docs.rockylinux.org/books/admin_guide/13-softwares/#dnf-command)
- [Other useful dnf options](https://docs.rockylinux.org/books/admin_guide/13-softwares/#other-useful-dnf-options)
- [Autoremove Command](https://dnf.readthedocs.io/en/latest/command_ref.html#autoremove-command-label)
- [Clean Command](https://dnf.readthedocs.io/en/latest/command_ref.html#clean-command-label)

### Steps

1. To remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed:

    ```sh
    sudo dnf autoremove
    ```

2. To delete cached package files and any data that have been left behind:

    ```sh
    sudo dnf clean all
    ```
