# Pi-hole

## Description

Pi-hole is a Linux network-level advertisement and Internet tracker blocking application which acts as a DNS sinkhole and optionally a DHCP server, intended for use on a private network.

## Directory

- [Pi-hole](#pi-hole)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Unbound](#unbound)
    - [Cloudflared](#cloudflared)
    - [Usage](#usage)
  - [Upstream DNS Server](#upstream-dns-server)
    - [Description](#description-2)
    - [Steps](#steps)

## References

- [Pi-hole](https://pi-hole.net)
- [Pi-hole documentation](https://docs.pi-hole.net/)
- [Wikipedia](https://en.wikipedia.org/wiki/Pi-hole)

---

## Setup

### Description

This is a brief description of the subtopic.

### References

- [Prerequisites](https://docs.pi-hole.net/main/prerequisites)
- [Installation](https://docs.pi-hole.net/main/basic-install)
- [Unbound](https://docs.pi-hole.net/guides/dns/unbound)
- [cloudflared (DoH)](https://docs.pi-hole.net/guides/dns/cloudflared)
- [Pi-hole + Unbound + DNS Over TLS (Ubiquiti/UniFi/DoT/DoH)](https://youtu.be/8ENLZmzm5vc)
- [Cloudflared, Unbound, or Both?](https://discourse.pi-hole.net/t/cloudflared-unbound-or-both/59245)

### Installation

1. Prepare and [configure](linux.md#configuration) a Debian or Ubuntu system to host the Pi-hole server. This system can be a bare metal device (i.e. Raspberry Pi) or a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template).

2. [Allow](firewall.md#adding-allow-rule) the following ports and their corresponding protocols to be open on the Pi-hole server using the system's firewall (i.e. `ufw`):

   - DNS: `53` (`tcp` and `udp`)
   - FTL: `4711` (`tcp`)
   - HTTP: `80` (`tcp`)
   - **(Optional)** DHCP: `67` (`tcp` and `udp`)
   - **(Optional)** DHCPv6: `546:547` (`udp`)

    As a result, your [firewall status](firewall.md#status) should contain the following rules:

    ```
    53                         ALLOW       Anywhere
    4711/tcp                   ALLOW       Anywhere
    80/tcp                     ALLOW       Anywhere
    67                         ALLOW       Anywhere
    546:547/udp                ALLOW       Anywhere
    53 (v6)                    ALLOW       Anywhere (v6)
    4711/tcp (v6)              ALLOW       Anywhere (v6)
    80/tcp (v6)                ALLOW       Anywhere (v6)
    67 (v6)                    ALLOW       Anywhere (v6)
    546:547/udp (v6)           ALLOW       Anywhere (v6)
    ```

3. Run the installer:

    ```sh
    curl -sSL https://install.pi-hole.net | bash
    ```

4. In the guided installer wizard, configure the following:

   - Static IP Needed: Select the **Continue** option
   - Select Upstream DNS Provider: Select the main DNS provider of your choice (i.e. `Cloudflare`)
   - Blocklists: Select the **Yes** option
   - Install the Admin Web Interface: Select the **Yes** option
   - Install `lighttpd` and the required PHP modules: Select the **Yes** option
   - Enable query logging: Select the **Yes** option
   - Select a privacy mode for FTL: Select the `0 - Show everything` option and select **Continue**

    When prompted with the **Installation Complete!** message, take note of the:

   - Address of the web interface (i.e. `http://192.168.0.106/admin`)
   - Admin Webpage login password

    Select the **OK** option once you are done.

5. **(Optional)** If you wish to change the default web interface password, run the following command:

    ```sh
    pihole -a -p
    ```

    Enter the new web interface password you wish to set when prompted.

6. Using a web browser, log into the Pi-hole web interface with the specified address and password.

### Recursive DNS Server

> [!IMPORTANT]  
> This part of the setup is optional. If you choose to proceed with this setup, please skip the conflicting [DNS-Over-HTTPS](#dns-over-https-doh) section.

This part of the guide details the process of setting up the Pi-hole server as a recursive DNS server solution:

1. [Install](package-manager.md#install-software) the `unbound` package using your package manager (i.e. `apt`).

2. Configure `unbound` server for the Pi-hole server:

    Create the configuration file:

    ```sh
    sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
    ```

    Add and save the following configuration provided by Pi-hole to the file:

    ```yaml
    server:
        # If no logfile is specified, syslog is used
        # logfile: "/var/log/unbound/unbound.log"
        verbosity: 0

        interface: 127.0.0.1
        port: 5335
        do-ip4: yes
        do-udp: yes
        do-tcp: yes

        # May be set to yes if you have IPv6 connectivity
        do-ip6: no

        # You want to leave this to no unless you have *native* IPv6. With 6to4 and
        # Terredo tunnels your web browser should favor IPv4 for the same reasons
        prefer-ip6: no

        # Use this only when you downloaded the list of primary root servers!
        # If you use the default dns-root-data package, unbound will find it automatically
        #root-hints: "/var/lib/unbound/root.hints"

        # Trust glue only if it is within the server's authority
        harden-glue: yes

        # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
        harden-dnssec-stripped: yes

        # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
        # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
        use-caps-for-id: no

        # Reduce EDNS reassembly buffer size.
        # IP fragmentation is unreliable on the Internet today, and can cause
        # transmission failures when large DNS messages are sent via UDP. Even
        # when fragmentation does work, it may not be secure; it is theoretically
        # possible to spoof parts of a fragmented DNS message, without easy
        # detection at the receiving end. Recently, there was an excellent study
        # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
        # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
        # in collaboration with NLnet Labs explored DNS using real world data from the
        # the RIPE Atlas probes and the researchers suggested different values for
        # IPv4 and IPv6 and in different scenarios. They advise that servers should
        # be configured to limit DNS messages sent over UDP to a size that will not
        # trigger fragmentation on typical network links. DNS servers can switch
        # from UDP to TCP when a DNS response is too big to fit in this limited
        # buffer size. This value has also been suggested in DNS Flag Day 2020.
        edns-buffer-size: 1232

        # Perform prefetching of close to expired message cache entries
        # This only applies to domains that have been frequently queried
        prefetch: yes

        # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
        num-threads: 1

        # Ensure kernel buffer is large enough to not lose messages in traffic spikes
        so-rcvbuf: 1m

        # Ensure privacy of local IP ranges
        private-address: 192.168.0.0/16
        private-address: 169.254.0.0/16
        private-address: 172.16.0.0/12
        private-address: 10.0.0.0/8
        private-address: fd00::/8
        private-address: fe80::/10
    ```

3. [Restart](systemd.md#restart-service) the `unbound.service` service.

4. Test to make sure the local recursive server is working:

    ```sh
    dig @127.0.0.1 -p 5335 pi-hole.net
    ```

    Sample successful output:

    ```
    ; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> pi-hole.net @127.0.0.1 -p 5335
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20505
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;pi-hole.net.                   IN      A

    ;; ANSWER SECTION:
    pi-hole.net.            300     IN      A       3.18.136.52

    ;; Query time: 472 msec
    ;; SERVER: 127.0.0.1#5335(127.0.0.1) (UDP)
    ;; WHEN: Thu Oct 17 11:21:52 +08 2024
    ;; MSG SIZE  rcvd: 56
    ```

5. Create a configuration file to set the maximum size for DNS packets when using EDNS:

    ```sh
    echo "edns-packet-max=1232" | sudo tee -a /etc/dnsmasq.d/99-edns.conf
    ```

    This ensures that `dnsmasq` uses a maximum DNS packet size of 1232 bytes to avoid issues with DNS packet fragmentation and can improve DNS query reliability.

6. This might not be needed depending on your server but for good measure, disable `resolveconf.conf` entry for `unbound`:

   - [Check the status](systemd.md#service-status) of the `unbound-resolvconf.service` service.
   - [Stop and disable](systemd.md#disable-service) the `unbound-resolvconf.service` service.
   - [Restart](systemd.md#restart-service) the `unbound.service` service.

7. [Configure the Upstream DNS Provider](#upstream-dns-server) for the Pi-hole server by checking the **Custom 1 (IPv4)** box and entering the following IP address: `127.0.0.1#5335`.

### DNS-Over-TLS (DoT)

> [!IMPORTANT]  
> This part of the setup is optional. If you choose to proceed with this setup, please skip the conflicting [DNS-Over-HTTPS](#dns-over-https-doh) section.

This part of the guide details the process of enabling DNS-Over-TLS on the Pi-hole server which prevents requests from being looked or tampered:

1. As this DoT solution relies on [Unbound](#recursive-dns-server), ensure you have set it up before proceeding.

2. Verify that you have a certificate installed on the Pi-hole server:

    ```sh
    ls -la /etc/ssl/certs/ca-certificates.crt
    ```

3. Update the `unbound` configuration file:

    ```sh
    sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
    ```

   - Make the following additions to the end of the file and save the changes:

      ```diff
            # Ensure privacy of local IP ranges
            private-address: 192.168.0.0/16
            private-address: 169.254.0.0/16
            private-address: 172.16.0.0/12
            private-address: 10.0.0.0/8
            private-address: fd00::/8
            private-address: fe80::/10
      +
      +     # Define trusted certificates
      +     tls-cert-bundle: /etc/ssl/certs/ca-certificates.crt
      +
      + forward-zone:
      +     name: "."
      +     forward-tls-upstream: yes
      +     # Cloudflare DNS servers as an example
      +     forward-addr: 1.1.1.1@853
      +     forward-addr: 1.0.0.1@853
      +     # You can add other DNS servers supporting DNS over TLS, like Google DNS or Quad9:
      +     # Google:
      +     # forward-addr: 8.8.8.8@853
      +     # forward-addr: 8.8.4.4@853
      +     # Quad9:
      +     # forward-addr: 9.9.9.9@853
      +     # forward-addr: 149.112.112.112@853
      ```

   - Sample of the updated `unbound` configuration file:

      ```yaml
      server:
          # If no logfile is specified, syslog is used
          # logfile: "/var/log/unbound/unbound.log"
          verbosity: 0

          interface: 127.0.0.1
          port: 5335
          do-ip4: yes
          do-udp: yes
          do-tcp: yes

          # May be set to yes if you have IPv6 connectivity
          do-ip6: no

          # You want to leave this to no unless you have *native* IPv6. With 6to4 and
          # Terredo tunnels your web browser should favor IPv4 for the same reasons
          prefer-ip6: no

          # Use this only when you downloaded the list of primary root servers!
          # If you use the default dns-root-data package, unbound will find it automatically
          #root-hints: "/var/lib/unbound/root.hints"

          # Trust glue only if it is within the server's authority
          harden-glue: yes

          # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
          harden-dnssec-stripped: yes

          # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
          # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
          use-caps-for-id: no

          # Reduce EDNS reassembly buffer size.
          # IP fragmentation is unreliable on the Internet today, and can cause
          # transmission failures when large DNS messages are sent via UDP. Even
          # when fragmentation does work, it may not be secure; it is theoretically
          # possible to spoof parts of a fragmented DNS message, without easy
          # detection at the receiving end. Recently, there was an excellent study
          # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
          # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
          # in collaboration with NLnet Labs explored DNS using real world data from the
          # the RIPE Atlas probes and the researchers suggested different values for
          # IPv4 and IPv6 and in different scenarios. They advise that servers should
          # be configured to limit DNS messages sent over UDP to a size that will not
          # trigger fragmentation on typical network links. DNS servers can switch
          # from UDP to TCP when a DNS response is too big to fit in this limited
          # buffer size. This value has also been suggested in DNS Flag Day 2020.
          edns-buffer-size: 1232

          # Perform prefetching of close to expired message cache entries
          # This only applies to domains that have been frequently queried
          prefetch: yes

          # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
          num-threads: 1

          # Ensure kernel buffer is large enough to not lose messages in traffic spikes
          so-rcvbuf: 1m

          # Ensure privacy of local IP ranges
          private-address: 192.168.0.0/16
          private-address: 169.254.0.0/16
          private-address: 172.16.0.0/12
          private-address: 10.0.0.0/8
          private-address: fd00::/8
          private-address: fe80::/10

          # Define trusted certificates
          tls-cert-bundle: /etc/ssl/certs/ca-certificates.crt

      forward-zone:
          name: "."
          forward-tls-upstream: yes
          # Cloudflare DNS servers as an example
          forward-addr: 1.1.1.1@853
          forward-addr: 1.0.0.1@853
          # You can add other DNS servers supporting DNS over TLS, like Google DNS or Quad9:
          # Google:
          # forward-addr: 8.8.8.8@853
          # forward-addr: 8.8.4.4@853
          # Quad9:
          # forward-addr: 9.9.9.9@853
          # forward-addr: 149.112.112.112@853
      ```

4. [Restart](systemd.md#restart-service) the `unbound.service` service.

### DNS-Over-HTTPS (DoH)

> [!IMPORTANT]  
> This part of the setup is optional. If you choose to proceed with this setup, please skip the conflicting [DNS-Over-TLS](#dns-over-tls-dot) section.

This part of the guide details the process of enabling DNS-Over-HTTPS on the Pi-hole server which prevents requests from being looked or tampered:

1. Determine the CPU architecture of the Pi-hole server:

    ```sh
    dpkg --print-architecture
    ```

    **Alternatively**, use the following command instead if your server does not have `dpkg` installed:

    ```sh
    uname -m
    ```

2. Download the latest `cloudflared` binary to the home directory:

    ```sh
    curl -Lo ~/cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-<architecture>
    ```

    Replace `<architecture>` with the CPU architecture of the Pi-hole server (i.e. `amd64` or `arm64`).

3. Make the `cloudflared` binary executable:

    ```sh
    chmod +x ~/cloudflared
    ```

4. Move the `cloudflared` binary to your path (i.e. `/usr/local/bin`):

    ```sh
    sudo mv ~/cloudflared /usr/local/bin/cloudflared
    ```

5. Verify that the `cloudflared` binary has been installed to your path:

    ```sh
    cloudflared -v
    ```

    Sample output showing that it has been installed successfully:

    ```
    cloudflared version 2024.10.0 (built 2024-10-10-0949 UTC)
    ```

6. Create a service user for running `cloudflared` on startup:

    ```sh
    sudo useradd -s /usr/sbin/nologin -r -M cloudflared
    ```

7. Create a configuration file for `cloudflared`:

    ```sh
   sudo nano /etc/default/cloudflared
    ```

    Add and save the following configuration provided by Pi-hole to the file:

    ```sh
    # Commandline args for cloudflared, using Cloudflare DNS
    CLOUDFLARED_OPTS=--port 5053 --upstream https://1.1.1.1/dns-query --upstream https://1.0.0.1/dns-query
    ```

8. Update the permissions of both the binary and configuration file for the Cloudflare service user:

    ```sh
    sudo chown cloudflared: /usr/local/bin/cloudflared /etc/default/cloudflared
    ```

9. Create a [systemd](systemd.md) service file for `cloudflared`:

    ```sh
    sudo nano /etc/systemd/system/cloudflared.service
    ```

    Add and save the following content to the service file:

    ```sh
    [Unit]
    Description=cloudflared DNS over HTTPS proxy
    After=syslog.target network-online.target

    [Service]
    Type=simple
    User=cloudflared
    EnvironmentFile=/etc/default/cloudflared
    ExecStart=/usr/local/bin/cloudflared proxy-dns $CLOUDFLARED_OPTS
    Restart=on-failure
    RestartSec=10
    KillMode=process

    [Install]
    WantedBy=multi-user.target
    ```

10. [Start and enable](systemd.md#enable-service) the `cloudflared.service` service.

11. Verify that DNS-Over-HTTPS is working on the Pi-hole server:

    ```sh
    dig @127.0.0.1 -p 5053 google.com
    ```

    Sample successful output:

    ```
    ; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> @127.0.0.1 -p 5053 google.com
    ; (1 server found)
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56103
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: opjt6abhpa2tlwne (echoed)
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             300     IN      A       74.125.130.138
    google.com.             300     IN      A       74.125.130.101
    google.com.             300     IN      A       74.125.130.113
    google.com.             300     IN      A       74.125.130.100
    google.com.             300     IN      A       74.125.130.139
    google.com.             300     IN      A       74.125.130.102

    ;; Query time: 44 msec
    ;; SERVER: 127.0.0.1#5053(127.0.0.1) (UDP)
    ;; WHEN: Fri Oct 18 01:20:11 +08 2024
    ;; MSG SIZE  rcvd: 207
    ```

12. [Configure the Upstream DNS Provider](#upstream-dns-server) for the Pi-hole server by checking the **Custom 1 (IPv4)** box and entering the following IP address: `127.0.0.1#5053`.

13. **(Optional)** Refer to the Pi-hole documentation on how to [update the `cloudflared` tool](https://docs.pi-hole.net/guides/dns/cloudflared/#updating-cloudflared) in the future.

### Usage

1. To use the Pi-hole server as the DNS provider for a specific client or network-wide, follow the steps laid out in the [Pi-hole documentation](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245).

---

## Upstream DNS Server

### Description

This details how to set the upstream DNS provider for the Pi-hole server.

### Steps

1. From the Pi-hole web interface, click the **Settings** menu item.

2. In the Settings view, navigate to the **DNS** tab.

3. In the **Upstream DNS Servers** section, configure the following:

   - Uncheck all checked boxes of Upstream DNS Provider(s) you wish to not use (i.e. `Cloudflare`)

   - To use any of the listed public DNS providers: simply check their corresponding boxes

   - To use custom DNS servers: Check the custom box(es) you require (i.e. `Custom 1 (IPv4)`) and enter the IP address of the desired DNS server for each box you have checked

   - Click the **Save** button
