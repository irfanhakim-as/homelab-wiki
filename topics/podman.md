# Podman

## Description

Podman (the POD MANager) is a tool for managing containers and images, volumes mounted into those containers, and pods made from groups of containers.

## Directory

- [Podman](#podman)
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
    - [Podman Compose](#podman-compose)

## References

- [Podman](https://podman.io)
- [GitHub](https://github.com/containers/podman)
- [ArchWiki](https://wiki.archlinux.org/title/Podman)

---

## Setup

### Description

This details the installation and configuration process of Podman on a Linux system.

### References

- [Install Podman in a static manner](https://github.com/89luca89/distrobox/blob/main/docs/posts/install_podman_static.md)
- [[Error] podman returns "Error: open /etc/containers/policy.json: no such file or directory" lilipod works](https://github.com/89luca89/distrobox/issues/1509)
- [policy.json](https://raw.githubusercontent.com/containers/podman/refs/heads/main/test/policy.json)
- [containers-policy.json](https://man.archlinux.org/man/containers-policy.json.5.en)
- [Podman Systemd unit templates](https://github.com/containers/podman/tree/main/contrib/systemd/user)
- [Podman Compose](https://github.com/containers/podman-compose)

### Installation

> [!NOTE]  
> Take note of steps that may be different depending on the method of installation.

This details how to install Podman on a Linux system:

1. Ensure both `/etc/subuid` and `/etc/subgid` are present on the system. If not, create the files:

    ```sh
    sudo touch /etc/subuid /etc/subgid
    ```

    and add a `subuid` and `subgid`:

    ```sh
    sudo usermod --add-subuid 100000-165535 --add-subgid 100000-165535 ${USER}
    ```

2. [Install](package-manager.md#install-software) the `podman` package using your package manager (i.e. `nix`) if it is available in your software repository. **Otherwise**, install from source:

   - Write a bash script that helps fetch and install the latest `podman` release:

      ```sh
      nano ~/get-podman-launcher.sh
      ```

      Add the following content to the script:

      ```sh
      curl -Lo "${HOME}/podman-launcher-amd64" "https://github.com/89luca89/podman-launcher/releases/latest/download/podman-launcher-amd64"
      chmod +x "${HOME}/podman-launcher-amd64"
      mkdir -p "${HOME}/.local/bin"
      mv -f "${HOME}/podman-launcher-amd64" "${HOME}/.local/bin/podman"
      ```

   - Install `podman` by running the script:

      ```sh
      bash ~/get-podman-launcher.sh
      ```

3. If you encounter any of the following errors in an attempt to use `podman` post-installation:

   - Podman requires a policy configuration file, but one may not be provided by default:

      ```
        Error: open /etc/containers/policy.json: no such file or directory
      ```

      To fix this:

      - Create the following configuration directory on the system:

        ```sh
        mkdir -p ~/.config/containers
        ```

      - Create the following policy configuration file:

        ```sh
        nano ~/.config/containers/policy.json
        ```

        Add and save the following configuration to the file:

        ```json
        {
          "default": [
            {
              "type": "insecureAcceptAnything"
            }
          ]
        }
        ```

   - Podman needs `newuidmap` and `newgidmap` for rootless containers:

      ```
        Error: command required for rootless mode with multiple IDs: exec: "newuidmap": executable file not found in $PATH
      ```

      To fix this, find the package that provides both of these commands and [install](package-manager.md#install-software) it using your package manager (i.e. `apt`).

4. To set up the necessary `systemd` unit files for Podman, create and enable the following `systemd` unit files on the system:

   - First and foremost, [check if the following `systemd` units already exist](systemd.md#list-systemd-units) on the system, both system/user-wide:

     - `podman.service`
     - `podman.socket`
     - `podman-restart.service`

   - For any of the above units that do not exist, create their corresponding unit file(s) according to the following steps:

     - First and foremost, create the following `systemd` configuration directory if it does not already exist:

        ```sh
        mkdir -p ~/.config/systemd/user
        ```

     - **(`podman.service`)** Create the following service unit file:

        ```sh
        nano ~/.config/systemd/user/podman.service
        ```

        Add and save the following configuration to the file:

        ```ini
        [Unit]
        Description=Podman API Service
        Requires=podman.socket
        After=podman.socket
        Documentation=man:podman-system-service(1)
        StartLimitIntervalSec=0

        [Service]
        Delegate=true
        Type=exec
        KillMode=process
        Environment=LOGGING="--log-level=info"
        ExecStart=</path/to/podman> system service

        [Install]
        WantedBy=default.target
        ```

        Replace `</path/to/podman>` with the actual path to the `podman` executable (i.e. `/usr/bin/podman`) on the system.

     - **(`podman.socket`)** Create the following socket unit file:

        ```sh
        nano ~/.config/systemd/user/podman.socket
        ```

        Add and save the following configuration to the file:

        ```ini
        [Unit]
        Description=Podman API Socket
        Documentation=man:podman-system-service(1)

        [Socket]
        ListenStream=%t/podman/podman.sock
        SocketMode=0660

        [Install]
        WantedBy=sockets.target
        ```

     - **(`podman-restart.service`)** Create the following service unit file:

        ```sh
        nano ~/.config/systemd/user/podman-restart.service
        ```

        Add and save the following configuration to the file:

        ```ini
        [Unit]
        Description=Podman Start All Containers With Restart Policy Set To Always
        Documentation=man:podman-start(1)
        StartLimitIntervalSec=0
        Wants=network-online.target
        After=network-online.target

        [Service]
        Type=oneshot
        RemainAfterExit=true
        Environment=LOGGING="--log-level=info"
        ExecStart=</path/to/podman> $LOGGING start --all --filter restart-policy=always
        ExecStop=</path/to/podman> $LOGGING stop --all --filter restart-policy=always

        [Install]
        WantedBy=default.target
        ```

        Replace `</path/to/podman>` with the actual path to the `podman` executable (i.e. `/usr/bin/podman`) on the system.

      **Otherwise**, if all of the above units already exist, skip this step entirely.

   - Lastly, [reload the `systemd` manager configuration](systemd.md#reload-systemd-manager-configuration) to apply the changes made and [enable](systemd.md#enable-service) the following `systemd` units:

     - `podman.socket` (you may need to [start](systemd.md#start-service) the socket as well)
     - `podman-restart.service`

5. To ensure that Podman containers that have been configured to start on boot (i.e. `restart-policy=always`) can do so successfully without needing to log into the system first, enable linger for boot startup:

    ```sh
    sudo loginctl enable-linger ${USER}
    ```

    Check that it has been enabled:

    ```sh
    loginctl show-user ${USER} | grep -i linger
    ```

    Sample output:

    ```
      Linger=yes
    ```

6. Go through the [post-installation setup steps](#post-install-setup) to complete the Podman setup.

### Post-Install Setup

This details the post-installation steps for a complete Podman setup:

1. For a _[Docker Compose](docker.md#docker-compose)-like_ compatibility, [install](package-manager.md#install-software) either one of the following packages using your package manager (i.e. `nix`) if it is available in your software repository:

   - `docker-compose`
   - `podman-compose`

---

## Usage

### Description

This details some common usage steps for a Container Runtime solution such as Podman.

### References

- [Getting started with Compose](https://podman-desktop.io/tutorial/getting-started-with-compose)
- [The Compose Specification](https://github.com/compose-spec/compose-spec/blob/main/spec.md)

### Podman Compose

> [!NOTE]  
> [Either Docker Compose or Podman Compose needs to be installed](#post-install-setup) before you are able to proceed with the following operations.

This details how to manage a containerised application deployment using Podman Compose:

1. To deploy a containerised application using Podman Compose:

   - Create a dedicated project directory (i.e. `~/.local/share/myapp`) for the containerised application:

      ```sh
      mkdir -p <project-directory>
      ```

      For example:

      ```sh
      mkdir -p ~/.local/share/myapp
      ```

   - Create a Compose file (i.e. `docker-compose.yml`) in the project directory (i.e. `~/.local/share/myapp`):

      ```sh
      nano <project-directory>/<compose-file>
      ```

      For example:

      ```sh
      nano ~/.local/share/myapp/docker-compose.yml
      ```

   - Define the containerised application's services in the Compose file:

     - Sample Compose file:

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
              - /home/myuser/.local/share/myapp/config:/config
            restart: unless-stopped
            security_opt:
              - no-new-privileges:true
        ```

        Do note that the `unless-stopped` `restart` policy on Podman _may not_ work across reboots the way it would with [Docker](docker.md) - if this is true and you need the containerised application to restart on boot, set the `restart` policy to `always` instead.

     - **(Optional)** You could also declare some of the values in the Compose file as variables. For example:

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

        ... and create an env file in the same directory as the Compose file (i.e. `~/.local/share/myapp/.env`):

        ```sh
        nano ~/.local/share/myapp/.env
        ```

        ... and set the defined variables in the env file. For example:

        ```sh
        APP_VERSION=0.1.0
        HOST_PORT=8000
        APP_PORT=8000
        APP_TIMEZONE=Etc/UTC
        APP_DIR=/home/myuser/.local/share/myapp
        ```

     - Save the Compose file (and env file) accordingly.

   - Deploy the application stack using the full path to the application's corresponding Compose file (i.e. `~/.local/share/myapp/docker-compose.yml`):

      ```sh
      podman-compose -f <compose-file-path> up -d --force-recreate
      ```

      For example:

      ```sh
      podman-compose -f ~/.local/share/myapp/docker-compose.yml up -d --force-recreate
      ```

      **Alternatively**, if you have supplied an env file, be sure to add the option to include the full path to the env file (i.e. `~/.local/share/myapp/.env`):

      ```sh
      podman-compose -f <compose-file-path> --env-file <env-file-path> up -d --force-recreate
      ```

2. To verify that all container(s) in the application stack are running:

    ```sh
    podman-compose -f <compose-file-path> ps -a
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml ps -a
    ```

3. To enter a running container in the application stack:

    ```sh
    podman-compose -f <compose-file-path> exec <service-name> <shell>
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml exec myapp sh
    ```

4. To view and follow the logs of container(s) in the application stack:

    ```sh
    podman-compose -f <compose-file-path> logs --follow
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml logs --follow
    ```

5. To stop the running application stack without removing the containers:

    ```sh
    podman-compose -f <compose-file-path> stop
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml stop
    ```

6. To restart the container services in the application stack:

    ```sh
    podman-compose -f <compose-file-path> restart
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml restart
    ```

    **Alternatively**, to restart a specific container service in the application stack:

    ```sh
    podman-compose -f <compose-file-path> restart <service-name>
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml restart myapp
    ```

7. To remove the application stack:

    ```sh
    podman-compose -f <compose-file-path> down
    ```

    For example:

    ```sh
    podman-compose -f ~/.local/share/myapp/docker-compose.yml down
    ```

    **Alternatively**, to remove the application stack and its volumes:

    ```sh
    podman-compose -f <compose-file-path> down --volumes
    ```

    Be aware though that this option will remove all of the application's persistent data permanently.
