# UFW

## Description

Uncomplicated Firewall (UFW) is a program for managing a netfilter firewall designed to be easy to use. It uses a command-line interface consisting of a small number of simple commands, and uses iptables for configuration.

## Directory

- [UFW](#ufw)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Enablement](#enablement)
    - [Description](#description-1)
    - [References](#references-1)
    - [Enabling Firewall](#enabling-firewall)
    - [Disabling Firewall](#disabling-firewall)
    - [Resetting Firewall](#resetting-firewall)
  - [Firewall Status](#firewall-status)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps)
  - [Allow Rule](#allow-rule)
    - [Description](#description-3)
    - [References](#references-3)
    - [Custom Allow Rule](#custom-allow-rule)
    - [Allow Rule by Application](#allow-rule-by-application)
    - [Allow Rule by Service](#allow-rule-by-service)
  - [Deny Rule](#deny-rule)
    - [Description](#description-4)
    - [References](#references-4)
    - [Steps](#steps-1)
  - [Delete Rule](#delete-rule)
    - [Description](#description-5)
    - [References](#references-5)
    - [Steps](#steps-2)

## References

- [Uncomplicated Firewall](https://en.wikipedia.org/wiki/Uncomplicated_Firewall)

---

## Enablement

### Description

This details the process of enabling, disabling, or resetting the system firewall using `ufw`.

### References

- [Enabling UFW](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu#step-4-enabling-ufw)
- [Disable or Reset Firewall](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu#step-9-disable-or-reset-firewall)

### Enabling Firewall

1. [Check the status](#firewall-status) and verify that the firewall is currently inactive.

2. To enable the firewall:

    ```sh
    sudo ufw enable
    ```

### Disabling Firewall

1. To disable the firewall:

    ```sh
    sudo ufw disable
    ```

### Resetting Firewall

1. To reset current firewall rules back to its defaults:

    ```sh
    sudo ufw reset
    ```

---

## Firewall Status

### Description

This details the process of checking the system firewall's status and rules using `ufw`.

### References

- [Checking UFW Status and Rules](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu#step-8-checking-ufw-status-and-rules)

### Steps

1. To check the current status and rules of the system firewall:

    ```sh
    sudo ufw status
    ```

2. If the firewall is currently inactive, it will return an output like the following sample:

    ```
    Status: inactive
    ```

3. **On the other hand**, if the firewall is active with rules set, it will return an output like the following sample:

    ```
    Status: active

    To                         Action      From
    --                         ------      ----
    9998                       DENY        Anywhere
    9997                       DENY        Anywhere
    9998 (v6)                  DENY        Anywhere (v6)
    9997 (v6)                  DENY        Anywhere (v6)

    9997                       DENY OUT    Anywhere
    9997 (v6)                  DENY OUT    Anywhere (v6)
    ```

4. **Alternatively**, you can add the `numbered` option to have the currently active rules listed in a numbered order:

    ```sh
    sudo ufw status numbered
    ```

    Sample output:

    ```
    Status: active

        To                         Action      From
        --                         ------      ----
    [ 1] 9998                       DENY IN     Anywhere
    [ 2] 9997                       DENY OUT    Anywhere                   (out)
    [ 3] 9997                       DENY IN     Anywhere
    [ 4] 9998 (v6)                  DENY IN     Anywhere (v6)
    [ 5] 9997 (v6)                  DENY OUT    Anywhere (v6)              (out)
    [ 6] 9997 (v6)                  DENY IN     Anywhere (v6)
    ```

    > [!TIP]  
    > These ordered numbers could be referenced such as when [deleting a rule](#delete-rule).

---

## Allow Rule

### Description

This details the process of adding allow rules of various types with `ufw`.

### References

- [How to Set Up a Firewall with UFW on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)

### Custom Allow Rule

This details the process of adding a custom allow rule with `ufw`:

1. To allow connections to a single specific port:

    ```sh
    sudo ufw allow <port-number>
    ```

    This automatically allows connections to the specified `<port-number>` (i.e. `2222`) for both available protocols, `tcp` and `udp`.

2. To allow connections to a single specific port and specific protocol, simply append the protocol at the end of the port number leading with a forward slash:

    ```sh
    sudo ufw allow <port-number>/<protocol>
    ```

    As an example, to configure the firewall to allow connections to the `port-number`, `2222` and `protocol`, `tcp`:

    ```sh
    sudo ufw allow 2222/tcp
    ```

3. Carrying over the same conventions we have established, should we want to allow connections to a range of ports:

    ```sh
    sudo ufw allow <start-port-number>-<end-port-number>
    ```

    As an example, to configure the firewall to allow connections to ports between `1000` and `2000`:

    ```sh
    sudo ufw allow 1000-2000
    ```

### Allow Rule by Application

This details the process of adding allow rules using available application profiles:

1. List down the available application profiles `ufw` could utilise in adding rules:

    ```sh
    sudo ufw app list
    ```

    Sample output:

    ```
    Available applications:
      OpenSSH
    ```

2. To allow connections predefined by a particular application profile:

    ```sh
    sudo ufw allow <application-profile>
    ```

    As an example, to configure the firewall to allow connections predefined by `OpenSSH`:

    ```sh
    sudo ufw allow OpenSSH
    ```

### Allow Rule by Service

This details the process of adding allow rules by service name:

1. To allow connections predefined by a particular service:

    ```sh
    sudo ufw allow <service-name>
    ```

    As an example, to configure the firewall to allow connections predefined by the `sshd` service:

    ```sh
    sudo ufw allow sshd
    ```

    This will add rules for allowing connections to ports and protocols predefined by the named service in the `/etc/services` file.

---

## Deny Rule

### Description

This details the process of adding deny rules with `ufw`.

### References

- [Denying Connections](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu#step-6-denying-connections)

### Steps

> [!TIP]  
> A lot of the command conventions are consistent with adding [Allow Rules](#allow-rule).

1. To deny incoming connections to a single specific port:

    ```sh
    sudo ufw deny in <port-number>
    ```

    As an example, to configure the firewall to deny incoming connections to the `port-number`, `2222`:

    ```sh
    sudo ufw deny in 2222
    ```

2. To deny outgoing connections to a single specific port:

    ```sh
    sudo ufw deny out <port-number>
    ```

    As an example, to configure the firewall to deny outgoing connections to the `port-number`, `2222`:

    ```sh
    sudo ufw deny out 2222
    ```

---

## Delete Rule

### Description

This details the process of deleting firewall rules with `ufw`.

### References

- [Deleting Rules](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu#step-7-deleting-rules)

### Steps

1. Get a [numbered list](#firewall-status) of the system's currently active firewall rules.

    Sample output:

    ```
    Status: active

        To                         Action      From
        --                         ------      ----
    [ 1] 22                         ALLOW IN    15.15.15.0/24
    [ 2] 80                         ALLOW IN    Anywhere
    ```

2. To delete a specific rule by its number in the list:

    ```sh
    sudo ufw delete <rule-number>
    ```

    As an example, to delete the rule number `2` (i.e. `80`):

    ```sh
    sudo ufw delete 2
    ```
