# Borg

## Description

BorgBackup (commonly referred to as Borg) is an open-source deduplicating backup program with support for compression and authenticated encryption.

## Directory

- [Borg](#borg)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Borg Server](#borg-server)
    - [Description](#description-1)
    - [References](#references-1)
    - [Borg Server Installation](#borg-server-installation)
      - [Install Borg Server on Docker](#install-borg-server-on-docker)
    - [Borg Server Post-Install Setup](#borg-server-post-install-setup)
      - [Borg Server Post-Install Setup on Docker](#borg-server-post-install-setup-on-docker)
  - [Borg Client](#borg-client)
    - [Description](#description-2)
    - [References](#references-2)
    - [Borg Client Installation](#borg-client-installation)
      - [Install Borg Client on Docker](#install-borg-client-on-docker)
      - [Install Borg Client on Helm](#install-borg-client-on-helm)
    - [Borg Client Post-Install Setup](#borg-client-post-install-setup)
      - [Borg Client Post-Install Setup on Docker](#borg-client-post-install-setup-on-docker)
      - [Borg Client Post-Install Setup on Helm](#borg-client-post-install-setup-on-helm)
  - [Usage](#usage)
    - [Description](#description-3)
    - [References](#references-3)
    - [Listing Archives](#listing-archives)
    - [Restoring from Backup](#restoring-from-backup)

## References

- [BorgBackup](https://www.borgbackup.org)
- [Wikipedia](https://en.wikipedia.org/wiki/Borg_(backup_software))

---

## Borg Server

### Description

This details how to deploy a Borg server in a containerised environment.

### References

- [BorgServer](https://github.com/Nold360/borgserver)

### Borg Server Installation

#### Install Borg Server on Docker

This details the installation steps for the Borg server as a containerised application using Docker:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime). The following considerations should be noted:

   - For each Borg client that will connect to this server, generate a dedicated SSH key pair:

     ```sh
     ssh-keygen -t ed25519 -C "<client-name>" -f ~/.ssh/<client-name> -N ""
     ```

     For example, for a client named `my-client`:

     ```sh
     ssh-keygen -t ed25519 -C "my-client" -f ~/.ssh/my-client -N ""
     ```

     This produces the private key (i.e. `~/.ssh/my-client`) and the public key (i.e. `~/.ssh/my-client.pub`) without a passphrase (as required for unattended automated backups). The [private key will be placed on the Borg client](#borg-client-installation) host, and the public key will be placed on the Borg server.

   - Either [disable the firewall](firewall.md#disablement) on the host deployment environment or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule):

     - `2222/tcp` (or whichever port you intend to configure as the SSH listening port)

2. [Deploy the Borg server stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - An app directory, on local or remote storage (i.e. `/home/myuser/.local/share/docker/borg-server`): This will be used for the server's SSH key configuration and related data.

   - A repository directory, on local or remote storage (i.e. `/mnt/storage/borg-repos`): This will be used to store the Borg backup repositories. Each Borg client [initialises their repository](#borg-client-post-install-setup) at a path within this directory as defined in their configuration - note that this only creates the final repository directory, so any intermediate directories in the path must be pre-created manually on the server beforehand.

   - A Compose file for the Borg server stack on the app directory (i.e. `/home/myuser/.local/share/docker/borg-server/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        borgserver:
          container_name: ${APP_CONTAINER}
          image: ghcr.io/nold360/borgserver:${APP_VERSION}
          ports:
            - "${SSH_PORT}:22/tcp"
          environment:
            - BORG_SERVE_ARGS=--restrict-to-path /backup
            - PUID=${HOST_USER_UID}
            - PGID=${HOST_USER_GID}
          volumes:
            - ${APP_DIR}/sshkeys:/sshkeys
            - ${REPO_DIR}:/backup
          labels:
            - exclude-from-borg-backup
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

   - An env file for the Borg server stack on the app directory (i.e. `/home/myuser/.local/share/docker/borg-server/.env`):

      ```sh
      SERVICE_NAME=borg-server
      APP_CONTAINER=borg-server
      APP_VERSION=1.4
      APP_DIR=/home/myuser/.local/share/docker/borg-server
      REPO_DIR=/mnt/storage/borg-repos
      SSH_PORT=2222
      HOST_USER_UID=1000
      HOST_USER_GID=1000
      ```

      Replace all of the values in the env file with your own accordingly.

   - Set up the Borg clients' SSH public keys in the app directory:

     - Pre-create the `sshkeys/clients/` subdirectory inside the app directory (i.e. `/home/myuser/.local/share/docker/borg-server`):

        ```sh
        mkdir -p <app-dir>/sshkeys/clients
        ```

        For example:

        ```sh
        mkdir -p /home/myuser/.local/share/docker/borg-server/sshkeys/clients
        ```

     - For each Borg client, copy the corresponding public key into the `sshkeys/clients/` directory without the `.pub` extension:

        ```sh
        cp ~/.ssh/<client-name>.pub <app-dir>/sshkeys/clients/<client-name>
        ```

        For example:

        ```sh
        cp ~/.ssh/my-client.pub /home/myuser/.local/share/docker/borg-server/sshkeys/clients/my-client
        ```

3. Go through the [post-installation setup steps](#borg-server-post-install-setup-on-docker) to complete the Borg server setup.

### Borg Server Post-Install Setup

#### Borg Server Post-Install Setup on Docker

> [!NOTE]  
> This guide assumes that you have [installed the Borg server](#install-borg-server-on-docker) as a containerised application.

This details the post-installation steps of the Borg server for a complete setup:

1. Verify that the Borg server is accessible by connecting to it over SSH using one of the configured client keys:

   ```sh
   ssh -i ~/.ssh/<client-name> -p <ssh-port> borg@<borg-server-host>
   ```

   For example:

   ```sh
   ssh -i ~/.ssh/my-client -p 2222 borg@borg.example.com
   ```

   A successful connection will place you in a restricted shell. Enter `exit` to disconnect.

2. **(Optional)** Set up [port forwarding](../courses/network.md#port-forwarding) on your router if the Borg server needs to be accessible to Borg clients outside the local network:

   - Service name: Set any suitable name that best describes the service or rule (i.e. `borg-server`)
   - Device IP address: Enter the private IP address of the Borg server (i.e. `192.168.0.106`)
   - Internal port: Enter the port number configured as the SSH listening port (i.e. `2222`)
   - External port: Enter the port number that remote clients will connect to (i.e. `2222`)
   - Protocol: `TCP`
   - Enabled: Toggle or check this box to enable the port forwarding rule

---

## Borg Client

### Description

This details how to deploy a Borg client to back up data to a Borg server.

### References

- [Borgmatic](https://github.com/borgmatic-collective/borgmatic)
- [Borgmatic in Docker](https://github.com/borgmatic-collective/docker-borgmatic)
- [moekai/borgmatic](https://charts.moekai.com/moekai/borgmatic)

### Borg Client Installation

#### Install Borg Client on Docker

> [!NOTE]  
> This guide assumes that you have [deployed a Borg server](#borg-server-installation) and have the corresponding SSH private key for the Borg client available.

This details the installation steps for the Borg client as a containerised application using Docker:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime).

2. [Deploy the Borg client stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - An app directory, on local or remote storage (i.e. `/home/myuser/.local/share/docker/borg-client`): This will be used for the client's backup configuration, SSH key, and state data.

   - A Compose file for the Borg client stack on the app directory (i.e. `/home/myuser/.local/share/docker/borg-client/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        borgmatic-client:
          container_name: ${APP_CONTAINER}
          image: ghcr.io/borgmatic-collective/borgmatic:${APP_VERSION}
          environment:
            - BACKUP_CRON=${CRON_SCHEDULE}
            - TZ=${APP_TIMEZONE}
            - BORG_PASSPHRASE=${BORG_PASSPHRASE}
            - DOCKERCLI=true
          volumes:
            - ${APP_DIR}/config:/etc/borgmatic.d:ro
            - ${APP_DIR}/ssh:/root/.ssh
            - ${APP_DIR}/cache:/root/.cache/borg
            - ${APP_DIR}/state:/root/.local/state/borgmatic
            - ${APP_DIR}/borg-config:/root/.config/borg
            - ${BACKUP_SRC}:${BACKUP_SRC}:ro
            - /var/run/docker.sock:/var/run/docker.sock
          labels:
            - exclude-from-borg-backup
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

      If you need to back up multiple source directories, add an additional volume entry for each one following the `${BACKUP_SRC}:${BACKUP_SRC}:ro` pattern (i.e. `${BACKUP_SRC_2}:${BACKUP_SRC_2}:ro`) and define the corresponding variables in the env file.

   - An env file for the Borg client stack on the app directory (i.e. `/home/myuser/.local/share/docker/borg-client/.env`):

      ```sh
      SERVICE_NAME=borg-client
      APP_CONTAINER=borg-client
      APP_VERSION=2.1.3
      APP_TIMEZONE=Asia/Kuala_Lumpur
      APP_DIR=/home/myuser/.local/share/docker/borg-client
      BACKUP_SRC=/home/myuser/.local/share/docker
      CRON_SCHEDULE=0 2 * * *
      BORG_PASSPHRASE=your-secure-passphrase
      ```

      Replace all of the values in the env file with your own accordingly.

   - Set up the required subdirectories and SSH private key in the app directory:

     - Pre-create the required subdirectories inside the app directory (i.e. `/home/myuser/.local/share/docker/borg-client`):

        ```sh
        mkdir -p <app-dir>/{config,ssh,cache,state,borg-config}
        ```

        For example:

        ```sh
        mkdir -p /home/myuser/.local/share/docker/borg-client/{config,ssh,cache,state,borg-config}
        ```

     - Copy the SSH private key into the `ssh/` subdirectory and restrict its permissions:

        ```sh
        cp ~/.ssh/<client-name> <app-dir>/ssh/<client-name>
        chmod 600 <app-dir>/ssh/<client-name>
        ```

        For example:

        ```sh
        cp ~/.ssh/my-client /home/myuser/.local/share/docker/borg-client/ssh/my-client
        chmod 600 /home/myuser/.local/share/docker/borg-client/ssh/my-client
        ```

   - A backup configuration file inside the `config/` subdirectory in the app directory (i.e. `/home/myuser/.local/share/docker/borg-client/config/config.yaml`):

      ```yaml
      ssh_command: ssh -i /root/.ssh/<client-name> -o StrictHostKeyChecking=accept-new

      repositories:
        - path: ssh://borg@<borg-server-host>:<ssh-port>/backup/<client-name>
          label: <repo-label>
          encryption: repokey-blake2

      source_directories:
        - /home/myuser/.local/share/docker

      exclude_patterns:
        - /home/myuser/.local/share/docker/borg-client

      mysql_databases:
        - name: all
          hostname: mysql
          port: 3306
          username: root
          password: <mysql-root-password>
          options: "--single-transaction --quick"

      mariadb_databases:
        - name: all
          hostname: mariadb
          port: 3306
          username: root
          password: <mariadb-root-password>

      postgresql_databases:
        - name: all
          hostname: postgres
          port: 5432
          username: postgres
          password: <pg-password>

      compression: lz4

      keep_daily: 7
      keep_weekly: 4
      keep_monthly: 6

      checks:
        - name: repository
          frequency: 2 weeks
        - name: archives
          frequency: 4 weeks

      apprise:
        services:
          - label: email
            url: mailtos://<smtp-host>:<smtp-port>?user=<email-address>&pass=<email-password>&from=<email-address>&to=<email-address>
        send_logs: true
        states:
          - finish
          - fail
        finish:
          title: Borgmatic backup finished
          body: Backup completed successfully.
        fail:
          title: Borgmatic backup FAILED
          body: Backup failed. Check logs.

      commands:
        - before: action
          when:
            - create
          run:
            - "docker ps -q --filter label=exclude-from-borg-backup > /tmp/borg-exclude.txt; docker ps -q | grep -v -F -f /tmp/borg-exclude.txt > /tmp/borg-stopped.txt; cat /tmp/borg-stopped.txt | xargs -r docker stop"
        - after: action
          when:
            - create
          run:
            - "if [ -f /tmp/borg-stopped.txt ]; then cat /tmp/borg-stopped.txt | xargs -r docker start; else docker ps -a -q --filter status=exited | xargs -r docker start; fi"
        - after: error
          run:
            - "if [ -f /tmp/borg-stopped.txt ]; then cat /tmp/borg-stopped.txt | xargs -r docker start; else docker ps -a -q --filter status=exited | xargs -r docker start; fi"
      ```

      The `commands` hooks stop all Docker containers on the same host that do not carry the `exclude-from-borg-backup` label before each backup and restart them afterwards. The Borg client container itself carries this label and is never stopped. Remove this section if you do not need this behaviour.

      Update the configuration above to match your actual backup setup.

3. Go through the [post-installation setup steps](#borg-client-post-install-setup-on-docker) to complete the Borg client setup.

#### Install Borg Client on Helm

> [!NOTE]  
> This guide assumes that you have [deployed a Borg server](#borg-server-installation) and have the corresponding SSH private key for the Borg client available.

This details the installation steps for the Borg client on a Kubernetes cluster using Helm:

1. Ensure Helm is [installed](helm.md#installation) on your system.

2. Add the Moekai [Helm chart repository](helm.md#adding-chart-repository) to your system:

   - Repository name: `moekai`
   - Repository source: `https://charts.moekai.com`

3. Get the [Helm values file](helm.md#get-helm-chart-values) as a `values.yaml` file for the following chart:

   - Repository: `moekai`
   - Chart: `borgmatic`

4. [Prepare the values file](helm.md#update-helm-values) with the following configuration considerations:

   - `borgmatic.global.sshPrivateKey`: Set the value to the full content of the SSH private key for this Borg client

      ```yaml
      borgmatic:
        global:
          sshPrivateKey: |
            -----BEGIN OPENSSH PRIVATE KEY-----
            <private-key-content>
            -----END OPENSSH PRIVATE KEY-----
      ```

   - `storage.data.enabled`: Set the value to `true` to enable persistent storage for the client's cache and state

      ```yaml
      storage:
        data:
          enabled: true
      ```

   - **(Optional)** `borgmatic.timezone`: Set the timezone used for the CronJob schedule

      ```yaml
      borgmatic:
        timezone: "Asia/Kuala_Lumpur"
      ```

   - **(Optional)** `resources.borgmatic.requests` and `resources.borgmatic.limits`: Update the resource requests and limits for the Borg client container according to your cluster resources

      ```yaml
      resources:
        borgmatic:
          requests:
            cpu: "50m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
      ```

   - For each backup job you wish to create, add an entry under `borgmatic.configs` with a unique `<config-name>` identifier (i.e. `myconfig`), which forms part of the resulting CronJob name (i.e. `<release>-borgmatic-<config-name>`). Repeat the following for each job:

     - `borgmatic.configs.<config-name>.schedule`: Set a cron expression for the backup schedule (i.e. `"0 2 * * *"` for daily at 2 AM)

        ```yaml
        borgmatic:
          configs:
            <config-name>:
              schedule: "0 2 * * *"
        ```

     - `borgmatic.configs.<config-name>.passphrase`: Set a secure passphrase for the Borg repository

        ```yaml
        borgmatic:
          configs:
            <config-name>:
              passphrase: "your-secure-passphrase"
        ```

        The chart injects this value automatically as the `BORG_PASSPHRASE` environment variable. Do not include the passphrase in the `content` field.

     - **(Optional)** `borgmatic.configs.<config-name>.secrets`: Define secret environment variables that will be available within the configuration content via `${ENV_VAR}` interpolation

        ```yaml
        borgmatic:
          configs:
            <config-name>:
              secrets:
                MYSQL_ROOT_PASSWORD: "<mysql-root-password>"
                MARIADB_ROOT_PASSWORD: "<mariadb-root-password>"
                PG_PASSWORD: "<pg-password>"
                APPRISE_EMAIL: "<email-address>"
                APPRISE_PASSWORD: "<email-password>"
        ```

     - **(Optional)** `borgmatic.configs.<config-name>.volumes`: Mount existing PVCs into the container for filesystem backups. Each entry requires a `claimName` and a `mountPath`

        ```yaml
        borgmatic:
          configs:
            <config-name>:
              volumes:
                - claimName: "app-data-pvc"
                  mountPath: "/mnt/app/data"
                - claimName: "app-logs-pvc"
                  mountPath: "/mnt/app/logs"
                  subPath: "data"
        ```

        PVCs are mounted read-only and must be in the same namespace as the Borg client. They must also support concurrent read access, so use `ReadWriteMany` (RWX) where possible, as `ReadWriteOnce` (RWO) only allows a single node to mount the PVC at a time (if this even works).

     - `borgmatic.configs.<config-name>.content`: Set the raw backup configuration content for this job

        ```yaml
        borgmatic:
          configs:
            <config-name>:
              content: |
                ssh_command: ssh -i /root/.ssh/id -o StrictHostKeyChecking=accept-new

                repositories:
                  - path: ssh://borg@<borg-server-host>:<ssh-port>/backup/<client-name>
                    label: <repo-label>
                    encryption: repokey-blake2

                source_directories:
                  - /mnt/app/data
                  - /mnt/app/logs

                exclude_patterns:
                  - /mnt/app/data/tmp

                mysql_databases:
                  - name: all
                    hostname: mysql
                    port: 3306
                    username: root
                    password: ${MYSQL_ROOT_PASSWORD}
                    options: "--single-transaction --quick"

                mariadb_databases:
                  - name: all
                    hostname: mariadb
                    port: 3306
                    username: root
                    password: ${MARIADB_ROOT_PASSWORD}

                postgresql_databases:
                  - name: all
                    hostname: postgres
                    port: 5432
                    username: postgres
                    password: ${PG_PASSWORD}

                compression: lz4

                keep_daily: 7
                keep_weekly: 4
                keep_monthly: 6

                checks:
                  - name: repository
                    frequency: 2 weeks
                  - name: archives
                    frequency: 4 weeks

                apprise:
                  services:
                    - label: email
                      url: mailtos://<smtp-host>:<smtp-port>?user=${APPRISE_EMAIL}&pass=${APPRISE_PASSWORD}&from=${APPRISE_EMAIL}&to=${APPRISE_EMAIL}
                  send_logs: true
                  states:
                    - finish
                    - fail
                  finish:
                    title: Borgmatic backup finished
                    body: Backup completed successfully.
                  fail:
                    title: Borgmatic backup FAILED
                    body: Backup failed. Check logs.
        ```

        The SSH private key is always mounted at `/root/.ssh/id` inside the container regardless of its original filename. Use `content: |` (literal block scalar) to preserve the YAML structure of the backup configuration.

        Update the configuration above to match your actual backup setup.

5. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you had prepared with the following recommended options:

   - Namespace: Deploy into the same namespace as any PVCs you plan to mount, if any, as PVCs cannot be shared across namespaces (include the flag that creates the namespace if it does not exist)
   - Release: A descriptive name for this Borg client deployment (i.e. `borgmatic-client-1`)
   - Repository: `moekai`
   - Chart: `borgmatic`

6. Go through the [post-installation setup steps](#borg-client-post-install-setup-on-helm) to complete the Borg client setup.

7. **(Optional)** Keep the Helm release values file (i.e. `values.yaml`) for future use (i.e. for updates).

### Borg Client Post-Install Setup

#### Borg Client Post-Install Setup on Docker

> [!NOTE]  
> This guide assumes that you have [installed the Borg client](#install-borg-client-on-docker) as a containerised application.

This details the post-installation steps of the Borg client for a complete setup:

1. Before the first scheduled backup can run, initialise the Borg repository on the remote server:

   - If the client's target repository path is nested more than one level under `/backup` (i.e. `/backup/<client-name>/repo` rather than just `/backup/<client-name>`), pre-create the parent directory on the Borg server host:

      ```sh
      mkdir -p <repo-dir>/<client-name>
      ```

      Repository initialisation creates the final repository directory itself, but requires its parent directory to already exist.

   - [Run the repository initialisation](../courses/container.md#container-runtime-usage) from inside the client container:

      ```sh
      docker exec -it <container-name> borgmatic --config /etc/borgmatic.d/<config-name>.yaml repo-create
      ```

      For example:

      ```sh
      docker exec -it borg-client borgmatic --config /etc/borgmatic.d/config.yaml repo-create
      ```

      The passphrase is automatically sourced from the `BORG_PASSPHRASE` environment variable set in the env file. Ensure it is stored securely (i.e. in a password manager), as the repository cannot be decrypted or recovered without it.

2. **(Optional)** [Run a manual backup](../courses/container.md#container-runtime-usage) from inside the client container to verify that the setup is working correctly:

   ```sh
   docker exec -it <container-name> borgmatic --config /etc/borgmatic.d/<config-name>.yaml --verbosity 1
   ```

   For example:

   ```sh
   docker exec -it borg-client borgmatic --config /etc/borgmatic.d/config.yaml --verbosity 1
   ```

   Once the backup completes successfully, the container will continue running and trigger subsequent backups automatically according to the configured `CRON_SCHEDULE`.

#### Borg Client Post-Install Setup on Helm

> [!NOTE]  
> This guide assumes that you have [installed the Borg client](#install-borg-client-on-helm) on a Kubernetes cluster using Helm.

This details the post-installation steps of the Borg client for a complete setup:

1. Before the first scheduled backup can run, initialise the Borg repository using the shell CronJob provided by the chart:

   - Create a one-off job from the suspended shell CronJob, replacing `<namespace>`, `<release>`, and `<config-name>` with your Helm release namespace, release name, and backup config name respectively:

      ```sh
      kubectl create job --namespace <namespace> borgmatic-shell \
        --from=cronjob/<release>-borgmatic-<config-name>-shell
      ```

      For example:

      ```sh
      kubectl create job --namespace borgmatic borgmatic-shell \
        --from=cronjob/borgmatic-client-1-borgmatic-myconfig-shell
      ```

   - Wait for the pod to start, then get the pod name:

      ```sh
      kubectl get pod --namespace <namespace> -l job-name=borgmatic-shell
      ```

   - Exec into the pod and initialise the repository:

      ```sh
      kubectl exec --namespace <namespace> -it <pod-name> -- \
        borgmatic init --encryption repokey-blake2 \
        --config /etc/borgmatic.d/<config-name>.yaml
      ```

      For example:

      ```sh
      kubectl exec --namespace borgmatic -it borgmatic-shell-xxxxx -- \
        borgmatic init --encryption repokey-blake2 \
        --config /etc/borgmatic.d/myconfig.yaml
      ```

   - Exit the pod and delete the shell job:

      ```sh
      kubectl delete job --namespace <namespace> borgmatic-shell
      ```

2. **(Optional)** Trigger a manual backup to verify that the setup is working:

   - Create a manual backup job, replacing `<namespace>`, `<release>`, and `<config-name>` accordingly:

      ```sh
      kubectl create job --namespace <namespace> <release>-borgmatic-<config-name>-manual \
        --from=cronjob/<release>-borgmatic-<config-name>
      ```

   - Follow the job's logs:

      ```sh
      kubectl logs --namespace <namespace> \
        -l "app.kubernetes.io/instance=<release>" \
        --follow
      ```

   - Delete the manual job once done:

      ```sh
      kubectl delete job --namespace <namespace> <release>-borgmatic-<config-name>-manual
      ```

---

## Usage

### Description

This details some common usage steps for Borg backups.

### References

- [How to inspect your backups](https://torsion.org/borgmatic/how-to/inspect-your-backups/)
- [How to extract a backup](https://torsion.org/borgmatic/how-to/restore-a-backup/)

### Listing Archives

This details how to list the available archives in a Borg repository:

- To list archives on a **Docker** deployment:

   ```sh
   docker exec -it <container-name> borgmatic --config /etc/borgmatic.d/<config-name>.yaml list
   ```

   For example:

   ```sh
   docker exec -it borg-client borgmatic --config /etc/borgmatic.d/config.yaml list
   ```

- To list archives on a **Helm** deployment, create and exec into a shell job as described in [Borg Client Post-Install Setup on Helm](#borg-client-post-install-setup-on-helm), then run:

   ```sh
   borgmatic --config /etc/borgmatic.d/<config-name>.yaml list --match-archives "*"
   ```

   The `--match-archives "*"` flag is required to also include archives created by manually-triggered jobs.

### Restoring from Backup

This details how to restore data from a Borg backup.

Replace `<archive-name>` in any of the commands below with a specific archive obtained from [listing archives](#listing-archives) to restore from a specific point in time, or use `latest` to restore from the most recent backup.

- To restore files on a **Docker** deployment:

   ```sh
   docker exec -it <container-name> borgmatic --config /etc/borgmatic.d/<config-name>.yaml extract \
     --archive <archive-name> \
     --destination <destination-path>
   ```

   For example, to restore the latest archive to `/tmp/restore`:

   ```sh
   docker exec -it borg-client borgmatic --config /etc/borgmatic.d/config.yaml extract \
     --archive latest \
     --destination /tmp/restore
   ```

- To restore database dumps on a **Docker** deployment:

   ```sh
   docker exec -it <container-name> borgmatic --config /etc/borgmatic.d/<config-name>.yaml restore \
     --archive <archive-name>
   ```

   For example:

   ```sh
   docker exec -it borg-client borgmatic --config /etc/borgmatic.d/config.yaml restore \
     --archive latest
   ```

- To restore files on a **Helm** deployment, create and exec into a shell job as described in [Borg Client Post-Install Setup on Helm](#borg-client-post-install-setup-on-helm), then run:

   ```sh
   borgmatic --config /etc/borgmatic.d/<config-name>.yaml extract \
     --archive <archive-name> \
     --destination <destination-path>
   ```

   For example, to restore the latest archive to `/tmp/restore`:

   ```sh
   borgmatic --config /etc/borgmatic.d/myconfig.yaml extract \
     --archive latest \
     --destination /tmp/restore
   ```

- To restore database dumps on a **Helm** deployment, create and exec into a shell job as described in [Borg Client Post-Install Setup on Helm](#borg-client-post-install-setup-on-helm), then run:

   ```sh
   borgmatic --config /etc/borgmatic.d/<config-name>.yaml restore --archive <archive-name>
   ```

   For example:

   ```sh
   borgmatic --config /etc/borgmatic.d/myconfig.yaml restore --archive latest
   ```
