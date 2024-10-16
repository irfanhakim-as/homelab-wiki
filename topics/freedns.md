# FreeDNS

## Description

The site FreeDNS began in 2001 because founder Joshua Anderson wanted to have fun with his domain hobby. He wanted to create a safe environment where other programmers could share domain names with one another at no cost, starting with friends. With that in mind, he launched the site from his domain AFRAID.ORG.

## Directory

- [FreeDNS](#freedns)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Acquiring a Domain](#acquiring-a-domain)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Register a Subdomain](#register-a-subdomain)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)
  - [Dynamic DNS](#dynamic-dns)
    - [Description](#description-3)
    - [References](#references-3)
    - [Ddclient](#ddclient)

## References

- [FreeDNS](https://freedns.afraid.org)

---

## Acquiring a Domain

### Description

This details how to acquire a domain name on FreeDNS.

### References

- [Register a new domain](https://developers.cloudflare.com/registrar/get-started/register-domain)

### Steps

1. Free and public domains are available on FreeDNS that can be used without the need to purchase. [Register a subdomain](#register-a-subdomain) for free instead.

---

## Register a Subdomain

### Description

This details how to register a subdomain on FreeDNS.

### References

- [Set up your own VPN at home with Raspberry Pi](https://notthebe.ee/blog/set-up-your-own-vpn-on-raspberry-pi)

### Steps

1. Visit the [Main Menu](https://freedns.afraid.org/menu) page on a web browser. Log into your FreeDNS account if you have not already.

2. From the centre of the main menu, click the **Domain Registry** menu item.

3. From the list of available domains, click the corresponding link of the domain you wish to register a subdomain to (i.e. `example.com`).

4. In the **Add a new subdomain** form, configure as such:

   - Type: Expand the dropdown and select the DNS type (i.e. `A`)
   - Subdomain: Add the name of the subdomain you wish to register (i.e. `mysubdomain`)
   - Domain: Leave this as the domain you had picked earlier (i.e. `example.com`)
   - Destination: Set this to the Public IP address of your home network (i.e. `237.84.2.178`)
   - Wildcard: Leave this unchecked

    Type in the CAPTCHA and click the **Save!** button.

---

## Dynamic DNS

### Description

This details how to keep DNS records up-to-date dynamically on FreeDNS.

### References

- [Set up your own VPN at home with Raspberry Pi](https://notthebe.ee/blog/set-up-your-own-vpn-on-raspberry-pi)
- [Dynamic DNS clients](https://freedns.afraid.org/scripts/freedns.clients.php)
- [Multiple Domains with ddclient (Namecheap)](https://www.labsrc.com/multiple-domains-with-ddclient-namecheap)

### Ddclient

1. [Provision a VM](../courses/vm.md#creating-a-virtual-machine-from-a-template) that will host the `ddclient` software.

2. On the VM, [install](package-manager.md#install-software) the `ddclient` package using the system package manager.

    When being prompted through the **Package configuration** process, simply press <kbd>Enter</kbd> to skip through the prompts.

3. Determine the name of the active network interface on the system:

    ```sh
    ip link
    ```

    Sample output:

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether fg:LA:Og:mQ:LQ:7Q brd ff:ff:ff:ff:ff:ff
    ```

    In this example, `enp6s18` is the name of the active network interface.

4. Update the ddclient service configuration file:

    ```sh
    sudo nano /etc/default/ddclient
    ````

   - Update the configuration file to be as such:

      ```sh
      # Configuration for ddclient scripts
      # generated from debconf on Thu Oct 10 12:03:05 UTC 2024
      #
      # /etc/default/ddclient

      # Set to "true" if ddclient should be run every time DHCP client ('dhclient'
      # from package isc-dhcp-client) updates the systems IP address.
      run_dhclient="false"

      # Set to "true" if ddclient should be run every time a new ppp connection is
      # established. This might be useful, if you are using dial-on-demand.
      run_ipup="false"

      # Set to "true" if ddclient should run in daemon mode
      # If this is changed to true, run_ipup and run_dhclient must be set to false.
      run_daemon="true"

      # Set the time interval between the updates of the dynamic DNS name in seconds.
      # This option only takes effect if the ddclient runs in daemon mode.
      daemon_interval="300"
      ```

   - Whereby the following values are set as follows:

     - run_dhclient: `"false"`
     - run_ipup: `"false"`
     - run_daemon: `"true"`

5. Update the ddclient dynamic DNS configuration file:

    ```sh
    sudo nano /etc/ddclient.conf
    ````

   - Original content of the configuration file:

      ```conf
      # Configuration file for ddclient generated by debconf
      #
      # /etc/ddclient.conf

      protocol=dyndns2
      use=web, web=checkip.dyndns.com, web-skip='IP Address'
      server=members.dyndns.org
      login=
      password=''
      ```

   - Make the following changes:

      ```diff
        # Configuration file for ddclient generated by debconf
        #
        # /etc/ddclient.conf

      - protocol=dyndns2
      - use=web, web=checkip.dyndns.com, web-skip='IP Address'
      - server=members.dyndns.org
      - login=
      - password=''
      + daemon=5m
      + timeout=10
      + syslog=no # log update msgs to syslog
      + pid=/var/run/ddclient.pid # record PID in file
      + ssl=yes # use ssl-support
      +
      + use=if, if=<network-interface>
      + server=freedns.afraid.org
      + protocol=freedns
      + login=<freedns-username>
      + password=<freedns-password>
      + <domain>
      ```

   - The following is an example of the updated configuration:

      ```conf
      # Configuration file for ddclient generated by debconf
      #
      # /etc/ddclient.conf

      daemon=5m
      timeout=10
      syslog=no # log update msgs to syslog
      pid=/var/run/ddclient.pid # record PID in file
      ssl=yes # use ssl-support

      use=if, if=enp6s18
      server=freedns.afraid.org
      protocol=freedns
      login=myuser
      password=mypassword
      mysubdomain.example.com
      ```

      **(Optional)** To register multiple subdomains, simply list them in the same line, separated by commas and spaces:

      ```conf
      mysubdomain.example.com, mysubdomain2.example.com, mysubdomain3.example.com
      ```

6. Finally, [enable](systemd.md#enable-service) and [restart](systemd.md#restart-service) the `ddclient.service` service.
