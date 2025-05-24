# NetworkManager

## Description

NetworkManager is a program for providing detection and configuration for systems to automatically connect to networks.

## Directory

- [NetworkManager](#networkmanager)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Update Connection by File](#update-connection-by-file)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Set Static IP and Update DNS](#set-static-ip-and-update-dns)
    - [Description](#description-2)
    - [References](#references-2)
    - [Steps](#steps-1)

## References

- [ArchWiki](https://wiki.archlinux.org/title/NetworkManager)

---

## Update Connection by File

### Description

This details how to update a NetworkManager connection using a configuration file.

### References

- [Edit a connection](https://wiki.archlinux.org/title/NetworkManager#Edit_a_connection)
- [nm-settings](https://man.archlinux.org/man/nm-settings.5)

### Steps

1. List down existing NetworkManager connections on the system:

    ```sh
    nmcli connection show
    ```

    Sample output:

    ```
      NAME                UUID                                  TYPE      DEVICE
      Wired connection 1  e7054040-a421-3bef-965d-bb7d60b7cecf  ethernet  enp0s25
      lo                  997f2782-f0fc-301d-bfba-15421a2735d8  loopback  lo
      preconfigured       92a0f7b3-2eba-49ab-a899-24d83978f308  wifi      --
    ```

2. List down any existing NetworkManager connection files on the system:

    ```sh
    sudo ls -l /etc/NetworkManager/system-connections/
    ```

    Sample output:

    ```
      -rw------- 1 root root 306 May 13 08:14 preconfigured.nmconnection
    ```

3. Update the connection file if it exists, otherwise create a new one:

   - Replace `<nmconnection-file>` with the name of the connection file.

      ```sh
      sudo nano "/etc/NetworkManager/system-connections/<nmconnection-file>"
      ```

      For example, if the `<nmconnection-file>` is `preconfigured.nmconnection`:

      ```sh
      sudo nano "/etc/NetworkManager/system-connections/preconfigured.nmconnection"
      ```

   - Make sure to add a unique `id` to the connection file if it is a new connection, or make note of the existing `id`:

      ```ini
      [connection]
      id=<id>
      ```

      In the following example, the `id` is set to `preconfigured`:

      ```ini
      [connection]
      id=preconfigured
      ```

   - Save all changes made to the new or existing connection file.

4. If it is a new connection file, ensure to correct the file's permissions:

    ```sh
    sudo chmod 600 /etc/NetworkManager/system-connections/'<nmconnection-file>'
    ```

    For example, if the `<nmconnection-file>` is `Wired connection 1.nmconnection`:

    ```sh
    sudo chmod 600 /etc/NetworkManager/system-connections/'Wired connection 1.nmconnection'
    ```

5. Refresh NetworkManager's runtime list of profiles and identify any new or updated connections:

    ```sh
    sudo nmcli connection reload
    ```

6. Activate the new or existing connection to apply the changes:

    ```sh
    sudo nmcli connection up '<id>'
    ```

    Replace `<id>` with the `id` of the connection file (i.e. `preconfigured`):

    ```sh
    sudo nmcli connection up 'preconfigured'
    ```

---

## Set Static IP and Update DNS

### Description

This details the process of setting a static IP address and updating the DNS server on a system using NetworkManager.

### References

- [#](#)

### Steps

1. Determine the name of the active network interface on the system (i.e. `eth0`).

2. [Create a new connection file](#update-connection-by-file) (i.e. `static-ip.nmconnection`) with the following configuration:

   - Add the following configuration to the connection file with your own intended values:

      ```ini
      [connection]
      id=<id>
      type=ethernet
      interface-name=<network-interface>

      [ipv4]
      method=manual
      address1=<ip-address>/24,<gateway>
      dns=<dns1>;<dns2>;
      ```

   - The following is an example of the updated configuration:

      ```ini
      [connection]
      id=static-ip
      type=ethernet
      interface-name=eth0

      [ipv4]
      method=manual
      address1=192.168.0.106/24,192.168.0.1
      dns=1.1.1.1;8.8.8.8;
      ```

3. After applying the new connection, verify that it is active:

    ```sh
    sudo nmcli connection show --active
    ```

    Ensure the `id` of the connection you created (i.e. `static-ip`) is present in the output, for example:

    ```
      NAME                UUID                                  TYPE      DEVICE
      static-ip           9c9f60a7-4c2b-3f3a-8e6d-4b0c2c1f5e6a  ethernet  eth0
      lo                  997f2782-f0fc-301d-bfba-15421a2735d8  loopback  lo
    ```
