# Docker

## Description

Docker is a set of platform as a service (PaaS) products that use OS-level virtualisation to deliver software in packages called containers.

## Directory

- [Docker](#docker)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Configuration](#configuration)
  - [Usage](#usage)
    - [Description](#description-2)
    - [References](#references-2)
    - [Docker Compose](#docker-compose)

## References

- [Docker](https://www.docker.com)
- [Wikipedia](https://en.wikipedia.org/wiki/Docker_(software))

---

## Setup

### Description

This details the installation and configuration process of Docker on a Linux system.

### References

- [Install Docker Engine](https://docs.docker.com/engine/install)
- [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
- [ArchWiki](https://wiki.archlinux.org/title/Docker)

### Installation

> [!NOTE]  
> Take note of steps that may be different on specific Linux distributions.

This details how to install Docker on a Linux system.

1. [Install](package-manager.md#install-software) the following package dependencies using your package manager (i.e. `apt`):

    **Debian/Ubuntu**:

   - `ca-certificates`
   - `curl`

    **RHEL**

   - `dnf-plugins-core`

2. Add and set up the Docker repository:

    **Debian**:

    ```sh
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

    **Ubuntu**:

    ```sh
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

    **RHEL**:

    ```sh
    sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
    ```

3. [Update the system](package-manager.md#update-software)'s software repository using your package manager (i.e. `apt`).

4. [Install](package-manager.md#install-software) the latest version of Docker packages using your package manager (i.e. `apt`):

    **Arch Linux**:

    - `docker`
    - `docker-buildx`
    - `docker-compose`

    **Debian/Ubuntu/RHEL**:

    - `docker-ce`
    - `docker-ce-cli`
    - `containerd.io`
    - `docker-buildx-plugin`
    - `docker-compose-plugin`

5. [Start and enable](systemd.md#enable-service) the following Docker services:

   - `docker.service`
   - `containerd.service`

### Configuration

This details how to configure Docker to allow a non-root user to run it.

1. [Create the `docker` group](linux.md#create-group) on the system.

2. [Add the service (non-root) user to the `docker` group](linux.md#add-user-to-group) to give the user root-level privileges for running Docker.

3. Reboot the server to apply the changes.

---

## Usage

### Description

This details some common usage steps for a container runtime solution such as Docker.

### References

- [Docker Compose](https://docs.docker.com/compose)
- [Compose file reference](https://docs.docker.com/compose/compose-file)

### Docker Compose

This details how to manage a containerised application deployment using Docker Compose:

1. To deploy a containerised application using Docker Compose:

   - Create a dedicated project directory (i.e. `~/.local/share/myapp`) for the containerised application:

      ```sh
      mkdir -p <project-directory>
      ```

      For example:

      ```sh
      mkdir -p ~/.local/share/myapp
      ```

   - Create a Docker compose file (i.e. `docker-compose.yml`) in the project directory (i.e. `~/.local/share/myapp`):

      ```sh
      nano <project-directory>/<compose-file>
      ```

      For example:

      ```sh
      nano ~/.local/share/myapp/docker-compose.yml
      ```

   - Define the containerised application's services in the compose file:

     - Sample compose file:

        ```yaml
        services:
          myapp:
            container_name: myapp
            image: docker.io/myuser/myapp:0.1.0
            ports:
              - 8000:8000
            environment:
              - TZ=Etc/UTC
            volumes:
              - ~/.local/share/myapp/config:/config
            restart: unless-stopped
            security_opt:
              - no-new-privileges:true
        ```

     - **(Optional)** You could also declare some of the values in the compose file as variables. For example:

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

        ... and create an env file in the same directory as the compose file (i.e. `~/.local/share/myapp/.env`):

        ```sh
        nano ~/.local/share/myapp/.env
        ```

        ... and set the defined variables in the env file. For example:

        ```env
        APP_VERSION=0.1.0
        HOST_PORT=8000
        APP_PORT=8000
        APP_TIMEZONE=Etc/UTC
        APP_DIR=~/.local/share/myapp
        ```

     - Save the compose file (and env file) accordingly.

   - Deploy the application stack using the full path to the application's corresponding compose file (i.e. `~/.local/share/myapp/docker-compose.yml`):

      ```sh
      docker compose -f <compose-file-path> up -d --force-recreate
      ```

      For example:

      ```sh
      docker compose -f ~/.local/share/myapp/docker-compose.yml up -d --force-recreate
      ```

      **Alternatively**, if you have supplied an env file, be sure to add the option to include the full path to the env file (i.e. `~/.local/share/myapp/.env`):

      ```sh
      docker compose -f <compose-file-path> --env-file <env-file-path> up -d --force-recreate
      ```

2. To verify that all container(s) in the application stack are running:

    ```sh
    docker compose -f <compose-file-path> ps -a
    ```

    For example:

    ```sh
    docker compose -f ~/.local/share/myapp/docker-compose.yml ps -a
    ```

3. To enter a running container in the application stack:

    ```sh
    docker compose -f <compose-file-path> exec <service-name> <shell>
    ```

    For example:

    ```sh
    docker compose -f ~/.local/share/myapp/docker-compose.yml exec myapp sh
    ```

4. To view and follow the logs of container(s) in the application stack:

    ```sh
    docker compose -f <compose-file-path> logs --follow
    ```

    For example:

    ```sh
    docker compose -f ~/.local/share/myapp/docker-compose.yml logs --follow
    ```

5. To stop the running application stack without removing the containers:

    ```sh
    docker compose -f <compose-file-path> stop
    ```

    For example:

    ```sh
    docker compose -f ~/.local/share/myapp/docker-compose.yml stop
    ```

6. To remove the application stack:

    ```sh
    docker compose -f <compose-file-path> down
    ```

    For example:

    ```sh
    docker compose -f ~/.local/share/myapp/docker-compose.yml down
    ```

    **Alternatively**, to remove the application stack and its volumes:

    ```sh
    docker compose -f <compose-file-path> down --volumes
    ```

    Be aware though that this option will remove all of the application's persistent data permanently.
