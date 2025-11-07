# Fail2Ban

## Description

Fail2Ban scans log files like `/var/log/auth.log` and bans IP addresses conducting too many failed login attempts. It does this by updating system firewall rules to reject new connections from those IP addresses, for a configurable amount of time.

## Directory

- [Fail2Ban](#fail2ban)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Setting Up Fail2Ban](#setting-up-fail2ban)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [References](#references-2)
    - [How to Configure Fail2Ban](#how-to-configure-fail2ban)
    - [Recommended Configurations](#recommended-configurations)
  - [Usage](#usage)
    - [Description](#description-3)
    - [References](#references-3)
    - [Return Jail Status](#return-jail-status)
    - [Undo Bans](#undo-bans)

## References

- [Fail2Ban](https://github.com/fail2ban/fail2ban)
- [ArchWiki](https://wiki.archlinux.org/title/Fail2ban)

---

## Setup

### Description

This details how to install and set up Fail2Ban on a system.

### References

- [Install Fail2Ban On CentOS 7](https://www.liquidweb.com/blog/install-fail2ban-on-centos-7)
- [How Fail2Ban Works to Protect Services on a Linux Server](https://www.digitalocean.com/community/tutorials/how-fail2ban-works-to-protect-services-on-a-linux-server)

### Setting Up Fail2Ban

This details how to set Fail2Ban up on a system:

1. [Install](package-manager.md#install-software) the `fail2ban` package on the server using your package manager (i.e. `apt`).

2. Configure Fail2Ban on the system according to the list of [recommended configurations](#recommended-configurations).

3. Ensure to [start and enable](systemd.md#enable-service) the `fail2ban.service` service on the system to have Fail2Ban run automatically on boot.

---

## Configuration

### Description

This details how to configure Fail2Ban and some recommended configuration options.

### References

- [Fail2ban Tutorial | How to Secure Your Server](https://youtu.be/kgdoVeyoO2E)
- [Configuration](https://wiki.archlinux.org/title/Fail2ban#Configuration)
- [How to Secure Your Linux Server with Fail2Ban Configuration](https://www.hostinger.com/my/tutorials/fail2ban-configuration)
- [How to assign a friendly name to a port number in Linux?](https://unix.stackexchange.com/questions/611406/how-to-assign-a-friendly-name-to-a-port-number-in-linux)
- [In Fail2Ban, How to Change the SSH port number?](https://serverfault.com/questions/382858/in-fail2ban-how-to-change-the-ssh-port-number)

### How to Configure Fail2Ban

This details the general steps to configure Fail2Ban:

1. It is not recommended to configure Fail2Ban using the default supplied configuration files; `fail2ban.conf` and `jail.conf`. If there isn't one already, create a copy of both of them in the same directory:

    ```sh
    sudo cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    ```

    Moving forward, any configuration changes should be made to either of these files; `fail2ban.local` and `jail.local`.

2. Depending on the configuration you wish to make, update the appropriate configuration file accordingly (i.e. `fail2ban.local` or `jail.local`):

    ```sh
    sudo nano /etc/fail2ban/fail2ban.local
    ```

    Save changes made to the configuration file.

3. To apply all changes made to either of the configuration files, [restart](systemd.md#restart-service) the `fail2ban.service` service.

### Recommended Configurations

This details some recommended configuration options for a Fail2Ban setup:

1. [Configure](#how-to-configure-fail2ban) the following important settings in the `jail.local` configuration file:

   - `bantime`: This parameter sets the length of time that a client will be banned for after failed authentication attempt(s).

      ```diff
        # "bantime" is the number of seconds that a host is banned.
      - bantime  = 10m
      + bantime  = 120m
      ```

      In this example, any client that should be banned would be banned for 120 minutes (i.e. 2 hours).

   - `findtime`: This setting determines the time period for failed login attempts.

      ```diff
        # A host is banned if it has generated "maxretry" during the last "findtime"
        # seconds.
      - findtime  = 10m
      + findtime  = 60m
      ```

      In this example, the number of failed authentication attempts are counted in 60 minute intervals (i.e. hourly).

   - `maxretry`: This sets the maximum number of unsuccessful login attempts within the defined time window

      ```diff
        # "maxretry" is the number of failures before a host get banned.
      - maxretry = 5
      + maxretry = 3
      ```

      In this example, the maximum number of failed authentication attempts within the defined time period is set to 3.

   - Based on these sample configuration values, client "A" would be banned for 2 hours, if they fail to authenticate, 3 times, within 60 minutes.

2. Enable the `sshd` jail on the system, if it has not been enabled by default:

   - [Update](#how-to-configure-fail2ban) the `jail.local` configuration file.

   - Locate the following `[sshd]` section in the file and add the following line to enable it:

      ```diff
        [sshd]
      -
      + enabled = true
        # To use more aggressive sshd modes set filter parameter "mode" in jail.local:
        # normal (default), ddos, extra or aggressive (combines all).
        # See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
        #mode   = normal
        port    = ssh
        logpath = %(sshd_log)s
        backend = %(sshd_backend)s
      ```

   - **(Important)** If your server is not using the default SSH port (i.e. `22`), update the `port` value to the updated port number accordingly (i.e. `2222`):

      ```diff
        [sshd]
        enabled = true
        # To use more aggressive sshd modes set filter parameter "mode" in jail.local:
        # normal (default), ddos, extra or aggressive (combines all).
        # See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
        #mode   = normal
      - port    = ssh
      + port    = 2222
        logpath = %(sshd_log)s
        backend = %(sshd_backend)s
      ```

      **Alternatively**, update the predefined `ssh` port number in the system's `services` file:

      ```sh
      sudo nano /etc/services
      ```

      Update the `ssh` port number to the updated port number accordingly (i.e. `2222`):

      ```diff
      - ssh             22/tcp                          # The Secure Shell (SSH) Protocol
      + ssh             2222/tcp                        # The Secure Shell (SSH) Protocol
      ```

   - [Verify](#return-jail-status) that the `sshd` jail is running on the system.

3. After you have finished all of your configurations, do not forget to [restart](systemd.md#restart-service) the `fail2ban.service` service on the system to apply the changes.

---

## Usage

### Description

This details some general usage of Fail2Ban using its terminal-based client.

### References

- [Delete all fail2ban bans in Ubuntu Linux](https://unix.stackexchange.com/questions/286119/delete-all-fail2ban-bans-in-ubuntu-linux)

### Return Jail Status

This details how to return the status of a specific jail or all jails on the system:

1. To get a list of enabled jails on the system, run the following command:

    ```sh
    sudo fail2ban-client status
    ```

    Sample output:

    ```
      Status
      |- Number of jail:      1
      `- Jail list:   sshd
    ```

    This indicates that the only jail enabled on the system is `sshd`.

2. To get the status of a specific jail, specify the name of the jail at the end of the same command:

    ```sh
    sudo fail2ban-client status <jail>
    ```

    For example:

    ```sh
    sudo fail2ban-client status sshd
    ```

    Sample output:

    ```
      Status for the jail: sshd
      |- Filter
      |  |- Currently failed: 0
      |  |- Total failed:     0
      |  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
      `- Actions
        |- Currently banned: 0
        |- Total banned:     0
        `- Banned IP list:
    ```

    This output also specifies the number of clients that have been banned by this jail.

### Undo Bans

This details how to undo any bans that have taken place on the system:

1. To unban all clients in all jails and database on the system:

    ```sh
    sudo fail2ban-client unban --all
    ```
