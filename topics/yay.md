# Yay

> [!NOTE]  
> Yay can be used as a direct replacement for [Pacman](pacman.md), with the added benefit of access to the [AUR](https://aur.archlinux.org). Its usage is identical to Pacman.

## Description

Yet Another Yogurt - An AUR Helper Written in Go.

## Directory

- [Yay](#yay)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Install Software](#install-software)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)
  - [Update Software](#update-software)
    - [Description](#description-3)
    - [References](#references-3)
    - [Steps](#steps-2)
  - [Remove Software](#remove-software)
    - [Description](#description-4)
    - [References](#references-4)
    - [Steps](#steps-3)
  - [Search Software](#search-software)
    - [Description](#description-5)
    - [References](#references-5)
    - [Steps](#steps-4)
  - [Clean Up](#clean-up)
    - [Description](#description-6)
    - [References](#references-6)
    - [Steps](#steps-5)

## References

- [Yay](https://github.com/Jguer/yay)

---

## Setup

### Description

This details the process of installing and setting up `yay`.

### References

- [Installation - Binary](https://github.com/Jguer/yay#binary)
- [Development packages upgrade](https://github.com/Jguer/yay#development-packages-upgrade)

### Steps

1. [Install](package-manager.md#install-software) the following packages using `pacman`:

   - `base-devel`
   - `git`

2. Clone the source repository to the home directory:

    ```sh
    git clone https://aur.archlinux.org/yay-bin.git ~/.yay-bin
    ```

3. Enter the local repository:

    ```sh
    cd ~/.yay-bin
    ```

4. Build and install the package:

    ```sh
    makepkg -si
    ```

5. Once `yay` is installed, generate a development package database for `*-git` packages that were installed without `yay`:

    ```sh
    yay -Y --gendb
    ```

6. Check for development package updates:

    ```sh
    yay -Syu --devel
    ```

7. Make development package updates permanently enabled:

    ```sh
    yay -Y --devel --save
    ```

---

## Install Software

### Description

This details the process of installing software.

### References

- [Pacman: Install Software](pacman.md#install-software)

### Steps

1. To install a single package:

    ```sh
    yay -S <package>
    ```

    As an example, to install the `vim` package:

    ```sh
    yay -S vim
    ```

2. **Optionally**, use the `--noconfirm` flag to automatically answer "yes" to all prompts:

    ```sh
    yay -S --noconfirm <package>
    ```

3. To install multiple packages in a single command, simply separate each package with a space:

    ```sh
    yay -S <package1> <package2> <package3>
    ```

---

## Update Software

### Description

This details the process of updating a software or the system.

### References

- [Pacman: Update Software](pacman.md#update-software)

### Steps

1. To update a particular package or multiple of them, simply [reinstall](#install-software) the package(s).

2. To update the entire system:

    ```sh
    yay -Syu
    ```

---

## Remove Software

### Description

This details the process of uninstalling software.

### References

- [Pacman: Remove Software](pacman.md#remove-software)

### Steps

1. To uninstall a single package along with its dependencies:

    ```sh
    yay -Rns <package>
    ```

    As an example, to uninstall the `vim` package:

    ```sh
    yay -Rns vim
    ```

2. **Optionally**, use the `--noconfirm` flag to automatically answer "yes" to all prompts:

    ```sh
    yay -Rns --noconfirm <package>
    ```

3. To uninstall multiple packages in a single command, simply separate each package with a space:

    ```sh
    yay -Rns <package1> <package2> <package3>
    ```

---

## Search Software

### Description

This details the process of searching software.

### References

- [Pacman: Search Software](pacman.md#search-software)

### Steps

1. To search for a package:

    ```sh
    yay -Ss <package>
    ```

    As an example, to search for the `vim` package:

    ```sh
    yay -Ss vim
    ```

---

## Clean Up

### Description

This details the process of cleaning up the system.

### References

- [Pacman: Clean Up](pacman.md#clean-up)

### Steps

1. To remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed:

    ```sh
    yay -Rns $(yay -Qdtq)
    ```

2. To delete cached package files of packages that have been installed or removed:

    ```sh
    yay -Scc
    ```
