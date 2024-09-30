# Firewalld

## Description

Firewalld provides a dynamically managed firewall with support for network/firewall zones that define the trust level of network connections or interfaces. It has support for IPv4, IPv6 firewall settings, ethernet bridges and IP sets.

## Directory

- [Firewalld](#firewalld)
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
    - [Allow Rule by Service](#allow-rule-by-service)
  - [Delete Rule](#delete-rule)
    - [Description](#description-4)
    - [References](#references-4)
    - [Steps](#steps-1)
  - [Make Changes Permanent](#make-changes-permanent)
    - [Description](#description-5)
    - [References](#references-5)
    - [Steps](#steps-2)
  - [Reload Firewall](#reload-firewall)
    - [Description](#description-6)
    - [References](#references-6)
    - [Steps](#steps-3)

## References

- [Firewalld](https://firewalld.org)

---

## Enablement

### Description

This details the process of enabling, disabling, or resetting the system firewall.

### References

- [System service commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#system-service-commands)
- [How to reset firewalld rules to Fedora default?](https://discussion.fedoraproject.org/t/how-to-reset-firewalld-rules-to-fedora-default/75627)

### Enabling Firewall

1. [Check the status](#firewall-status) and verify that the firewall is currently inactive.

2. To enable the firewall, [start and enable](systemd.md#enable-service) the `firewalld.service` service.

### Disabling Firewall

1. To disable the firewall, [stop](systemd.md#stop-service) and [disable](systemd.md#disable-service) the `firewalld.service` service.

### Resetting Firewall

1. To reset current firewall rules back to its (zone) defaults:

    ```sh
    sudo firewall-cmd --load-zone-defaults=<zone-name> --permanent
    ```

    As an example, to reset the firewall to its defaults for the default zone, `public`:

    ```sh
    sudo firewall-cmd --load-zone-defaults=public --permanent
    ```

---

## Firewall Status

### Description

This details the process of checking the system firewall's status and rules.

### References

- [System service commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#system-service-commands)
- [Basic firewalld configuration and management commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#basic-firewalld-configuration-and-management-commands)

### Steps

1. To check the current status of the system firewall, you can simply check the [status](systemd.md#service-status) of the `firewalld.service` service or by running:

    ```sh
    sudo firewall-cmd --state
    ```

2. If the firewall is currently inactive, it will return an output as such:

    ```
    not running
    ```

    **On the other hand**, if the firewall is active, it will return the following output:

    ```
    running
    ```

3. To check the currently active firewall rules:

    ```sh
    sudo firewall-cmd --list-all
    ```

---

## Allow Rule

### Description

This details the process of adding allow rules of various types.

### References

- [Port management commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#service-management-commands)
- [firewalld one-liner define both TCP and UDP](https://a.opnxng.com/exchange/unix.stackexchange.com/questions/612189/firewalld-one-liner-define-both-tcp-and-udp)
- [Service management commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#service-management-commands)

### Custom Allow Rule

This details the process of adding a custom allow rule using port numbers and protocols:

1. To allow connections to a single specific port and protocol:

    ```sh
    sudo firewall-cmd --add-port=<port-number>/<protocol>
    ```

    This allows connections to the specified `<port-number>` (i.e. `2222`) and `<protocol>` (i.e. `tcp`) on the default firewall zone.

2. To allow connections to a single specific port and multiple protocols:

    ```sh
    sudo firewall-cmd --add-port=<port-number>/{<protocol1>,<protocol2>}
    ```

    As an example, to configure the firewall to allow connections to the `port-number`, `2222` and `protocol`s, `tcp` and `udp`:

    ```sh
    sudo firewall-cmd --add-port=2222/{tcp,udp}
    ```

3. Carrying over the same conventions we have established, should we want to allow connections to a range of ports for a specific protocol:

    ```sh
    sudo firewall-cmd --add-port=<start-port-number>-<end-port-number>/<protocol>
    ```

    As an example, to configure the firewall to allow connections to ports between `1000` and `2000` for the `protocol` `tcp`:

    ```sh
    sudo firewall-cmd --add-port=1000-2000/tcp
    ```

4. After making your (permanent) changes, [reload the firewall](#reload-firewall) to apply them.

### Allow Rule by Service

This details the process of adding allow rules by service name:

1. List down the available services you could add to your firewall:

    ```sh
    firewall-cmd --get-services
    ```

    Sample output:

    ```
    RH-Satellite-6 RH-Satellite-6-capsule amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit collectd condor-collector ctdb dhcp dhcpv6 dhcpv6-client distcc dns dns-over-tls docker-registry docker-swarm dropbox-lansync elasticsearch etcd-client etcd-server finger foreman foreman-proxy freeipa-4 freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp galera ganglia-client ganglia-master git grafana gre high-availability http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target isns jenkins kadmin kdeconnect kerberos kibana klogin kpasswd kprop kshell kube-apiserver ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve matrix mdns memcache minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql murmur mysql nbd nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy prometheus proxy-dhcp ptp pulseaudio puppetmaster quassel radius rdp redis redis-sentinel rpc-bind rquotad rsh rsyncd rtsp salt-master samba samba-client samba-dc sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync spotify-sync squid ssdp ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls telnet tentacle tftp tftp-client tile38 tinc tor-socks transmission-client upnp-client vdsm vnc-server wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server zabbix-agent zabbix-server
    ```

2. To allow connections predefined by a particular service:

    ```sh
    sudo firewall-cmd --add-service=<service-name>
    ```

    As an example, to configure the firewall to allow connections predefined by the `ssh` service:

    ```sh
    sudo firewall-cmd --add-service=ssh
    ```

    This will add rules for allowing connections to ports and protocols predefined by the named service.

3. To make these changes persistent across a system reboot or a [firewall reload](#reload-firewall), make the changes [permanent](#make-changes-permanent).

---

## Delete Rule

### Description

This details the process of deleting firewall rules.

### References

- [Port management commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#service-management-commands)
- [Service management commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#service-management-commands)

### Steps

1. Get a list of the system's currently active [firewall rules](#firewall-status).

    Sample output:

    ```
    public (active)
    target: default
    icmp-block-inversion: no
    interfaces: enp6s18
    sources:
    services: cockpit dhcpv6-client
    ports: 2222/tcp
    protocols:
    forward: no
    masquerade: no
    forward-ports:
    source-ports:
    icmp-blocks:
    rich rules:
    ```

2. To delete a specific rule by its port and protocol:

    ```sh
    sudo firewall-cmd --remove-port=<port-number>/<protocol>
    ```

    As an example, to delete the rule allowing connections to the `port-number`, `2222` and `protocol`, `tcp`:

    ```sh
    sudo firewall-cmd --remove-port=2222/tcp
    ```

3. To delete a specific rule by its service name:

    ```sh
    sudo firewall-cmd --remove-service=<service-name>
    ```

    As an example, to delete the rule predefined by the `ssh` service:

    ```sh
    sudo firewall-cmd --remove-service=ssh
    ```

4. To make these changes persistent across a system reboot or a [firewall reload](#reload-firewall), make the changes [permanent](#make-changes-permanent).

---

## Make Changes Permanent

### Description

This details the simple process of making changes to the firewall permanent.

### References

- [Saving your changes](https://docs.rockylinux.org/guides/security/firewalld-beginners#saving-your-changes)

### Steps

Any changes made to the firewall without explicitly making them permanent will take effect immediately but will reset after a firewall reload or a system reboot. To make changes permanent:

1. Either make the rule permanent from the get-go using the `--permanent` flag:

   - For example, add the `--permanent` flag when [adding an allow rule](#allow-rule):

        ```sh
        sudo firewall-cmd --add-port=<port-number>/<protocol> --permanent
        ```

        or likewise, when [deleting a rule](#delete-rule):

        ```sh
        sudo firewall-cmd --remove-port=<port-number>/<protocol> --permanent
        ```

   - This method will make changes permanent, but the changes themselves will not take effect immediately. After making your permanent changes, [reload the firewall](#reload-firewall) to apply them.

2. **Alternatively**, the safer method of making changes permanent is by making runtime (i.e. temporary) changes as usual, and then making all the new changes permanent with a single command once you have verified they are working as intended.

   - As an example, [add an allow rule](#allow-rule) or [delete a rule](#delete-rule) as usual (i.e. without using the `--permanent` flag)

   - Verify that the current firewall configuration, with the changes you have made, is working as intended.

   - Save the current firewall configuration, including your changes, permanently:

        ```sh
        sudo firewall-cmd --runtime-to-permanent
        ```

---

## Reload Firewall

### Description

This details the simple process of reloading new changes to the firewall.

### References

- [Basic firewalld configuration and management commands](https://docs.rockylinux.org/guides/security/firewalld-beginners#basic-firewalld-configuration-and-management-commands)

### Steps

1. Any [permanent changes](#make-changes-permanent) made to the firewall rules will not take effect until the firewall is reloaded. To reload the firewall:

    ```sh
    sudo firewall-cmd --reload
    ```

    > [!TIP]  
    > Firewall rule changes that have not been made [permanent](#make-changes-permanent) will disappear after a firewall reload or a system reboot.
