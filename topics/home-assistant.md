# Home Assistant

## Description

Home Assistant is free and open-source software used for home automation. It serves as an integration platform and smart home hub, allowing users to control smart home devices.

## Directory

- [Home Assistant](#home-assistant)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Install HA as a Docker Container](#install-ha-as-a-docker-container)
    - [Configuration](#configuration)
  - [Zigbee2MQTT](#zigbee2mqtt)
    - [Description](#description-2)
    - [References](#references-2)
    - [Install Z2M as a Docker Container](#install-z2m-as-a-docker-container)
    - [Z2M Post-Install Setup](#z2m-post-install-setup)
    - [Integrating Z2M with Home Assistant](#integrating-z2m-with-home-assistant)
  - [VS Code Server](#vs-code-server)
    - [Description](#description-3)
    - [References](#references-3)
    - [Install Code-server as a Docker Container](#install-code-server-as-a-docker-container)
    - [Integrating Code-server with Home Assistant](#integrating-code-server-with-home-assistant)
  - [HACS](#hacs)
    - [Description](#description-4)
    - [References](#references-4)
    - [Install HACS on Home Assistant Container](#install-hacs-on-home-assistant-container)
  - [Usage](#usage)
    - [Description](#description-5)
    - [References](#references-5)
    - [Configuring Home Assistant](#configuring-home-assistant)
    - [Adding an Integration to Home Assistant](#adding-an-integration-to-home-assistant)
    - [Adding a Dashboard to Home Assistant](#adding-a-dashboard-to-home-assistant)

## References

- [Home Assistant](https://www.home-assistant.io)
- [Wikipedia](https://en.wikipedia.org/wiki/Home_Assistant)

---

## Setup

### Description

This details the installation and setup process of a Home Assistant (HA) server.

### References

- [JimsGarage/Home-Assistant](https://github.com/JamesTurland/JimsGarage/tree/main/Home-Assistant)
- [Reverse proxy using NGINX](https://community.home-assistant.io/t/reverse-proxy-using-nginx/196954)
- [HTTP](https://www.home-assistant.io/integrations/http)
- [Switch Home Assistant to Use PostgreSQL Instead of SQLite](https://unixorn.github.io/post/hass-using-postgresql-instead-of-sqlite)
- [Let's Build A Smart Home with Home Assistant](https://youtu.be/6z-ilfbzDlY)

### Install HA as a Docker Container

This details the installation process of HA using the Container installation method:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#linux-containers-lxc); ensure that [Docker is installed and set up](../courses/container.md#setting-up-docker). For simplicity, the following considerations should be noted:

   - Ports that will be used and requires allowing on the host machine vary depending on the (intended) smart home setup, hence, it is recommended not to enable or set up a [firewall](firewall.md) on the system.

2. [Deploy the HA stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/home-assistant`): This will be used for at least the database volume directory which specifically requires local storage.

   - **(Optional)** A remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/home-assistant`): This can be used for anything else that supports remote storage such as the Home Assistant configuration volume directory.

   - A Docker compose file for the Home Assistant stack on either app directory (i.e. `/mnt/smb/docker/home-assistant/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        home-assistant:
          container_name: ${HA_CONTAINER}
          image: ghcr.io/home-assistant/home-assistant:${HA_VERSION}
          ports:
            - 8123:8123
          environment:
            - TZ=${APP_TIMEZONE}
            - DB_URL=postgresql://${DB_USER}:${DB_PASSWORD}@${HA_DB_CONTAINER}/${DB_NAME}
          volumes:
            - ${REMOTE_APP_DIR}/config:/config
            - /etc/localtime:/etc/localtime:ro
            - /run/dbus:/run/dbus:ro
          depends_on:
            - home-assistant-db
          networks:
            - frontend
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

        home-assistant-db:
          container_name: ${HA_DB_CONTAINER}
          image: docker.io/postgres:${POSTGRES_VERSION}
          environment:
            - POSTGRES_USER=${DB_USER}
            - POSTGRES_PASSWORD=${DB_PASSWORD}
            - POSTGRES_DB=${DB_NAME}
            # ensure the database gets created correctly
            # https://github.com/matrix-org/synapse/blob/master/docs/postgres.md#set-up-database
            - POSTGRES_INITDB_ARGS=--encoding=UTF-8
          volumes:
            - ${LOCAL_APP_DIR}/pgdata:/var/lib/postgresql/data
          networks:
            - frontend
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        frontend:
      ```

      If you are not using a remote app directory, replace `${REMOTE_APP_DIR}` with `${LOCAL_APP_DIR}` accordingly.

   - An env file for the Home Assistant stack on either of the app directory (i.e. `/mnt/smb/docker/home-assistant/.env`):

      ```sh
      SERVICE_NAME=home-assistant
      HA_CONTAINER=home-assistant
      HA_DB_CONTAINER=home-assistant-db
      HA_VERSION=2025.6.3
      APP_TIMEZONE=Asia/Kuala_Lumpur
      REMOTE_APP_DIR=/mnt/smb/docker/home-assistant
      POSTGRES_VERSION=16.3-bookworm
      DB_USER=postgres
      DB_PASSWORD=secret
      DB_NAME=home-assistant-db
      LOCAL_APP_DIR=/home/myuser/.local/share/docker/home-assistant
      ```

      Omit any variable(s) you may not need (i.e. `REMOTE_APP_DIR`) and replace the rest of the values with your own accordingly.

   - Either before deploying, of which you will need to pre-create the directory (i.e. `/mnt/smb/docker/home-assistant/config`), or after deployment - update the Home Assistant configuration based on the [recommended configuration options](#configuration).

3. **(Optional)** On the same host system where you have deployed your HA stack, [deploy and set up a VS Code Server (Code-server) stack](#install-code-server-as-a-docker-container) so it could be used to configure the HA configuration files from within the HA server.

4. **(Optional)** [Install and set up the HACS integration](#install-hacs-on-home-assistant-container) on the HA server as a means to install custom elements and integrations on the server, graphically.

5. **(Optional)** On the same host system where you have deployed your HA stack, [deploy and set up a Zigbee2MQTT (Z2M) stack](#install-z2m-as-a-docker-container) to manage and use Zigbee devices on the HA server.

### Configuration

This details some recommended configuration options for a HA setup:

1. [Configure HA](#configuring-home-assistant) to use PostgreSQL instead of the default SQLite database:

   - Based on the `DB_URL` environment variable set in the Docker compose file, add the following variable in your HA instance's secrets file (`secrets.yaml`):

      ```yaml
      db_conn_string: !env_var DB_URL
      ```

   - Add the following configuration option to your HA instance's configuration file (`configuration.yaml`):

      ```yaml
      recorder:
        db_url: !secret db_conn_string
      ```

2. **(Optional)** [Configure HA](#configuring-home-assistant) to allow for reverse proxy if you wish to [set up a domain](../courses/network.md#registering-subdomains) for the HA instance:

   - Add the following configuration option to your HA instance's configuration file (`configuration.yaml`):

      ```yaml
      # Reference: https://www.home-assistant.io/integrations/http
      http:
        # For extra security set this to only accept connections on localhost if NGINX is on the same machine
        # Uncommenting this will mean that you can only reach Home Assistant using the proxy, not directly via IP from other clients.
        # server_host: 127.0.0.1
        use_x_forwarded_for: true
        # You must set the trusted proxy IP address so that Home Assistant will properly accept connections
        # Set this to your NGINX machine IP, or localhost if hosted on the same machine.
        trusted_proxies:
          - <local-network>
        # Flag indicating whether additional IP filtering is enabled.
        ip_ban_enabled: true
        # Number of failed login attempts from a single IP after which it will be automatically banned if ip_ban_enabled is true.
        login_attempts_threshold: 5
      ```

      Replace `<local-network>` with your local network range (i.e. `192.168.0.0/24`).

---

## Zigbee2MQTT

### Description

This details setting up Zigbee2MQTT (Z2M) with Home Assistant (HA).

### References

- [How to setup zigbee2mqtt with home assistant!](https://youtu.be/C71SpthNkmU)
- [Mosquitto MQTT Broker - Explanation and Setup](https://youtu.be/2S_kZo_ElxY)
- [The Home Automation Must Buy - SLZB-06M & Zigbee2MQTT](https://youtu.be/-KOqaPfYPYw)
- [Basic configuration](https://www.zigbee2mqtt.io/guide/configuration/adapter-settings.html)
- [Supported Adapters](https://www.zigbee2mqtt.io/guide/adapters)
- [Docker Compose](https://www.zigbee2mqtt.io/guide/installation/02_docker.html#docker-compose)
- [JimsGarage/Zigbee2MQTT](https://github.com/JamesTurland/JimsGarage/blob/main/Zigbee2MQTT)

### Install Z2M as a Docker Container

> [!IMPORTANT]  
> This guide assumes that you wish to incorporate Z2M into your smart home setup and that you have a hardware device capable of Z2M (i.e. [SONOFF ZBDongle-P](https://sonoff.tech/products/sonoff-zigbee-3-0-usb-dongle-plus-zbdongle-p)). If said device requires a firmware update, make sure to do so before proceeding.

> [!NOTE]  
> This guide assumes that you have [deployed and set up HA](#setup) as a Docker container.

This details the installation process of Z2M and Mosquitto containers as companion to the HA server:

1. First and foremost, ensure that the host machine has read-write access to the Z2M-capable device either through physical attachment (i.e. bare metal host machine) or [hardware passthrough](../courses/hypervisor.md#hardware-passthrough) (i.e. containerised or virtualised host machine).

2. Identify the name of the Docker network (i.e. `frontend`) of your HA installation:

    ```sh
    docker network ls --format "{{.Name}}" | grep -i frontend
    ```

    Sample output:

    ```
      home-assistant_frontend
    ```

    Take note of this name (i.e. `home-assistant_frontend`) as we will use it to join the network on the Z2M stack.

3. Locate and identify the Z2M-capable device paths on the host system:

   - Run the following command and locate the Z2M-capable device:

      ```sh
      ls -l /dev/serial/by-id/
      ```

      Sample output:

      ```
        total 0
        lrwxrwxrwx 1 root root 13 Jul  3 14:00 usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0 -> ../../ttyUSB0
      ```

   - Extract the following values based on the value of the last column corresponding to the device (i.e. `usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0 -> ../../ttyUSB0`):

     - **DEVICE_BY_ID_PATH**: The device's symlink path which leads with `/dev/serial/by-id/` and trails with the device identifier (i.e. `usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0`)

        ```
          /dev/serial/by-id/<device-identifier>
        ```

        Sample value:

        ```
          /dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0
        ```

     - **DEVICE_TTY_PATH**: The physical device path that the symlink (i.e. `/dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0`) points to

        ```sh
        readlink -f <device-by-id-path>
        ```

        For example:

        ```sh
        readlink -f /dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0
        ```

        Sample value:

        ```
          /dev/ttyUSB0
        ```

     - Take note of both the `DEVICE_BY_ID_PATH` (i.e. `/dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0`) and `DEVICE_TTY_PATH` (i.e. `/dev/ttyUSB0`) values which will later be used to passthrough the Z2M-capable device to the Z2M container.

4. [Deploy the Z2M stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/z2m`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/z2m`): This will be used for the Z2M stack's volumes.

   - A Docker compose file for the Z2M stack on the app directory (i.e. `/mnt/smb/docker/z2m/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        mosquitto:
          container_name: ${MQTT_CONTAINER}
          image: docker.io/eclipse-mosquitto:${MQTT_VERSION}
          ports:
            - 1883:1883
            - 9001:9001
          deploy:
            resources:
              limits:
                memory: 256M
          volumes:
            - ${APP_DIR}/mosquitto/config:/mosquitto/config
            - ${APP_DIR}/mosquitto/data:/mosquitto/data
            - ${APP_DIR}/mosquitto/log:/mosquitto/log
          networks:
            - ha-network
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

        zigbee2mqtt:
          container_name: ${ZIGBEE2MQTT_CONTAINER}
          image: ghcr.io/koenkk/zigbee2mqtt:${ZIGBEE2MQTT_VERSION}
          ports:
            - 8080:8080 # frontend port
          # group_add:
          #   - dialout
          environment:
            - TZ=${APP_TIMEZONE}
          volumes:
            - ${APP_DIR}/zigbee2mqtt/data:/app/data # persistent data storage
            - /run/udev:/run/udev:ro
          depends_on:
            - mosquitto
          devices:
            # Make sure this matches your adapter location
            - ${DEVICE_BY_ID_PATH}:${DEVICE_TTY_PATH}
          networks:
            - ha-network
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        ha-network:
          name: ${HA_NETWORK}
          external: true
      ```

   - An env file for the Z2M stack on the app directory (i.e. `/mnt/smb/docker/z2m/.env`):

      ```sh
      SERVICE_NAME=z2m
      MQTT_CONTAINER=mosquitto
      ZIGBEE2MQTT_CONTAINER=zigbee2mqtt
      MQTT_VERSION=2.0.21-openssl
      APP_DIR=/mnt/smb/docker/z2m
      HA_NETWORK=home-assistant_frontend
      ZIGBEE2MQTT_VERSION=2.4.0
      APP_TIMEZONE=Asia/Kuala_Lumpur
      DEVICE_BY_ID_PATH=/dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_9020107cf473ef11852cdb1e313510fd-if00-port0
      DEVICE_TTY_PATH=/dev/ttyUSB0
      ```

      Replace the values with your own accordingly.

   - Pre-configure the Mosquitto server for the initial configuration before deployment:

     - Pre-create the Mosquitto container's config volume directory inside the app directory (i.e. `/mnt/smb/docker/z2m`):

        ```sh
        mkdir -p <app-dir>/mosquitto/config
        ```

        For example:

        ```sh
        mkdir -p /mnt/smb/docker/z2m/mosquitto/config
        ```

     - Inside the Mosquitto container's config volume directory (i.e. `/mnt/smb/docker/z2m/mosquitto/config`), create the configuration file (i.e. `mosquitto.conf`):

        ```sh
        nano <mosquitto-config-dir>/mosquitto.conf
        ```

        For example:

        ```sh
        nano /mnt/smb/docker/z2m/mosquitto/config/mosquitto.conf
        ```

        Add and save the following configuration to the file:

        ```sh
        #allow_anonymous false
        listener 1883
        listener 9001
        protocol websockets
        persistence true
        #password_file /mosquitto/config/pwfile
        persistence_file mosquitto.db
        persistence_location /mosquitto/data
        ```

5. Follow the [post-installation steps](#z2m-post-install-setup) to complete the Z2M stack setup.

### Z2M Post-Install Setup

This details the post-installation steps of the Z2M stack for a complete setup:

1. Generate the login credentials for the Mosquitto container and lock it behind authentication:

   - [Enter the Mosquitto container](../courses/container.md#docker-usage) using the compose file (i.e. `/mnt/smb/docker/z2m/docker-compose.yml`), the `mosquitto` service, and the `sh` shell.

   - Inside the container, create the credentials file (i.e. `/mosquitto/config/pwfile`) for a brand new user (i.e. `mqttuser`):

      ```sh
      mosquitto_passwd -c /mosquitto/config/pwfile <user>
      ```

      For example:

      ```sh
      mosquitto_passwd -c /mosquitto/config/pwfile mqttuser
      ```

      When prompted to enter a password, enter your secure password accordingly.

   - Update the earlier pre-configured configuration file from inside the container (i.e. `/mosquitto/config/mosquitto.conf`):

      ```sh
      vi /mosquitto/config/mosquitto.conf
      ```

      Uncomment the following commented lines to lock the Mosquitto container behind authentication:

      ```diff
      - #allow_anonymous false
      + allow_anonymous false
        listener 1883
        listener 9001
        protocol websockets
        persistence true
      - #password_file /mosquitto/config/pwfile
      + password_file /mosquitto/config/pwfile
        persistence_file mosquitto.db
        persistence_location /mosquitto/data
      ```

      Sample configuration:

      ```sh
      allow_anonymous false
      listener 1883
      listener 9001
      protocol websockets
      persistence true
      password_file /mosquitto/config/pwfile
      persistence_file mosquitto.db
      persistence_location /mosquitto/data
      ```

   - Exit the Mosquitto container, and [restart](../courses/container.md#docker-usage) the `mosquitto` container service to apply the changes.

2. Complete the Z2M server onboarding process and hook the Mosquitto server to it as a broker:

   - Launch the Z2M server web interface at `http://<z2m-server-host>:8080` (i.e. `http://192.168.0.106:8080`) on a web browser.

   - In the **Zigbee2MQTT Onboarding** form, configure the following settings according to your setup:

     - Found Devices: Expand the dropdown and select the Z2M-capable device you have attached and wish to use as the coordinator (i.e. `/dev/ttyUSB0`)
     - Coordinator/Adapter Type/Stack/Driver: Expand the dropdown and select the appropriate driver for your device (i.e. `ember`)
     - MQTT Base Topic: Set a unique name as the base topic or leave as default (i.e. `zigbee2mqtt`)
     - MQTT Server: Set the value to the Mosquitto container's endpoint (i.e. `mqtt://mosquitto:1883`)
     - MQTT User: Add the dedicated user you have created in the Mosquitto container (i.e. `mqttuser`)
     - MQTT Password: Add the secure password you have set for the Mosquitto container user
     - Frontend enabled?: Check the corresponding box to enable the Mosquitto server graphical frontend
     - **(Optional)** Home Assistant enabled?: Check the corresponding box to allow for [HA integration](#integrating-z2m-with-home-assistant)

      Click the **Submit** button to apply the configuration.

   - Once the configuration form has been submitted, wait for the page to reload and take you to the actual web interface - if it instead takes you to the same onboarding form, go through the configuration form again and check for any missing or inaccurate values.

3. **(Optional)** [Add the MQTT integration to your HA server](#integrating-z2m-with-home-assistant) so that compatible Zigbee devices that have been added to the Z2M server could be used from within HA.

### Integrating Z2M with Home Assistant

This details how to add the MQTT integration to the HA server:

1. [Integrate Z2M to your HA server](#adding-an-integration-to-home-assistant), while noting the following options:

   - Select brand: `MQTT`

   - What do you want to add?: `MQTT`

   - Connection information of your MQTT broker:

     - Broker: Set the Mosquitto server host (i.e. `mosquitto`)
     - Port: Set the Mosquitto server port number (i.e. `1883`)
     - Username: Add the Mosquitto server username (i.e. `mqttuser`)
     - Password: Add the secure password of the Mosquitto server user

   - Device created:

     - Device name: Set a descriptive name for the Z2M coordinator or leave as default (i.e. `Zigbee2MQTT Bridge`)
     - Area: Expand the dropdown and select the area where the device is located or leave it empty

2. [Incorporate Z2M to your HA server as a dashboard](#adding-a-dashboard-to-home-assistant), while noting the following options:

   - Add dashboard: `Webpage`

   - URL: Set the URL of the VS Code server to `http://<z2m-server-host>:8080` (i.e. `http://192.168.0.106:8080`)

   - Dashboard information:

     - Title: Set a descriptive and unique title for the dashboard (i.e. `Zigbee2MQTT`)
     - Icon: Expand the dropdown and select an appropriate icon to represent the dashboard (i.e. `mdi:zigbee`)
     - Admin only: Check the corresponding switch to enable restricting access to only the admin users
     - Show in sidebar: Check the corresponding switch to enable showing the dashboard menu in the sidebar

3. Once the Z2M stack has been integrated with the HA server through this integration, Zigbee devices added to the Z2M server (i.e. through the added dashboard) will automatically appear and be available for use on the HA server.

---

## VS Code Server

### Description

This details setting up VS Code Server (Code-server) with Home Assistant (HA).

### References

- [User / Group Identifiers](https://docs.linuxserver.io/images/docker-code-server/#user-group-identifiers)
- [Installing Home Assistant using Docker Compose](https://pimylifeup.com/home-assistant-docker-compose)

### Install Code-server as a Docker Container

> [!NOTE]  
> This guide assumes that you have [deployed and set up HA](#setup) as a Docker container.

This details the installation process of a Code-server container as companion to the HA server:

1. Identify the name of the Docker network (i.e. `frontend`) of your HA installation:

    ```sh
    docker network ls --format "{{.Name}}" | grep -i frontend
    ```

    Sample output:

    ```
      home-assistant_frontend
    ```

    Take note of this name (i.e. `home-assistant_frontend`) as we will use it to join the network on the Code-server stack.

2. Pertaining to the HA server's config directory, identify the following items:

   - The full path to the config directory on the host system (i.e. `/mnt/smb/docker/home-assistant/config`)

   - The full path to the config directory on the container (i.e. `/config`)

   - The `UID` of the user who owns the config directory on the host system (i.e. `1000`):

      ```sh
      stat -c '%u' <ha-host-config-dir>
      ```

      For example:

      ```sh
      stat -c '%u' /mnt/smb/docker/home-assistant/config
      ```

      Sample output:

      ```
        1000
      ```

   - The `GID` of the user who owns the config directory on the host system (i.e. `1000`):

      ```sh
      stat -c '%g' <ha-host-config-dir>
      ```

      For example:

      ```sh
      stat -c '%g' /mnt/smb/docker/home-assistant/config
      ```

      Sample output:

      ```
        1000
      ```

3. [Deploy the Code-server stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/code-server`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/code-server`): Since the Code-server stack does not have its own volume, this will only be used for its Docker compose and env file.

   - A Docker compose file for the Code-server stack on the app directory (i.e. `/mnt/smb/docker/code-server/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        code-server:
          container_name: ${VSCODE_CONTAINER}
          image: ghcr.io/linuxserver/code-server:${VSCODE_VERSION}
          ports:
            - 8443:8443
          environment:
            - TZ=${APP_TIMEZONE}
            - DEFAULT_WORKSPACE=${HA_CONTAINER_CONFIG_DIR}
            - PUID=${HA_HOST_CONFIG_DIR_UID}
            - PGID=${HA_HOST_CONFIG_DIR_GID}
          volumes:
            - ${HA_HOST_CONFIG_DIR}:${HA_CONTAINER_CONFIG_DIR}
          networks:
            - ha-network
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        ha-network:
          name: ${HA_NETWORK}
          external: true
      ```

   - An env file for the Code-server stack on the app directory (i.e. `/mnt/smb/docker/code-server/.env`):

      ```sh
      SERVICE_NAME=code-server
      VSCODE_CONTAINER=code-server
      VSCODE_VERSION=4.101.2-ls283
      APP_TIMEZONE=Asia/Kuala_Lumpur
      HA_CONTAINER_CONFIG_DIR=/config
      HA_HOST_CONFIG_DIR_UID=1000
      HA_HOST_CONFIG_DIR_GID=1000
      HA_HOST_CONFIG_DIR=/mnt/smb/docker/home-assistant/config
      HA_NETWORK=home-assistant_frontend
      ```

      Replace the values with your own accordingly.

4. **(Optional)** [Add the Code-server integration to your HA server](#integrating-code-server-with-home-assistant) so that the VS Code server could be used to configure HA configuration files from within HA.

### Integrating Code-server with Home Assistant

This details how to add the VS Code Server to the HA server as a dashboard:

1. [Integrate Code-server to your HA server as a dashboard](#adding-a-dashboard-to-home-assistant), while noting the following options:

   - Add dashboard: `Webpage`

   - URL: Set the URL of the VS Code server to `http://<code-server-host>:8443` (i.e. `http://192.168.0.106:8443`)

   - Dashboard information:

     - Title: Set a descriptive and unique title for the dashboard (i.e. `VS Code`)
     - Icon: Expand the dropdown and select an appropriate icon to represent the dashboard (i.e. `mdi:microsoft-visual-studio-code`)
     - Admin only: Check the corresponding switch to enable restricting access to only the admin users
     - Show in sidebar: Check the corresponding switch to enable showing the dashboard menu in the sidebar

2. Once the Code-server has been integrated with the HA server through this dashboard, you may access it right from the HA server to update HA configuration files accordingly.

---

## HACS

### Description

The Home Assistant Community Store (HACS) is a custom integration that provides a UI to manage custom elements in Home Assistant.

### References

- [HACS](https://hacs.xyz)
- [To download HACS](https://www.hacs.xyz/docs/use/download/download)
- [Initial configuration](https://www.hacs.xyz/docs/use/configuration/basic)
- [Home Assistant Community Store (HACS) - Installation Guide](https://youtu.be/oJfvT4uml3U)

### Install HACS on Home Assistant Container

> [!NOTE]  
> This guide assumes that you have [deployed and set up HA](#setup) as a Docker container.

This details the installation steps of HACS on the HA server:

1. [Enter the HA container](../courses/container.md#docker-usage) using the compose file (i.e. `/mnt/smb/docker/home-assistant/docker-compose.yml`), the `home-assistant` service, and the `bash` shell.

2. Inside the container, run the HACS installation script:

    ```sh
    wget -O - https://get.hacs.xyz | bash -
    ```

    Wait for the download and installation process to finish, after seeing the following output:

    ```
      INFO: Installation complete.

      INFO: Remember to restart Home Assistant before you configure it
    ```

3. Exit the HA container, and [restart](../courses/container.md#docker-usage) the `home-assistant` container service for the installation to take effect.

4. [Integrate HACS to your HA server](#adding-an-integration-to-home-assistant), while noting the following options:

   - Select brand: `HACS`

   - Before you can setup HACS you need to acknowledge the following: Check all of the presented checkboxes to acknowledge the statements and proceed

   - Waiting for device activation: Copy the provided key (i.e. `FRXY-8291`) and click on the provided GitHub link (i.e. `https://github.com/login/device`) to proceed with the user verification using a GitHub account

5. Once HACS has been installed and added to the HA server through this integration, you may access it right from the HA server to download and install custom elements or integrations to the server.

---

## Usage

### Description

This details some common usage steps for a Home Assistant (HA) server.

### References

- [Environment variables in yaml files](https://community.home-assistant.io/t/environment-variables-in-yaml-files/117157/4)
- [Configuration.yaml](https://www.home-assistant.io/docs/configuration)
- [Storing secrets](https://www.home-assistant.io/docs/configuration/secrets)
- [Environment variables](https://www.home-assistant.io/docs/configuration/yaml/#environment-variables)

### Configuring Home Assistant

This details how to configure HA through its configuration file (`configuration.yaml`) and secrets file (`secrets.yaml`):

1. Update the HA configuration file (`configuration.yaml`) from the host machine, or the HA server itself:

   - To update the config file from the host machine:

      ```sh
      nano <path-to-config>/configuration.yaml
      ```

   - **Alternatively**, to update the config file from the HA server (i.e. container):

      ```sh
      nano /config/configuration.yaml
      ```

2. Define your configuration options, bearing in mind its `yaml` syntax and configuration specifications, to the configuration file (`configuration.yaml`):

   - Typically, adding a configuration option to the config file would look something like so:

      ```diff
        some_existing_config:
          some_option: some_value
      +
      + my_config:
      +   my_option: my_value
      ```

      For example:

      ```yaml
        some_existing_config:
          some_option: some_value

        my_config:
          my_option: my_value
      ```

   - Note however that values set in the configuration file (i.e. `my_value`) needs to be set in plain text, which means that this method may not be secure for sensitive data such as passwords and tokens. To address that:

     - Set your secret value as an environment variable (i.e. `MY_SECRET_VAR`). For example, in the Docker compose file of your HA server:

        ```yaml
            environment:
              - MY_SECRET_VAR=some-secret-value
        ```

     - Update the HA secrets file (`secrets.yaml`) from the host machine, or the HA server itself:

        To update the secrets file from the host machine:

        ```sh
        nano <path-to-config>/secrets.yaml
        ```

        **Alternatively**, to update the secrets file from the HA server (i.e. container):

        ```sh
        nano /config/secrets.yaml
        ```

     - Add a variable (i.e. `my_secret_var`) to the secrets file and set the value to its corresponding environment variable (i.e. `MY_SECRET_VAR`):

        ```diff
          some_password: welcome
        + my_secret_var: !env_var MY_SECRET_VAR
        ```

        For example:

        ```yaml
          some_password: welcome
          my_secret_var: !env_var MY_SECRET_VAR
        ```

     - Finally, add the intended configuration option to the config file as you normally would, except, set the value to its corresponding variable from the secrets file (i.e. `my_secret_var`):

        ```diff
          some_existing_config:
            some_option: some_value
        +
        + my_config:
        +   my_option: !secret my_secret_var
        ```

        For example:

        ```yaml
          some_existing_config:
            some_option: some_value

          my_config:
            my_option: !secret my_secret_var
        ```

3. **(Optional)** If you are updating the configuration of a running HA server, verify that your updated configuration is valid before applying the configuration changes:

   - Visit `http://<ha-server-host>:8123/developer-tools/yaml` (i.e. `http://192.168.0.106:8123/developer-tools/yaml`) on a web browser.

   - Under the **Check and restart** section, click the **CHECK CONFIGURATION** button.

   - If the result returns a message such as the following:

      ```
        Configuration will not prevent Home Assistant from starting!
      ```

      Your pending configuration changes should be safe to apply.

4. Apply your pending configuration changes by deploying the HA server or restarting the server if it is already running.

### Adding an Integration to Home Assistant

This details how to add a supported integration to the HA server:

1. Launch the HA server web interface at `http://<ha-server-host>:8123` (i.e. `http://192.168.0.106:8123`) on a web browser.

2. Click on the **Settings** button found on the left sidebar of the web interface.

3. In the **Settings** page, click the **Devices & services** menu option.

4. By default, you should be taken to the **Integrations** section. If not, navigate to it accordingly.

5. Click the **ADD INTEGRATION** button found at the bottom of the **Integrations** page.

6. In the newly opened **Select brand** dialog, use the provided search bar to look for the name of the integration, and click on the corresponding entry.

7. You may be presented with more options to pick and configure when adding the integration - go through them accordingly.

8. You may also be presented with the option to set the location of the integration or device (i.e. `Living Room`) once you are done adding it to HA - do so or skip it as you see fit.

### Adding a Dashboard to Home Assistant

This details how to add a dashboard to the HA server:

1. Launch the HA server web interface at `http://<ha-server-host>:8123` (i.e. `http://192.168.0.106:8123`) on a web browser.

2. Click on the **Settings** button found on the left sidebar of the web interface.

3. In the **Settings** page, click the **Dashboards** menu option.

4. Click the **ADD DASHBOARD** button found at the bottom of the **Dashboards** page.

5. In the newly opened **Add dashboard** dialog, select the entry corresponding to the type of dashboard you wish to add (i.e. `Webpage`).

6. You may be presented with more options to pick and configure when adding the dashboard - go through them accordingly.
