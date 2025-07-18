# Immich

## Description

Immich is a high performance self-hosted photo and video management solution.

## Directory

- [Immich](#immich)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Post-Install Setup](#post-install-setup)
    - [Configuration](#configuration)
  - [Remote Machine Learning](#remote-machine-learning)
    - [Description](#description-2)
    - [References](#references-2)
    - [Remote ML Setup](#remote-ml-setup)
  - [Usage](#usage)
    - [Description](#description-3)
    - [Configuring Immich](#configuring-immich)

## References

- [Immich](https://immich.app)
- [GitHub](https://github.com/immich-app/immich)

---

## Setup

### Description

This details how to install and set up an Immich server in a containerised environment.

### References

- [Docker Compose [Recommended]](https://immich.app/docs/install/docker-compose)
- [Latest Releases](https://github.com/immich-app/immich/releases/latest)
- [Hardware Transcoding](https://immich.app/docs/features/hardware-transcoding)
- [Hardware-Accelerated Machine Learning](https://immich.app/docs/features/ml-hardware-acceleration)
- [Post installation steps](https://immich.app/docs/install/post-install)
- [Fixing faces](https://www.reddit.com/r/immich/comments/1i0ka3i/fixing_faces)
- [Reverse Proxy](https://immich.app/docs/administration/reverse-proxy)

### Installation

This details the installation steps for Immich as a Docker container:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [Docker is installed and set up](../courses/container.md#setting-up-docker). The following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the system or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule): `2283/tcp`

2. [Deploy the Immich stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/immich`): This will be used for at least the database volume directory which specifically requires local storage.

   - **(Optional)** A remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/immich`): This can be used for anything else that supports remote storage such as the Immich stack's deployment files and volume(s).

   - **(Optional)** A remote app directory, on remote mounted storage (i.e. `/mnt/smb/media/photos/immich`): This will be used for the large media files that will be uploaded to Immich.

   - A Docker compose file for the Immich stack on the app directory (i.e. `/mnt/smb/docker/immich/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        immich-server:
          container_name: ${IMMICH_CONTAINER}
          image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
          ports:
            - 2283:2283
          env_file:
            - .env
          #extends:
          #  file: hwaccel.transcoding.yml
          #  service: ${HWACCEL_TRANS_SVC}
          volumes:
            - ${UPLOAD_LOCATION}:/usr/src/app/upload
            - /etc/localtime:/etc/localtime:ro
          depends_on:
            - redis
            - database
          healthcheck:
            disable: false
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

        immich-machine-learning:
          container_name: ${IMMICH_ML_CONTAINER}
          # For hardware acceleration, add one of -[armnn, cuda, rocm, openvino, rknn] to the image tag.
          # Example tag: ${IMMICH_VERSION:-release}-cuda
          image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}${HWACCEL_ML_SVC:+-${HWACCEL_ML_SVC}}
          env_file:
            - .env
          #extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
          #  file: hwaccel.ml.yml
          #  service: ${HWACCEL_ML_SVC}${HWACCEL_ML_WSL:+-${HWACCEL_ML_WSL}}
          volumes:
            - ${ML_CACHE_LOCATION}:/cache
          healthcheck:
            disable: false
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

        redis:
          container_name: ${REDIS_CONTAINER}
          image: docker.io/valkey/valkey:${REDIS_VERSION}
          healthcheck:
            test: redis-cli ping || exit 1
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

        database:
          container_name: ${POSTGRES_CONTAINER}
          image: ghcr.io/immich-app/postgres:${POSTGRES_VERSION}
          environment:
            POSTGRES_PASSWORD: ${DB_PASSWORD}
            POSTGRES_USER: ${DB_USERNAME}
            POSTGRES_DB: ${DB_DATABASE_NAME}
            POSTGRES_INITDB_ARGS: '--data-checksums'
            # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
            #DB_STORAGE_TYPE: 'HDD'
          volumes:
            # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
            - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

      **(Optional)** If your host deployment environment is capable of hardware transcoding, uncomment the following section in the Docker compose file:

      ```diff
        immich-server:
          container_name: ${IMMICH_CONTAINER}
          ...
      -   #extends:
      -   #  file: hwaccel.transcoding.yml
      -   #  service: ${HWACCEL_TRANS_SVC}
      +   extends:
      +     file: hwaccel.transcoding.yml
      +     service: ${HWACCEL_TRANS_SVC}
          ...
      ```

      **(Optional)** If your host deployment environment is capable of hardware-accelerated machine learning, uncomment the following section in the Docker compose file:

      ```diff
        immich-machine-learning:
          container_name: ${IMMICH_ML_CONTAINER}
          ...
      -   #extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
      -   #  file: hwaccel.ml.yml
      -   #  service: ${HWACCEL_ML_SVC}${HWACCEL_ML_WSL:+-${HWACCEL_ML_WSL}}
      +   extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
      +     file: hwaccel.ml.yml
      +     service: ${HWACCEL_ML_SVC}${HWACCEL_ML_WSL:+-${HWACCEL_ML_WSL}}
          ...
      ```

      **(Optional)** If the local storage where the local app directory is located is a hard disk drive (HDD), uncomment the following line in the Docker compose file:

      ```diff
        database:
          container_name: ${POSTGRES_CONTAINER}
          ...
          environment:
            ...
            # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      -     #DB_STORAGE_TYPE: 'HDD'
      +     DB_STORAGE_TYPE: 'HDD'
          ...
      ```

   - An env file for the Immich stack on the app directory (i.e. `/mnt/smb/docker/immich/.env`):

      ```sh
      # The env variables below this line are specific to the homelab-wiki guide deployment
      ###################################################################################
      SERVICE_NAME=immich
      IMMICH_CONTAINER=immich_server
      IMMICH_ML_CONTAINER=immich_machine_learning
      REDIS_CONTAINER=immich_redis
      POSTGRES_CONTAINER=immich_postgres
      ML_CACHE_LOCATION=/mnt/smb/docker/immich/model-cache
      REDIS_VERSION=8-bookworm@sha256:fec42f399876eb6faf9e008570597741c87ff7662a54185593e74b09ce83d177
      POSTGRES_VERSION=14-vectorchord0.4.3-pgvectors0.2.0
      # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
      #HWACCEL_TRANS_SVC=quicksync
      # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference
      #HWACCEL_ML_SVC=openvino
      # set to 'wsl' if deploying on WSL2
      #HWACCEL_ML_WSL=wsl

      # The env variables below this line were provided by the Immich project
      # You can find documentation for all the supported env variables at https://immich.app/docs/install/environment-variables
      ###################################################################################
      # The location where your uploaded files are stored
      UPLOAD_LOCATION=/mnt/smb/media/photos/immich

      # The location where your database files are stored. Network shares are not supported for the database
      DB_DATA_LOCATION=${HOME}/.local/share/docker/immich/pgdata

      # To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
      # TZ=Etc/UTC

      # The Immich version to use. You can pin this to a specific version like "v1.71.0"
      IMMICH_VERSION=v1.135.3

      # Connection secret for postgres. You should change it to a random password
      # Please use only the characters `A-Za-z0-9`, without special characters or spaces
      DB_PASSWORD=changethispassword

      # The values below this line do not need to be changed
      ###################################################################################
      DB_USERNAME=postgres
      DB_DATABASE_NAME=immich
      ```

      **(Optional)** If your host deployment environment is capable of hardware transcoding, uncomment the following line in the env file:

      ```diff
        # The env variables below this line are specific to the homelab-wiki guide deployment
        ###################################################################################
        ...
        # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
      - #HWACCEL_TRANS_SVC=quicksync
      + HWACCEL_TRANS_SVC=quicksync
        ...
      ```

      **(Optional)** If your host deployment environment is capable of hardware-accelerated machine learning, uncomment the following line in the env file:

      ```diff
        # The env variables below this line are specific to the homelab-wiki guide deployment
        ###################################################################################
        ...
        # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference
      - #HWACCEL_ML_SVC=openvino
      + HWACCEL_ML_SVC=openvino
        ...
      ```

      **(Optional)** If your host deployment environment is capable of hardware-accelerated machine learning and is running on WSL2, uncomment the following line in the env file:

      ```diff
        # The env variables below this line are specific to the homelab-wiki guide deployment
        ###################################################################################
        ...
        # set to 'wsl' if deploying on WSL2
      - #HWACCEL_ML_WSL=wsl
      + HWACCEL_ML_WSL=wsl
        ...
      ```

      Replace all of the values in the env file with your own accordingly.

   - **(Optional)** A Docker compose file for hardware-accelerated transcoding on the app directory if your host deployment environment is capable of hardware transcoding (i.e. `/mnt/smb/docker/immich/hwaccel.transcoding.yml`):

      ```yaml
      # Configurations for hardware-accelerated transcoding

      # If using Unraid or another platform that doesn't allow multiple Compose files,
      # you can inline the config for a backend by copying its contents
      # into the immich-microservices service in the docker-compose.yml file.

      # See https://immich.app/docs/features/hardware-transcoding for more info on using hardware transcoding.

      services:
        cpu: {}

        nvenc:
          deploy:
            resources:
              reservations:
                devices:
                  - driver: nvidia
                    count: 1
                    capabilities:
                      - gpu
                      - compute
                      - video

        quicksync:
          devices:
            - /dev/dri:/dev/dri

        rkmpp:
          security_opt: # enables full access to /sys and /proc, still far better than privileged: true
            - systempaths=unconfined
            - apparmor=unconfined
          group_add:
            - video
          devices:
            - /dev/rga:/dev/rga
            - /dev/dri:/dev/dri
            - /dev/dma_heap:/dev/dma_heap
            - /dev/mpp_service:/dev/mpp_service
            #- /dev/mali0:/dev/mali0 # only required to enable OpenCL-accelerated HDR -> SDR tonemapping
          volumes:
            #- /etc/OpenCL:/etc/OpenCL:ro # only required to enable OpenCL-accelerated HDR -> SDR tonemapping
            #- /usr/lib/aarch64-linux-gnu/libmali.so.1:/usr/lib/aarch64-linux-gnu/libmali.so.1:ro # only required to enable OpenCL-accelerated HDR -> SDR tonemapping

        vaapi:
          devices:
            - /dev/dri:/dev/dri

        vaapi-wsl: # use this for VAAPI if you're running Immich in WSL2
          devices:
            - /dev/dri:/dev/dri
            - /dev/dxg:/dev/dxg
          volumes:
            - /usr/lib/wsl:/usr/lib/wsl
          environment:
            - LIBVA_DRIVER_NAME=d3d12
      ```

   - **(Optional)** A Docker compose file for hardware-accelerated machine learning on the app directory if your host deployment environment is capable of hardware-accelerated machine learning (i.e. `/mnt/smb/docker/immich/hwaccel.ml.yml`):

      ```yaml
      # Configurations for hardware-accelerated machine learning

      # If using Unraid or another platform that doesn't allow multiple Compose files,
      # you can inline the config for a backend by copying its contents
      # into the immich-machine-learning service in the docker-compose.yml file.

      # See https://immich.app/docs/features/ml-hardware-acceleration for info on usage.

      services:
        armnn:
          devices:
            - /dev/mali0:/dev/mali0
          volumes:
            - /lib/firmware/mali_csffw.bin:/lib/firmware/mali_csffw.bin:ro # Mali firmware for your chipset (not always required depending on the driver)
            - /usr/lib/libmali.so:/usr/lib/libmali.so:ro # Mali driver for your chipset (always required)

        rknn:
          security_opt:
            - systempaths=unconfined
            - apparmor=unconfined
          devices:
            - /dev/dri:/dev/dri

        cpu: {}

        cuda:
          deploy:
            resources:
              reservations:
                devices:
                  - driver: nvidia
                    count: 1
                    capabilities:
                      - gpu

        rocm:
          group_add:
            - video
          devices:
            - /dev/dri:/dev/dri
            - /dev/kfd:/dev/kfd

        openvino:
          device_cgroup_rules:
            - 'c 189:* rmw'
          devices:
            - /dev/dri:/dev/dri
          volumes:
            - /dev/bus/usb:/dev/bus/usb

        openvino-wsl:
          devices:
            - /dev/dri:/dev/dri
            - /dev/dxg:/dev/dxg
          volumes:
            - /dev/bus/usb:/dev/bus/usb
            - /usr/lib/wsl:/usr/lib/wsl
      ```

3. Go through the [post-installation setup steps](#post-install-setup) for the Immich server.

4. **(Optional)** If your host deployment environment for Immich is not capable of hardware-accelerated machine learning, but you have a separate deployment environment that is capable, you have the option to [set up a remote machine learning server](#remote-machine-learning) on said system as a companion to the main Immich server.

### Post-Install Setup

This details the post-installation steps of the Immich server for a complete setup:

1. Launch the Immich server web interface at `http://<immich-server-host>:2283` (i.e. `http://192.168.0.106:2283`) on a web browser.

2. For the initial launch of the Immich web interface, click the **Getting Started** button.

3. In the **Admin Registration** view, fill in the following information in the provided form:

   - Admin Email: Set the email address of the Immich admin user (i.e. `admin@example.com`)
   - Password: Set a secure password for the Immich admin user
   - Confirm password: Confirm the password you have set for the Immich admin user
   - Name: Specify the name of the Immich admin user (i.e. `Admin`)

   Submit the form by clicking the **Sign up** button.

4. In the **Login** page, fill in the admin user credentials, and click the **Login** button.

5. As part of the initial onboarding process, go through the process as following:

   - In the **Onboarding** page that is _welcoming_ you, click the **Theme** button to proceed.

   - In the **COLOR THEME** step, select your desired theme (i.e. `DARK`), then click the **Privacy** button to proceed.

   - In the **PRIVACY** step, set your desired privacy settings, then click the **Storage Template** button to proceed.

   - In the **STORAGE TEMPLATE** step, select to enable the **Enable storage template engine** option, then configure the following:

     - Preset: Expand the dropdown and select the desired preconfigured storage convention (i.e. `2022/02/IMAGE_56437`)
     - Template: Customise the template further or leave it as default according to your selected preset (i.e. `{{y}}/{{MM}}/{{filename}}`)

   - Click the **Done** button to complete the onboarding process.

6. Populate the Immich server with users that will be using the server to store their media files:

   - Launch the Immich server web interface at `http://<immich-server-host>:2283` (i.e. `http://192.168.0.106:2283`) on a web browser.

   - In the **Login** page, fill in the admin user credentials, and click the **Login** button.

   - Click on the user profile icon from the top right corner of the web interface, and click the **Administration** button.

   - By default, you should be taken to the **User Management** page - if not, click on the **Users** menu option found on the left sidebar of the web interface.

   - In the **User Management** view, click the **Create user** button found on the top right corner of the page, and configure the form as following:

     - Email: Set the email address of the Immich user (i.e. `admin@example.com`)
     - Send welcome email: Select to enable if you wish the user to receive a welcome email upon registration
     - Password: Set a secure password for the Immich user
     - Confirm password: Confirm the password you have set for the Immich user
     - Require user to change password on first login: Select to enable if you wish to force the user to change the pre-assigned password after their initial login
     - Name: Specify the name of the Immich user (i.e. `User`)
     - Quota Size (GiB): Set a storage usage limit for the user or leave as default (i.e. `Unlimited`)

      Click the **Create** button to submit the user registration form.

7. Configure the Immich server based on the [recommended configuration options](#configuration).

### Configuration

This details some recommended configuration options for an Immich setup:

1. [Configure the Immich server](#configuring-immich)'s facial recognition settings to better detect and differentiate faces:

   - From the **Machine Learning Settings**, expand the **Facial Recognition** section, and configure the following:

     - Enable facial recognition: Select to enable the facial recognition feature on the Immich server
     - Facial recognition model: Expand the dropdown and select the `buffalo_l` model
     - Minimum detection score: Set the intended minimum confidence score for face detection (i.e. `0.7`)
     - Maximum recognition distance: Set the intended maximum distance between two faces to be considered the same person (i.e. `0.4`)
     - Minimum recognized faces: Set the intended number of recognised faces for a person to be registered (i.e. `5`)

2. **(Optional)** [Configure Immich](#configuring-immich) to disable transcoding if the host deployment environment is not capable of hardware-accelerated transcoding:

   - From the **Video Transcoding Settings**, expand the **Transcode Policy** section, and configure the following:

     - Transcode Policy: Expand the dropdown and select the `Don't transcode any videos, may break playback on some clients` option

3. **(Optional)** If you wish to set up [reverse proxy](../courses/network.md#registering-subdomains) for the Immich server, you may need to include the following custom annotations to the Ingress resource:

    ```yaml
    customAnnotations:
      - prefix: ""
        name: "proxy-body-size"
        value: "50000M"
      - prefix: ""
        name: "proxy-connect-timeout"
        value: "600"
      - prefix: ""
        name: "proxy-read-timeout"
        value: "1800"
      - prefix: ""
        name: "proxy-send-timeout"
        value: "600"
      - prefix: "nginx.org"
        name: "send-timeout"
        value: "6000"
    ```

    This should help resolve some issues with uploading large files to the Immich server through its public domain.

---

## Remote Machine Learning

### Description

To alleviate performance issues on low-memory systems like the Raspberry Pi, you may also host Immich's machine learning (ML) container on a more powerful system, such as your laptop or desktop computer.

### References

- [Remote Machine Learning](https://immich.app/docs/guides/remote-machine-learning)

### Remote ML Setup

> [!NOTE]  
> This assumes that you have [set up an Immich server](#immich-server) on a host deployment environment that is not capable of hardware-accelerated machine learning.

This details how to set up a remote ML server for Immich if its host deployment environment is not capable of hardware-accelerated machine learning:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [Docker is installed and set up](../courses/container.md#setting-up-docker). The following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the system or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule): `3003/tcp`

2. [Deploy the Immich machine learning stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/immich-remote-ml`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/immich-remote-ml`): This will be used for the Immich ML stack's deployment files and volume(s).

   - A Docker compose file for the Immich ML stack on the app directory (i.e. `/mnt/smb/docker/immich-remote-ml/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        immich-machine-learning:
          container_name: ${IMMICH_ML_CONTAINER}
          # For hardware acceleration, add one of -[armnn, cuda, rocm, openvino, rknn] to the image tag.
          # Example tag: ${IMMICH_VERSION:-release}-cuda
          image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}${HWACCEL_ML_SVC:+-${HWACCEL_ML_SVC}}
          ports:
            - 3003:3003
          #extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
          #  file: hwaccel.ml.yml
          #  service: ${HWACCEL_ML_SVC}${HWACCEL_ML_WSL:+-${HWACCEL_ML_WSL}}
          volumes:
            - ${ML_CACHE_LOCATION}:/cache
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

      **(Optional)** If your host deployment environment is capable of hardware-accelerated machine learning, uncomment the following section in the Docker compose file:

      ```diff
        immich-machine-learning:
          container_name: ${IMMICH_ML_CONTAINER}
          ...
      -   #extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
      -   #  file: hwaccel.ml.yml
      -   #  service: ${HWACCEL_ML_SVC}${HWACCEL_ML_WSL:+-${HWACCEL_ML_WSL}}
      +   extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
      +     file: hwaccel.ml.yml
      +     service: ${HWACCEL_ML_SVC}${HWACCEL_ML_WSL:+-${HWACCEL_ML_WSL}}
          ...
      ```

   - An env file for the Immich ML stack on the app directory (i.e. `/mnt/smb/docker/immich-remote-ml/.env`):

      ```sh
      # The env variables below this line are specific to the homelab-wiki guide deployment
      ###################################################################################
      SERVICE_NAME=immich_remote_ml
      IMMICH_ML_CONTAINER=immich_machine_learning
      ML_CACHE_LOCATION=/mnt/smb/docker/immich-remote-ml/model-cache
      IMMICH_VERSION=v1.135.3
      # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference
      #HWACCEL_ML_SVC=openvino
      # set to 'wsl' if deploying on WSL2
      #HWACCEL_ML_WSL=wsl
      ```

      **(Optional)** If your host deployment environment is capable of hardware-accelerated machine learning, uncomment the following line in the env file:

      ```diff
        # The env variables below this line are specific to the homelab-wiki guide deployment
        ###################################################################################
        ...
        # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference
      - #HWACCEL_ML_SVC=openvino
      + HWACCEL_ML_SVC=openvino
        ...
      ```

      **(Optional)** If your host deployment environment is capable of hardware-accelerated machine learning and is running on WSL2, uncomment the following line in the env file:

      ```diff
        # The env variables below this line are specific to the homelab-wiki guide deployment
        ###################################################################################
        ...
        # set to 'wsl' if deploying on WSL2
      - #HWACCEL_ML_WSL=wsl
      + HWACCEL_ML_WSL=wsl
        ...
      ```

      Replace all of the values in the env file with your own accordingly.

   - **(Optional)** A Docker compose file for hardware-accelerated machine learning on the app directory if your host deployment environment is capable of hardware-accelerated machine learning (i.e. `/mnt/smb/docker/immich-remote-ml/hwaccel.ml.yml`):

      ```yaml
      # Configurations for hardware-accelerated machine learning

      # If using Unraid or another platform that doesn't allow multiple Compose files,
      # you can inline the config for a backend by copying its contents
      # into the immich-machine-learning service in the docker-compose.yml file.

      # See https://immich.app/docs/features/ml-hardware-acceleration for info on usage.

      services:
        armnn:
          devices:
            - /dev/mali0:/dev/mali0
          volumes:
            - /lib/firmware/mali_csffw.bin:/lib/firmware/mali_csffw.bin:ro # Mali firmware for your chipset (not always required depending on the driver)
            - /usr/lib/libmali.so:/usr/lib/libmali.so:ro # Mali driver for your chipset (always required)

        rknn:
          security_opt:
            - systempaths=unconfined
            - apparmor=unconfined
          devices:
            - /dev/dri:/dev/dri

        cpu: {}

        cuda:
          deploy:
            resources:
              reservations:
                devices:
                  - driver: nvidia
                    count: 1
                    capabilities:
                      - gpu

        rocm:
          group_add:
            - video
          devices:
            - /dev/dri:/dev/dri
            - /dev/kfd:/dev/kfd

        openvino:
          device_cgroup_rules:
            - 'c 189:* rmw'
          devices:
            - /dev/dri:/dev/dri
          volumes:
            - /dev/bus/usb:/dev/bus/usb

        openvino-wsl:
          devices:
            - /dev/dri:/dev/dri
            - /dev/dxg:/dev/dxg
          volumes:
            - /dev/bus/usb:/dev/bus/usb
            - /usr/lib/wsl:/usr/lib/wsl
      ```

3. [Configure the Immich server](#configuring-immich) to use the Immich ML server for machine learning tasks:

   - From the **Machine Learning Settings** section, configure the following:

     - Enable machine learning: Select to enable machine learning features on the Immich server
     - URL: Click the **Add URL** button and add the URL of the Immich ML server at `http://<immich-ml-server-host>:3003` (i.e. `http://192.168.0.106:3003`)

---

## Usage

### Description

This details some common usage steps for an Immich server.

### Configuring Immich

This details how to configure an Immich server through the web interface:

1. Launch the Immich server web interface at `http://<immich-server-host>:2283` (i.e. `http://192.168.0.106:2283`) on a web browser.

2. In the **Login** page, fill in the admin user credentials, and click the **Login** button.

3. Click on the user profile icon from the top right corner of the web interface, and click the **Administration** button.

4. Click the **Settings** menu option found on the left sidebar to navigate to the **System Settings** page.

5. In the **System Settings** page, expand any of the available sections, and configure them accordingly.

6. Save the configuration changes made by clicking each corresponding section's **Save** button.
