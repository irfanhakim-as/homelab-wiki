# Timeshift

## Description

Timeshift for Linux is an application that provides functionality similar to the System Restore feature in Windows and the Time Machine tool in Mac OS. Timeshift protects your system by taking incremental snapshots of the file system at regular intervals.

## Directory

- [Timeshift](#timeshift)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Configuration](#configuration)

## References

- [Timeshift](https://github.com/linuxmint/timeshift)
- [ArchWiki](https://wiki.archlinux.org/title/Timeshift)

---

## Setup

### Description

This details the installation and configuration steps of Timeshift as a system snapshot solution.

### References

- [Installation](https://github.com/linuxmint/timeshift/blob/master/README.md#installation)

### Installation

This details the installation process of Timeshift:

1. [Install](package-manager.md#install-software) the `timeshift` package using the system's package manager (i.e. `apt`).

2. [Configure Timeshift](#configuration) before use.

### Configuration

This details the recommended configuration options of Timeshift:

1. Backup the existing Timeshift configuration file:

    ```sh
    sudo cp /etc/timeshift/timeshift.json /etc/timeshift/timeshift.json.bak
    ```

    If the default configuration file does not yet exist, create an initial snapshot first:

    ```sh
    sudo timeshift --create --comment 'initial snapshot'
    ```

2. Update the Timeshift configuration:

   - Update the configuration file:

      ```sh
      sudo nano /etc/timeshift/timeshift.json
      ```

   - If you have more than single storage attached to the system, ensure that Timeshift is configured to use the correct storage device for storing snapshots:

      In most cases, we would want the root storage device. Run the following command to get its UUID:

      ```sh
      sudo blkid | grep -i root | grep -o 'UUID="[^"]*"' | head -n1 | cut -d'"' -f2
      ```

      Sample output:

      ```
        8f9c2b41-7e21-4f9d-beb4-2a4f3bcd0ac6
      ```

      In the configuration file, update the following setting with the UUID of the intended storage device (i.e. `8f9c2b41-7e21-4f9d-beb4-2a4f3bcd0ac6`) accordingly:

      ```diff
      - "backup_device_uuid" : "3a6f79c2-42b7-4f8e-8b8a-91d4dc9ea2c3",
      + "backup_device_uuid" : "8f9c2b41-7e21-4f9d-beb4-2a4f3bcd0ac6",
      ```

   - Configure Timeshift to take daily snapshots:

      ```diff
      - "schedule_daily" : "false",
      + "schedule_daily" : "true",
      ```

   - **(Optional)** By default, Timeshift excludes any home directories found on the system from snapshots. Add to the list any other paths you wish to exclude:

      ```diff
        "exclude" : [
          "/home/myuser/**",
      -   "/root/**"
      +   "/root/**",
      +   "/path/to/directory/**"
        ],
      ```

3. Save changes made to the configuration file.
