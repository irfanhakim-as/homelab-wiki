# SeaweedFS

## Description

SeaweedFS is a distributed object storage system with a built-in S3-compatible API gateway, suitable for self-hosting. It stores objects efficiently across master, volume, and filer components, and exposes them through an S3-compatible interface.

## Directory

- [SeaweedFS](#seaweedfs)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
      - [Install SeaweedFS on Docker](#install-seaweedfs-on-docker)
    - [Post-Install Setup](#post-install-setup)
      - [SeaweedFS Post-Install Setup on Docker](#seaweedfs-post-install-setup-on-docker)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [References](#references-2)
    - [Add S3 Identity](#add-s3-identity)
      - [Add S3 Identity on Docker](#add-s3-identity-on-docker)
  - [Usage](#usage)
    - [Description](#description-3)
    - [Provision an S3 Bucket](#provision-an-s3-bucket)

## References

- [SeaweedFS](https://github.com/seaweedfs/seaweedfs)

---

## Setup

### Description

This details how to deploy SeaweedFS in a containerised environment.

### References

- [SeaweedFS Docker](https://github.com/seaweedfs/seaweedfs/blob/master/docker/README.md)

### Installation

#### Install SeaweedFS on Docker

This details the installation steps for SeaweedFS as a containerised application using Docker:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime). The following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the host deployment environment or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule):

     - `8333/tcp` (or whichever port you intend to configure as the S3 API port)

2. [Deploy the SeaweedFS stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - An app directory, on local or remote storage (i.e. `/home/myuser/.local/share/docker/seaweedfs`): This will be used for the master server and filer metadata, as well as the S3 credentials configuration.

   - A volume directory, on local or remote storage (i.e. `/mnt/storage/seaweedfs-data`): This will be used to store the actual object data. This should point to a dedicated storage location with sufficient capacity.

   - Set up the required subdirectories in the app directory:

     - Pre-create the required subdirectories inside the app directory (i.e. `/home/myuser/.local/share/docker/seaweedfs`):

       ```sh
       mkdir -p <app-dir>/{master,filer,config}
       ```

       For example:

       ```sh
       mkdir -p /home/myuser/.local/share/docker/seaweedfs/{master,filer,config}
       ```

   - A Compose file for the SeaweedFS stack on the app directory (i.e. `/home/myuser/.local/share/docker/seaweedfs/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        seaweedfs-master:
          container_name: ${APP_CONTAINER}-master
          image: docker.io/chrislusf/seaweedfs:${APP_VERSION}
          command: master -ip=seaweedfs-master -ip.bind=0.0.0.0 -defaultReplication=000 -metricsPort=9324
          ports:
            - "${MASTER_PORT}:9333"
            - "${MASTER_GRPC_PORT}:19333"
          volumes:
            - ${APP_DIR}/master:/data
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true
          deploy:
            resources:
              limits:
                cpus: ${MASTER_CPU_LIMIT}
                memory: ${MASTER_MEMORY_LIMIT}

        seaweedfs-volume:
          container_name: ${APP_CONTAINER}-volume
          image: docker.io/chrislusf/seaweedfs:${APP_VERSION}
          command: volume -ip=seaweedfs-volume -mserver=seaweedfs-master:9333 -ip.bind=0.0.0.0 -port=8080 -dir=/data -metricsPort=9325
          depends_on:
            - seaweedfs-master
          ports:
            - "${VOLUME_PORT}:8080"
            - "${VOLUME_GRPC_PORT}:18080"
          volumes:
            - ${VOLUME_DIR}:/data
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true
          deploy:
            resources:
              limits:
                cpus: ${VOLUME_CPU_LIMIT}
                memory: ${VOLUME_MEMORY_LIMIT}

        seaweedfs-filer:
          container_name: ${APP_CONTAINER}-filer
          image: docker.io/chrislusf/seaweedfs:${APP_VERSION}
          command: filer -ip=seaweedfs-filer -master=seaweedfs-master:9333 -ip.bind=0.0.0.0 -metricsPort=9326
          depends_on:
            - seaweedfs-master
            - seaweedfs-volume
          ports:
            - "${FILER_PORT}:8888"
            - "${FILER_GRPC_PORT}:18888"
          volumes:
            - ${APP_DIR}/filer:/data
          networks:
            - default
          restart: unless-stopped
          tty: true
          stdin_open: true
          security_opt:
            - no-new-privileges:true
          deploy:
            resources:
              limits:
                cpus: ${FILER_CPU_LIMIT}
                memory: ${FILER_MEMORY_LIMIT}

        seaweedfs-s3:
          container_name: ${APP_CONTAINER}-s3
          image: docker.io/chrislusf/seaweedfs:${APP_VERSION}
          command: s3 -filer=seaweedfs-filer:8888 -ip.bind=0.0.0.0 -port=8333 -config=/etc/seaweedfs/s3.json -allowDeleteBucketNotEmpty=false -metricsPort=9327
          depends_on:
            - seaweedfs-master
            - seaweedfs-volume
            - seaweedfs-filer
          ports:
            - "${S3_PORT}:8333"
          volumes:
            - ${APP_DIR}/config/s3.json:/etc/seaweedfs/s3.json:ro
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true
          deploy:
            resources:
              limits:
                cpus: ${S3_CPU_LIMIT}
                memory: ${S3_MEMORY_LIMIT}

      networks:
        default:
      ```

   - An env file for the SeaweedFS stack on the app directory (i.e. `/home/myuser/.local/share/docker/seaweedfs/.env`):

      ```sh
      SERVICE_NAME=seaweedfs
      APP_CONTAINER=seaweedfs
      APP_VERSION=4.22
      APP_DIR=/home/myuser/.local/share/docker/seaweedfs
      VOLUME_DIR=/mnt/storage/seaweedfs-data
      MASTER_PORT=9333
      MASTER_GRPC_PORT=19333
      VOLUME_PORT=8080
      VOLUME_GRPC_PORT=18080
      FILER_PORT=8888
      FILER_GRPC_PORT=18888
      S3_PORT=8333
      MASTER_CPU_LIMIT=0
      MASTER_MEMORY_LIMIT=512m
      VOLUME_CPU_LIMIT=0
      VOLUME_MEMORY_LIMIT=512m
      FILER_CPU_LIMIT=0
      FILER_MEMORY_LIMIT=1g
      S3_CPU_LIMIT=0
      S3_MEMORY_LIMIT=2g
      ```

      Replace all of the values in the env file with your own accordingly.

   - An S3 credentials configuration file on the app directory (i.e. `/home/myuser/.local/share/docker/seaweedfs/config/s3.json`): [Add at least one S3 identity](#add-s3-identity-on-docker) to this file before deploying the stack.

3. Go through the [post-installation setup steps](#seaweedfs-post-install-setup-on-docker) to complete the SeaweedFS setup.

### Post-Install Setup

#### SeaweedFS Post-Install Setup on Docker

> [!NOTE]
> This guide assumes that you have [installed SeaweedFS](#install-seaweedfs-on-docker) as a containerised application.

This details the post-installation steps of SeaweedFS for a complete setup:

1. [Verify that all four containers are running](../courses/container.md#container-runtime-usage). Sample output:

   ```
   CONTAINER ID   IMAGE                          ...   STATUS        NAMES
   a1b2c3d4e5f6   chrislusf/seaweedfs:4.22       ...   Up 2 minutes  seaweedfs-s3
   b2c3d4e5f6a7   chrislusf/seaweedfs:4.22       ...   Up 2 minutes  seaweedfs-filer
   c3d4e5f6a7b8   chrislusf/seaweedfs:4.22       ...   Up 2 minutes  seaweedfs-volume
   d4e5f6a7b8c9   chrislusf/seaweedfs:4.22       ...   Up 2 minutes  seaweedfs-master
   ```

2. Test that the S3 gateway is responding on the host:

   ```sh
   curl http://localhost:<s3-port>
   ```

   For example:

   ```sh
   curl http://localhost:8333
   ```

   Sample output:

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Error><Code>AccessDenied</Code><Message>Access Denied.</Message><Resource>/</Resource><RequestId>7D4A1C8F3B2E950A6F1D8C4B</RequestId></Error>
   ```

   An XML response, whether a bucket list or an access error, confirms the S3 gateway is running and accepting requests.

3. Set up the initial S3 identities and buckets:

   - [Add an S3 identity](#add-s3-identity-on-docker) for an admin user. The following action grants full access to all S3 operations:

     - `Admin`

   - For every bucket required, [create the named bucket](s3.md#create-a-bucket) using the admin identity.

   - For every client (i.e. application or user) that needs access to the S3 gateway, [add an S3 identity](#add-s3-identity-on-docker) with credentials and actions scoped to their intended bucket. The following actions are recommended for a client with full read and write access to a specific bucket:

     - `Read:<bucket-name>/*`
     - `Write:<bucket-name>/*`
     - `List:<bucket-name>`
     - `Tagging:<bucket-name>/*`
     - `Admin:<bucket-name>` (if the client requires bucket-level access such as `HeadBucket` to verify the bucket exists before operating)

4. **(Optional)** [Set up a reverse proxy](../courses/network.md#reverse-proxy) for the SeaweedFS S3 endpoint to make it accessible over HTTPS outside the local network. This is the recommended approach for external access. The following considerations should be noted:

   - Your [SeaweedFS domain](../courses/network.md#registering-subdomains) should point directly to your public IP, not proxied through Cloudflare or similar services (i.e. DNS-only mode), as proxying can interfere with S3 request signature verification.
   - Configure the proxy host to forward to the host's LAN IP (i.e. `192.168.0.106`) and the S3 API port (i.e. `8333`).

---

## Configuration

### Description

This details how to configure SeaweedFS after it has been deployed.

### References

- [S3 Credentials](https://github.com/seaweedfs/seaweedfs/wiki/S3-Credentials)
- [SeaweedFS S3 credentials example](https://github.com/seaweedfs/seaweedfs/blob/master/docker/compose/s3.json)

### Add S3 Identity

#### Add S3 Identity on Docker

> [!NOTE]
> This guide assumes that you have [installed SeaweedFS](#install-seaweedfs-on-docker) as a containerised application.

This details how to add a new S3 identity to a SeaweedFS Docker deployment:

1. Generate a new access key and secret key for the identity:

   ```sh
   openssl rand -hex 32
   ```

   Run the command twice, once for the access key and once for the secret key.

2. Add a new entry to the `identities` list in the S3 credentials configuration file (i.e. `/home/myuser/.local/share/docker/seaweedfs/config/s3.json`):

   ```diff
     {
       "identities": [
   +     {
   +       "name": "<identity-name>",
   +       "credentials": [
   +         {
   +           "accessKey": "<access-key>",
   +           "secretKey": "<secret-key>"
   +         }
   +       ],
   +       "actions": [
   +         "<action>"
   +       ]
   +     },
         ...
       ]
     }
   ```

   For example, to add a `backup-client` identity with access scoped to a bucket named `my-backups`:

   ```diff
     {
       "identities": [
   +     {
   +       "name": "backup-client",
   +       "credentials": [
   +         {
   +           "accessKey": "c9a1f4d23e87b605a7c2e94f183d6a52",
   +           "secretKey": "8e3b7d6c1a90f452b38e1c4d7a5f2e09"
   +         }
   +       ],
   +       "actions": [
   +         "Read:my-backups/*",
   +         "List:my-backups",
   +         "Tagging:my-backups/*",
   +         "Write:my-backups/*"
   +       ]
   +     },
         ...
       ]
     }
   ```

   Actions can be applied globally or scoped to a specific bucket. The scoping format depends on whether the action targets the bucket itself or objects inside it:

   - **Object-level actions** are scoped with `<action>:<bucket-name>/*`, where `/*` refers to all objects within the bucket:

     - `Read:<bucket-name>/*`: Download objects.
     - `Write:<bucket-name>/*`: Upload and delete objects.
     - `Tagging:<bucket-name>/*`: Read and manage object tags.
     - `Read_ACP:<bucket-name>/*`: Read access control policies for objects and buckets.
     - `Write_ACP:<bucket-name>/*`: Write access control policies for objects and buckets.

   - **Bucket-level actions** are scoped with `<action>:<bucket-name>`, as they operate on the bucket as a whole rather than on individual objects:

     - `List:<bucket-name>`: List objects within a bucket.
     - `Admin:<bucket-name>`: Full access to all S3 operations globally. When scoped to a bucket, limits access to management of that specific bucket only (e.g. configuring ownership controls and ACLs), without granting object-level access.

3. [Restart](../courses/container.md#container-runtime-usage) the `seaweedfs-s3` container service for the changes to take effect.

---

## Usage

### Description

This details some common usage steps for SeaweedFS.

### Provision an S3 Bucket

> [!NOTE]
> This guide assumes that you have [installed and set up SeaweedFS](#setup) as a containerised application.

This details how to provision a complete S3 bucket with dedicated access credentials on SeaweedFS:

1. First and foremost, ensure the following pre-requisites are met:

   - [AWS CLI is installed](s3.md#install-aws-cli) on the client system.
   - An [AWS CLI profile](s3.md#configure-aws-cli-profile) has been configured for an S3 identity with admin access (e.g. `Admin`) to the SeaweedFS S3 API, for use in creating the bucket.

2. [Create the bucket](s3.md#create-a-bucket) using the admin AWS CLI profile.

3. [Add an S3 identity](#add-s3-identity-on-docker) scoped to the bucket to generate the access and secret keys. The following actions are recommended for a client with full read and write access to the bucket:

   - `Read:<bucket-name>/*`
   - `Write:<bucket-name>/*`
   - `List:<bucket-name>`
   - `Tagging:<bucket-name>/*`
   - **(Optional)** `Admin:<bucket-name>` (if the client requires bucket-level access such as `HeadBucket` to verify the bucket exists before operating)
