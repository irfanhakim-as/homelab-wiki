# Portainer

## Description

Portainer is an open-source, lightweight, and user-friendly web interface for managing Docker, Docker Swarm, Kubernetes, and Azure Container Instances.

## Directory

- [Portainer](#portainer)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Post-Install Setup](#post-install-setup)
  - [Usage](#usage)
    - [Description](#description-2)
    - [References](#references-2)
    - [Deploying Containers](#deploying-containers)
    - [Entering Containers](#entering-containers)
    - [Monitoring Container Logs](#monitoring-container-logs)
    - [Stopping Containers](#stopping-containers)
    - [Restarting Containers](#restarting-containers)
    - [Deleting Containers](#deleting-containers)

## References

- [Portainer](https://www.portainer.io)

---

## Setup

### Description

This details how to install and set up a Portainer server in a containerised environment.

### References

- [Install Portainer CE with Docker on Linux](https://docs.portainer.io/start/install-ce/server/docker/linux)
- [A Guide to Homelab: Your FIRST container - Docker & Portainer](https://youtu.be/-d4Y3Zhhwcw)

### Installation

This details how to install Portainer as a containerised application:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime). For simplicity, the following considerations should be noted:

   - Ports that will be used and requires allowing vary for each container you host on the Portainer server, hence, it is recommended not to enable or set up a [firewall](firewall.md) on the system.

2. [Deploy the Portainer stack with Compose](../courses/container.md#container-runtime-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/portainer`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/portainer`): This will be used for the Portainer stack's volume(s).

   - A Compose file for the Portainer stack on the app directory (i.e. `/mnt/smb/docker/portainer/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        portainer:
          container_name: ${APP_CONTAINER}
          image: docker.io/portainer/portainer-ce:${APP_VERSION}
          ports:
            - 8000:8000
            - 9443:9443
            - 9000:9000 # for http
          volumes:
            - ${APP_DIR}/data:/data
            - ${CONTAINER_RUNTIME_SOCKET}:/var/run/docker.sock
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

   - An env file for the Portainer stack on the app directory (i.e. `/mnt/smb/docker/portainer/.env`):

      ```sh
      SERVICE_NAME=portainer
      APP_CONTAINER=portainer
      APP_VERSION=2.27.9
      APP_DIR=/mnt/smb/docker/portainer
      CONTAINER_RUNTIME_SOCKET=/var/run/docker.sock
      ```

      The `CONTAINER_RUNTIME_SOCKET` value is dependent on the Container Runtime you are using for the deployment:

      - If you are using Docker, get the path to the Docker socket file using:

        ```sh
        docker context inspect --format '{{.Endpoints.docker.Host}}' | sed 's|.*://||'
        ```

        Sample output:

        ```
          /var/run/docker.sock
        ```

      - If you are using Podman, get the path to the Podman socket file using:

        ```sh
        podman info --format '{{.Host.RemoteSocket.Path}}'
        ```

        Sample output:

        ```
          /run/user/1000/podman/podman.sock
        ```

      Replace the values with your own accordingly.

3. Go through the [post-installation setup steps](#post-install-setup) for the Portainer server.

### Post-Install Setup

This details the post-installation steps of the Portainer stack for a complete setup:

1. Launch the Portainer server web interface at `http://<portainer-server-host>:9000` (i.e. `http://192.168.0.106:9000`) on a web browser.

2. For the initial launch of the Portainer web interface, under the **New Portainer installation** section, fill in the following information in the provided form:

   - Username: Set a unique, descriptive username for the Portainer user (i.e. `admin`)
   - Password: Set a secure password for the Portainer user
   - Confirm password: Confirm the password you have set for the Portainer user

   Submit the form by clicking the **Create user** button.

3. In the **Quick Setup** page, click the **Get Started** button to "proceed using the local environment which Portainer is running in".

4. In the **Home** page, under the **Environments** section, click the **local** environment.

5. Going forward, [anything you do](#usage) pertaining to your Portainer environment could be done from the selected environment's **Dashboard** page here.

---

## Usage

### Description

This details some common usage steps for managing containers and application stacks in a Portainer environment.

### References

- [Using Portainer: Docker/Swarm/Podman](https://docs.portainer.io/user/docker)

### Deploying Containers

This details how to deploy or update container(s) in an application stack:

1. From the selected environment's **Dashboard** page, click the **Stacks** menu option.

2. From the **Stacks list** view in the **Stacks** page, click the **Add stack** button to add a new application stack. **Alternatively**, select the **Name** link corresponding to the application stack you wish to update.

3. If you are adding a new stack, in the **Create stack** form, configure the following:

   - Name: Set a unique, descriptive name for the application stack (i.e. `myapp`)
   - Build method: Select the **Web editor** menu option to define the Compose and env file from the web interface

    **Alternatively**, if you are updating an existing stack, in the **Stack details** view, click the **Editor** section.

4. In the provided **Web editor** for the application stack, define or update the content of its Compose file. For example:

    ```yaml
    services:
      myapp:
        container_name: myapp
        image: docker.io/myuser/myapp:${APP_VERSION}
        ports:
          - ${HOST_PORT}:${APP_PORT}
        environment:
          - TZ=${APP_TIMEZONE}
        volumes:
          - ${APP_DIR}/config:/config
        restart: unless-stopped
        security_opt:
          - no-new-privileges:true
    ```

5. **(Optional)** Create an env file (`stack.env`) for the application stack by expanding the **Environment variables** section (if it is collapsed), clicking the **Advanced mode** link, and define or update the content of the env file using the provided editor accordingly. For example:

    ```sh
    APP_VERSION=0.1.0
    HOST_PORT=8000
    APP_PORT=8000
    APP_TIMEZONE=Etc/UTC
    APP_DIR=/home/myuser/.local/share/myapp
    ```

6. **(Optional)** In the **Access control** section, enable the switch corresponding to the **Enable access control** option to control access to the application stack. **Alternatively**, if you are updating an existing stack, click the **Change ownership** link in the same section.

7. Under the **Actions** section, click the **Deploy the stack** button to deploy the application stack for the first time. **Alternatively**, click the **Update the stack** button if you are updating an existing stack.

### Entering Containers

This details how to enter the container(s) in an application stack:

1. From the selected environment's **Dashboard** page, click the **Containers** menu option.

2. From the **Container list** view in the **Containers** page, select the **Name** link corresponding to the container you wish to enter.

3. In the container's **Container details** view, under the **Container status** section, click the **Console** link.

4. In the container's **Container console** view, under the **Execute** section, configure the following:

   - Command: Expand the dropdown and select the intended shell option (i.e. `/bin/bash`)
   - Use custom command: **Alternatively**, check the corresponding switch to enable this option and fill in the **Command** field with the intended custom command (i.e. `ps aux`)
   - User: Set a specific user to enter the container as or leave as default (i.e. `root`)

    Click the **Connect** button.

5. Once you have done what you needed to do in the container, click the **Disconnect** button to exit the container.

### Monitoring Container Logs

This details how to follow the logs of container(s) in an application stack:

1. From the selected environment's **Dashboard** page, click the **Containers** menu option.

2. From the **Container list** view in the **Containers** page, select the **Name** link corresponding to the container you wish to monitor.

3. In the container's **Container details** view, under the **Container status** section, click the **Logs** link.

4. View and follow the logs of the container from the **Container logs** view. **Optionally**, configure the **Log viewer settings** form as necessary.

### Stopping Containers

This details how to stop container(s) in an application stack:

1. From the selected environment's **Dashboard** page, click the **Stacks** menu option.

2. From the **Stacks list** view in the **Stacks** page, select the **Name** link corresponding to the application stack which the container(s) belong to.

3. In the application stack's **Stack details** view, under the **Stack** section, click the **Stop this stack** button to stop all container(s) in the application stack.

4. **Alternatively**, to stop a specific selection of container(s), under the **Containers** section, select the corresponding checkboxes of container(s) you wish to stop, and click the **Stop** button.

### Restarting Containers

This details how to restart container(s) in an application stack:

1. From the selected environment's **Dashboard** page, click the **Stacks** menu option.

2. From the **Stacks list** view in the **Stacks** page, select the **Name** link corresponding to the application stack which the container(s) belong to.

3. In the application stack's **Stack details** view, under the **Containers** section, select the corresponding checkboxes of container(s) you wish to restart, and click the **Restart** button.

### Deleting Containers

This details how to delete container(s) in an application stack:

1. From the selected environment's **Dashboard** page, click the **Stacks** menu option.

2. From the **Stacks list** view in the **Stacks** page, select the **Name** link corresponding to the application stack which the container(s) belong to.

3. In the application stack's **Stack details** view, under the **Stack** section, click the **Delete this stack** button to permanently delete all container(s) in the application stack.

4. **Alternatively**, to delete a specific selection of container(s), under the **Containers** section, select the corresponding checkboxes of container(s) you wish to delete, and click the **Remove** button.
