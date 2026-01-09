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
    - [Post-Install Setup](#post-install-setup)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [References](#references-2)
    - [Recommended Configurations](#recommended-configurations)
  - [Recursive DNS Server](#recursive-dns-server)
    - [Description](#description-3)
    - [References](#references-3)
    - [Unbound Setup](#unbound-setup)
  - [DNS-Over-TLS (DoT)](#dns-over-tls-dot)
    - [Description](#description-4)
    - [References](#references-4)
    - [DoT Setup](#dot-setup)
  - [DNS-Over-HTTPS (DoH)](#dns-over-https-doh)
    - [Description](#description-5)
    - [References](#references-5)
    - [DoH Setup](#doh-setup)
  - [Synchronise Multiple Pi-hole Servers](#synchronise-multiple-pi-hole-servers)
    - [Description](#description-6)
    - [References](#references-6)
    - [Nebula Sync Setup](#nebula-sync-setup)
  - [Usage](#usage)
    - [Description](#description-7)
    - [References](#references-7)
    - [Upstream DNS Server](#upstream-dns-server)
    - [Adding a Group](#adding-a-group)
    - [Adding a Client to a Group](#adding-a-client-to-a-group)
    - [Adding a Domain List](#adding-a-domain-list)
    - [Subscribe to a Domain List](#subscribe-to-a-domain-list)

## References

- [Pi-hole](https://pi-hole.net)
- [Pi-hole documentation](https://docs.pi-hole.net/)
- [Wikipedia](https://en.wikipedia.org/wiki/Pi-hole)

---

## Setup

### Description

This details the process of setting up Pi-hole as the local DNS server for use with your local servers or devices.

### References

- [Prerequisites](https://docs.pi-hole.net/main/prerequisites)
- [Installation](https://docs.pi-hole.net/main/basic-install)
- [Cloudflared, Unbound, or Both?](https://discourse.pi-hole.net/t/cloudflared-unbound-or-both/59245)

### Installation

1. Prepare and [configure](linux.md#configuration) a Debian or Ubuntu system to host the Pi-hole server. This system can be running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or even an [LXC Container](../courses/container.md#create-lxc-container). The following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the system or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule):

     - DNS: `53` (`tcp` and `udp`)
     - FTL: `4711` (`tcp`)
     - HTTP: `80` (`tcp`)
     - **(Optional)** DHCP: `67` (`tcp` and `udp`)
     - **(Optional)** DHCPv6: `546:547` (`udp`)

2. Run the installer:

    ```sh
    curl -sSL https://install.pi-hole.net | bash
    ```

3. In the guided installer wizard, configure the following:

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

4. **(Optional)** If you wish to change the default web interface password, run the following command:

    ```sh
    pihole -a -p
    ```

    Enter the new web interface password you wish to set when prompted.

5. Go through the [post-installation setup steps](#post-install-setup) for a complete Pi-hole server setup.

### Post-Install Setup

This details the post-installation steps of the Pi-hole server for a complete setup:

1. **(Optional)** Enable either [DNS-Over-TLS](#dns-over-tls-dot) or [DNS-Over-HTTPS](#dns-over-https-doh) on the Pi-hole server to prevent DNS requests from being looked or tampered.

2. **(Optional)** Set up the Pi-hole server as a [recursive DNS server](#recursive-dns-server) - skip this step if you are using [DNS-Over-HTTPS](#dns-over-https-doh).

3. **(Optional)** Proceed to configure the Pi-hole server according to the [recommended configuration options](#recommended-configurations).

4. **(Optional)** Create a backup Pi-hole server for redundancy using the same [setup](#setup) process and [synchronise all of your Pi-hole servers](#synchronise-multiple-pi-hole-servers) to ensure consistency.

5. To use the Pi-hole server as the DNS provider for a specific client or network-wide, follow the steps laid out in the [Pi-hole documentation](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245).

---

## Configuration

### Description

This details how to configure a Pi-hole server as well as some recommended configuration options.

### References

- [Is there a way to block internet access to a device using Pi-Hole?](https://www.reddit.com/r/pihole/comments/jqafa4/is_there_a_way_to_block_internet_access_to_a)

### Recommended Configurations

This details some recommended configuration options for a Pi-hole server setup:

1. Block a group of client devices from accessing the internet (i.e. for local IoT devices) through DNS:

   - [Create a dedicated group](#adding-a-group) (i.e. `IoT`) for the client devices.

   - [Create a domain block list](#adding-a-domain-list) using a RegEx filter:

     - Regular Expression: `.*`
     - Group assignment: Select the group(s) that this list should be assigned to (i.e. `IoT`)

   - Finally, [add the client devices](#adding-a-client-to-a-group) to the newly created group (i.e. `IoT`).

---

## Recursive DNS Server

> [!IMPORTANT]  
> If you choose to proceed with this setup, please skip the conflicting [DNS-Over-HTTPS](#dns-over-https-doh) section.

### Description

This details the process of setting up the Pi-hole server as a recursive DNS server solution.

### References

- [Unbound](https://docs.pi-hole.net/guides/dns/unbound)

### Unbound Setup

This details the process of setting up Unbound on the Pi-hole server to set it up as a recursive DNS server:

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

---

## DNS-Over-TLS (DoT)

> [!IMPORTANT]  
> If you choose to proceed with this setup, please skip the conflicting [DNS-Over-HTTPS](#dns-over-https-doh) section.

### Description

This details the process of enabling DNS-Over-TLS on the Pi-hole server which prevents requests from being looked or tampered.

### References

- [Pi-hole + Unbound + DNS Over TLS (Ubiquiti/UniFi/DoT/DoH)](https://youtu.be/8ENLZmzm5vc)

### DoT Setup

This details the process of enabling DNS-Over-TLS on the Pi-hole server using Unbound:

1. As this DoT solution relies on [Unbound](#unbound-setup), ensure you have set it up before proceeding.

2. Verify that you have a certificate installed on the Pi-hole server:

    ```sh
    ls -l /etc/ssl/certs/ca-certificates.crt
    ```

3. Update the `unbound` configuration file to enable DoT:

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

---

## DNS-Over-HTTPS (DoH)

> [!IMPORTANT]  
> If you choose to proceed with this setup, please skip the conflicting [DNS-Over-TLS](#dns-over-tls-dot) section.

### Description

This details the process of enabling DNS-Over-HTTPS on the Pi-hole server which prevents requests from being looked or tampered.

### References

- [cloudflared (DoH)](https://docs.pi-hole.net/guides/dns/cloudflared)

### DoH Setup

This details the process of enabling DNS-Over-HTTPS on the Pi-hole server using Cloudflared:

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

9. Create a [`systemd`](systemd.md) service file for `cloudflared`:

    ```sh
    sudo nano /etc/systemd/system/cloudflared.service
    ```

    Add and save the following content to the service file:

    ```ini
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

---

## Synchronise Multiple Pi-hole Servers

> [!NOTE]  
> This assumes that you have [set up multiple Pi-hole servers](#setup) for redundancy.

### Description

This details the process of synchronising multiple Pi-hole server replicas with a single _primary_ Pi-hole server for consistency.

### References

- [nebula-sync](https://github.com/lovelaze/nebula-sync)
- [Pi-hole Syncing… But Smarter...](https://technotim.live/posts/pihole-sync-nebula)

### Nebula Sync Setup

This details the process of setting up a Nebula Sync server in a containerised environment to synchronise multiple Pi-hole servers:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime).

2. [Deploy the Nebula Sync stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/nebula-sync`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/nebula-sync`): This will be used for the Nebula Sync stack's deployment files.

   - A Compose file for the Nebula Sync stack on the app directory (i.e. `/mnt/smb/docker/nebula-sync/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        nebula-sync:
          container_name: ${APP_CONTAINER}
          image: ghcr.io/lovelaze/nebula-sync:${APP_VERSION}
          env_file:
            - .env
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

   - An env file for the Nebula Sync stack on the app directory (i.e. `/mnt/smb/docker/nebula-sync/.env`):

      ```sh
      # The env variables below this line are specific to the homelab-wiki guide deployment
      ###################################################################################
      SERVICE_NAME=nebula-sync
      APP_CONTAINER=nebula-sync
      APP_VERSION=v0.11.0
      # The env variables below this line were provided by the nebula-sync project
      # You can find documentation for all the supported env variables at https://github.com/lovelaze/nebula-sync#configuration
      ###################################################################################
      PRIMARY=http://192.168.0.106|password
      REPLICAS=http://192.168.0.107|password,http://192.168.0.108|password
      FULL_SYNC=false
      RUN_GRAVITY=false
      CRON=*/15 * * * *
      TZ=Etc/UTC
      CLIENT_SKIP_TLS_VERIFICATION=false

      SYNC_CONFIG_DNS=true
      SYNC_CONFIG_DHCP=false
      SYNC_CONFIG_NTP=false
      SYNC_CONFIG_RESOLVER=true
      SYNC_CONFIG_DATABASE=true
      SYNC_CONFIG_MISC=true
      SYNC_CONFIG_DEBUG=true

      SYNC_GRAVITY_DHCP_LEASES=false
      SYNC_GRAVITY_GROUP=true
      SYNC_GRAVITY_AD_LIST=true
      SYNC_GRAVITY_AD_LIST_BY_GROUP=true
      SYNC_GRAVITY_DOMAIN_LIST=true
      SYNC_GRAVITY_DOMAIN_LIST_BY_GROUP=true
      SYNC_GRAVITY_CLIENT=true
      SYNC_GRAVITY_CLIENT_BY_GROUP=true
      ```

      Replace all of the values in the env file with your own accordingly, bearing in mind the following notes:

      - If you intend to use `https` to specify the address of the primary Pi-hole server and its replica(s), please ensure that you are either using [valid domains you have set up through reverse proxy](../courses/network.md#reverse-proxy) (i.e. `https://pi-hole-1.example.com`), or set the value of `CLIENT_SKIP_TLS_VERIFICATION` to `true` if that is not the case (i.e. `https://192.168.0.106`).
      - Set `FULL_SYNC` to `true` if you wish to synchronise everything between all of your Pi-hole servers, or set it to `false` and define each `SYNC_CONFIG_*` and `SYNC_GRAVITY_*` variables according to your needs.

3. Once the Nebula Sync stack has been deployed, [check the logs](../courses/container.md#container-runtime-usage) of the Nebula Sync container and verify that it is synchronising your Pi-hole servers successfully. Sample log output:

    ```sh
      2025-08-08T10:35:40Z INF Starting nebula-sync v0.11.0
      2025-08-08T10:35:40Z INF Running sync mode=selective replicas=1
      2025-08-08T10:35:40Z INF Authenticating clients...
      2025-08-08T10:35:40Z INF Syncing teleporters...
      2025-08-08T10:35:41Z INF Syncing configs...
      2025-08-08T10:35:41Z INF Invalidating sessions...
      2025-08-08T10:35:41Z INF Sync completed
    ```

---

## Usage

### Description

This details some common usage steps for a Pi-hole server.

### References

- [Pi-hole regular expressions tutorial](https://docs.pi-hole.net/regex/tutorial)

### Upstream DNS Server

This details how to set the upstream DNS provider for the Pi-hole server:

1. From the Pi-hole web interface, click the **Settings** menu item.

2. In the Settings view, navigate to the **DNS** tab.

3. In the **Upstream DNS Servers** section, configure the following:

   - Uncheck all checked boxes of Upstream DNS Provider(s) you wish to not use (i.e. `Cloudflare`)

   - To use any of the listed public DNS providers: simply check their corresponding boxes

   - To use custom DNS servers: Check the custom box(es) you require (i.e. `Custom 1 (IPv4)`) and enter the IP address of the desired DNS server for each box you have checked

   - Click the **Save** button

### Adding a Group

This details how to add a group to the Pi-hole server:

1. From the Pi-hole web interface, under the **GROUP MANAGEMENT** section, click the **Groups** menu item.

2. In the **Group management** view, configure the **Add a new group** form like so:

   - Name: Set a unique, descriptive name for the group (i.e. `Default`)
   - **(Optional)** Comment: Describe briefly the purpose of the group (i.e. `The default group`)

    Click the **Add** button to submit the form and create the group.

3. Under the **List of groups** section, ensure that the newly created group's corresponding **Status** has been toggled to `Enabled`.

### Adding a Client to a Group

This details how to register or name a client known to the network and add it to a group on the Pi-hole server:

1. From the Pi-hole web interface, under the **GROUP MANAGEMENT** section, click the **Clients** menu item.

2. In the **Client group management** view, configure the **Add a new client** form like so:

   - Known clients: Expand the dropdown and select the MAC address corresponding to the client device (i.e. `01:00:5E:90:10:00`)
   - **(Optional)** Comment: Set a unique, descriptive name for the client device (i.e. `my-phone`)
   - Group assignment: Expand the dropdown and select one or more group(s) to assign to the client device (i.e. `Default`)

    Click the **Add** button to submit the form and register the client.

3. To assign an existing client to a group, under the **List of configured clients** section, expand the **Group assignment** dropdown corresponding to the client device and select the group(s) to assign to it, and click the **Apply** button.

### Adding a Domain List

This details how to create a domain allow or block list on the Pi-hole server:

1. From the Pi-hole web interface, under the **GROUP MANAGEMENT** section, click the **Domains** menu item.

2. In the **Domain management** view, configure the **Add a new domain or regex filter** form like so:

   - Choose between **Domain**, to compose your list using exact domain names, or **RegEx filter**, to compose your list using regular expressions

   - Domain:

     - Domain: Add a single or multiple domain name(s) to the list, separated by a whitespace (i.e. `example.com`)
     - Add domain as wildcard: Check the corresponding box to enable the option if you wish to include all subdomains of the domain

   - RegEx filter:

     - Regular Expression: Add a [regular expression](https://docs.pi-hole.net/regex/tutorial) to the list that matches the domain(s) you wish to include in the list (i.e. `^[0-9][^a-z]+\.((com)|(edu))$`)

   - **(Optional)** Comment: Describe briefly the purpose or content of the list (i.e. `Block internet access`)

   - Group assignment: Expand the dropdown and select one or more group(s) to assign to the list (i.e. `Default`)

    Click either the **Add to denied domains** button, to create the domain block list, or the **Add to allowed domains** button, to create the domain allow list.

3. To assign an existing list to a group, under the **List of domains** section, expand the **Group assignment** dropdown corresponding to the list and select the group(s) to assign to it, and click the **Apply** button.

### Subscribe to a Domain List

This details how to subscribe to an external domain list on the Pi-hole server:

1. From the Pi-hole web interface, under the **GROUP MANAGEMENT** section, click the **Lists** menu item.

2. In the **Subscribed lists group management** view, configure the **Add a new subscribed list** form like so:

   - Address: Add the publicly accessible URL to the file containing the list of domains (i.e. `https://example.com/hosts`)
   - **(Optional)** Comment: Describe briefly the purpose or content of the list (i.e. `Adlist`)
   - Group assignment: Expand the dropdown and select one or more group(s) to assign to the list (i.e. `Default`)

    Click either the **Add blocklist** button, to subscribe to the list as a domain block list, or the **Add allowlist** button, to subscribe to the list as a domain allow list.

3. To assign an existing list to a group, under the **Subscribed lists** section, expand the **Group assignment** dropdown corresponding to the list and select the group(s) to assign to it, and click the **Apply** button.
