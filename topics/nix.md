# Nix

## Description

Nix is a cross-platform package manager for Unix-like systems and a functional language to configure those systems.

## Directory

- [Nix](#nix)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Post-Install Setup](#post-install-setup)
  - [Usage](#usage)
    - [Description](#description-2)
    - [References](#references-2)
    - [Listing Available Channels](#listing-available-channels)
    - [Adding a Channel](#adding-a-channel)
    - [Updating a Channel](#updating-a-channel)
    - [Install Software](#install-software)
    - [Update Software](#update-software)
    - [Remove Software](#remove-software)
    - [Search Software](#search-software)
    - [Clean Up](#clean-up)

## References

- [NixOS](https://nixos.org)
- [Nix](https://github.com/NixOS/nix)
- [ArchWiki](https://wiki.archlinux.org/title/Nix)
- [Wikipedia](https://en.wikipedia.org/wiki/Nix_(package_manager))

---

## Setup

### Description

This details the process of installing and setting up Nix as a package manager.

### References

- [Installing Lix](https://lix.systems/install)
- [The Lix Installer Usage](https://git.lix.systems/lix-project/lix-installer#usage)
- [Installation](https://wiki.archlinux.org/title/Nix#Installation)
- [nix-installer](https://github.com/NixOS/nix-installer)
- [Nix - The Best Package Manager](https://youtu.be/BwEIXIjLTNs)
- [Error: experimental Nix feature ‘nix-command’ is disabled](https://discourse.nixos.org/t/error-experimental-nix-feature-nix-command-is-disabled/18089)

### Installation

1. There are multiple ways of installing Nix. Install Nix using **ONE** of the following installation methods:

   - **(Recommended)** The (experimental) Nix installer:

        ```sh
        curl --proto '=https' --tlsv1.2 -sSf -L https://artifacts.nixos.org/experimental-installer | sh -s -- install
        ```

   - The Lix installer:

        ```sh
        curl --proto '=https' --tlsv1.2 -sSf -L https://install.lix.systems/lix | sh -s -- install
        ```

        **(Optional)** If you wish to install Nix following (install) plans other than the default, such as one dedicated to the Steam Deck, simply add the name of the planner (i.e. `steam-deck`) to the install command:

        ```sh
        curl --proto '=https' --tlsv1.2 -sSf -L https://install.lix.systems/lix | sh -s -- install steam-deck
        ```

        In most cases, leaving the installer to determine a plan best suited for your system is sufficient.

2. Follow the guided installation steps provided by the installer until completion.

3. Start a new shell session or reboot the system to start using Nix.

4. Go through the [post-installation setup steps](#post-install-setup) to complete the Nix setup.

### Post-Install Setup

This details the post-installation steps for a complete Nix setup:

1. By default, there might not be a [channel (i.e. software repository) available on the system](#listing-available-channels) for you to install packages from. If that is the case, add the following channel(s) on the system:

   - `nixpkgs`: `https://nixos.org/channels/nixpkgs-unstable`

2. Enable Nix's experimental features on the system:

   - Create the Nix configuration directory if it does not exist already:

        ```sh
        mkdir -p ~/.config/nix
        ```

   - Create or update the configuration file:

        ```sh
        nano ~/.config/nix/nix.conf
        ```

   - Add and save the following line to the configuration file:

        ```sh
        experimental-features = nix-command flakes
        ```

3. **(Optional)** On a non-NixOS system, graphical applications (i.e. Alacritty, MPV, etc.) may not work out of the box:

   - [Install and set up NixGL](https://github.com/irfanhakim-as/linux-wiki/blob/master/topics/nix.md#installing-nixgl) on the system.
   - Going forward, if you have [installed any Nix packages](#install-software) that are graphical and would not work without NixGL, [update the application desktop file](https://github.com/irfanhakim-as/linux-wiki/blob/master/topics/nix.md#update-application-desktop-file-to-use-nixgl) to use NixGL by default.

---

## Usage

### Description

This details some common usage steps for basic package management using Nix.

### References

- [Usage](https://wiki.archlinux.org/title/Nix#Usage)
- [Quick Start](https://docs.lix.systems/manual/lix/stable/quick-start.html)
- [nix-channel](https://docs.lix.systems/manual/lix/stable/command-ref/nix-channel.html)
- [nix-env](https://docs.lix.systems/manual/lix/stable/command-ref/nix-env.html)
- [nix search](https://docs.lix.systems/manual/lix/stable/command-ref/new-cli/nix3-search.html)

### Listing Available Channels

This details the process of listing available Nix software repositories (channels) on the system:

1. To list available channels on the system:

    ```sh
    nix-channel --list
    ```

    Sample output:

    ```
    nixpkgs https://nixos.org/channels/nixpkgs-unstable
    ```

### Adding a Channel

This details the process of adding a Nix software repository (channel) to the system:

1. To add a Nix channel on the system:

    ```sh
    nix-channel --add <url> [name]
    ```

    - Replace `<url>` with the URL of the channel you wish to add (i.e. `https://nixos.org/channels/nixpkgs-unstable`)
    - **(Optional)** Replace `[name]` with a descriptive name for the channel (i.e. `nixpkgs`)
    - For example:

        ```sh
        nix-channel --add https://nixos.org/channels/nixpkgs-unstable nixpkgs
        ```

2. [Update the channel](#updating-a-channel) to get the latest collection of available packages and updates.

### Updating a Channel

This details the process of updating a Nix software repository (channel) on the system:

1. To update a particular channel or multiple of them on the system:

    ```sh
    nix-channel --update [names]
    ```

    - For example, to update a single channel called `nixpkgs`:

        ```sh
        nix-channel --update nixpkgs
        ```

    - Or, in a different example, to update a channel called `channel1` and another called `channel2`:

        ```sh
        nix-channel --update channel1 channel2
        ```

2. **Alternatively**, to update all available channels on the system:

    ```sh
    nix-channel --update
    ```

### Install Software

This details the process of installing software:

1. [Update the available channels](#updating-a-channel) on the system to get the latest collection of available packages and updates.

2. To install a single Nix package from a channel:

    ```sh
    nix-env -iA <channel>.<package>
    ```

    As an example, to install the `hello` package from the `nixpkgs` channel:

    ```sh
    nix-env -iA nixpkgs.hello
    ```

3. To install multiple packages in a single command, simply separate each package with a space:

    ```sh
    nix-env -iA <channel>.<package> <channel>.<package> <channel>.<package>
    ```

### Update Software

This details the process of updating a software or the system:

1. [Update the available channels](#updating-a-channel) on the system to get the latest collection of available packages and updates.

2. To update a particular Nix package or multiple of them, specify the name of the package(s):

    ```sh
    nix-env -u <package>
    ```

    - For example, to update a single package called `hello`:

        ```sh
        nix-env -u hello
        ```

    - Or, in a different example, to update a package called `package1` and another called `package2`:

        ```sh
        nix-env -u package1 package2
        ```

3. To update the entire system:

    ```sh
    nix-env -u
    ```

### Remove Software

This details the process of uninstalling software:

1. To uninstall a single Nix package from the system:

    ```sh
    nix-env -e <package>
    ```

    As an example, to uninstall the `hello` package:

    ```sh
    nix-env -e hello
    ```

2. To uninstall multiple packages in a single command, simply separate each package with a space:

    ```sh
    nix-env -e <package> <package> <package>
    ```

3. **(Optional)** Uninstalling a package does not automatically get rid of orphaned packages and dependencies. To get rid of them from time to time, perform a [clean up](#clean-up).

### Search Software

This details the process of searching software:

1. To search for a Nix package:

    ```sh
    nix search nixpkgs <package>
    ```

    As an example, to search for the `hello` package:

    ```sh
    nix search nixpkgs hello
    ```

### Clean Up

This details the process of cleaning up the system:

1. To remove old Nix package versions and unused dependencies:

    ```sh
    nix-collect-garbage
    ```

2. **(Optional)** To also remove old generations (snapshots of previous package installations) and perform a more thorough cleanup of the system:

    ```sh
    nix-collect-garbage --delete-old
    ```
