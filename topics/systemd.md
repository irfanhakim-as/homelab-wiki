# Systemd

## Description

Systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system.

## Directory

- [Systemd](#systemd)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Systemctl Usage](#systemctl-usage)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
    - [Start Service](#start-service)
    - [Enable Service](#enable-service)
    - [Stop Service](#stop-service)
    - [Disable Service](#disable-service)
    - [Restart Service](#restart-service)
    - [Service Status](#service-status)
    - [Reload Systemd Manager Configuration](#reload-systemd-manager-configuration)
    - [List Systemd Units](#list-systemd-units)

## References

- [Systemd](https://wiki.archlinux.org/title/Systemd)

---

## Systemctl Usage

### Description

This details how to use the `systemctl` command to manage services on a Linux system.

### References

- [Basic systemctl usage](https://wiki.archlinux.org/title/Systemd#Basic_systemctl_usage)

### Steps

> [!NOTE]  
> In the following examples, we'll assume the name of the service is `unit.service`. Replace it with the actual name of the service you wish to manage.

### Start Service

1. To start a service, once, now:

    ```sh
    sudo systemctl start unit.service
    ```

### Enable Service

1. To enable a service to start on every boot:

    ```sh
    sudo systemctl enable unit.service
    ```

2. **Alternatively**, to start a service now and enable the service on boot in a single command:

    ```sh
    sudo systemctl enable --now unit.service
    ```

### Stop Service

1. To stop a service, once, now:

    ```sh
    sudo systemctl stop unit.service
    ```

### Disable Service

1. To disable an enabled service from starting on every boot:

    ```sh
    sudo systemctl disable unit.service
    ```

### Restart Service

1. To restart a service, useful for applying changes to certain services:

    ```sh
    sudo systemctl restart unit.service
    ```

### Service Status

1. To check the status of a service:

    ```sh
    sudo systemctl status unit.service
    ```

### Reload Systemd Manager Configuration

Some operations may require reloading the `systemd` manager configuration.

1. To reload the `systemd` manager configuration:

    ```sh
    sudo systemctl daemon-reload
    ```

### List Systemd Units

This details how to list down all or specific `systemd` unit(s) on the system:

1. To list down all system-wide `systemd` unit(s) on the system:

    ```sh
    systemctl list-unit-files --no-pager
    ```

    You may also use `grep` if you wish to filter the results for specific unit(s):

    ```sh
    systemctl list-unit-files --no-pager | grep -E <filter>
    ```

    For example:

    ```sh
    systemctl list-unit-files --no-pager | grep -E 'unit.service'
    ```

2. To list down all user-specific `systemd` unit(s) on the system:

    ```sh
    systemctl --user list-unit-files --no-pager
    ```

    You may also use `grep` if you wish to filter the results for specific unit(s):

    ```sh
    systemctl --user list-unit-files --no-pager | grep -E <filter>
    ```

    For example:

    ```sh
    systemctl --user list-unit-files --no-pager | grep -E 'unit.service'
    ```

3. **Alternatively**, to list down **ALL** `systemd` unit(s) on the system:

    ```sh
    { systemctl list-unit-files --no-pager; systemctl --user list-unit-files --no-pager; }
    ```

    You may also use `grep` if you wish to filter the results for specific unit(s):

    ```sh
    { systemctl list-unit-files --no-pager; systemctl --user list-unit-files --no-pager; } | grep -E <filter>
    ```

    For example:

    ```sh
    { systemctl list-unit-files --no-pager; systemctl --user list-unit-files --no-pager; } | grep -E 'unit.service'
    ```
