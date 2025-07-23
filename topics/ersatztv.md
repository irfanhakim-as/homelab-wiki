# ErsatzTV

## Description

ErsatzTV is beta software for configuring and streaming custom live channels using your media library.

## Directory

- [ErsatzTV](#ersatztv)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Installation](#installation)
    - [Post-Install Setup](#post-install-setup)

## References

- [ErsatzTV](https://ersatztv.org)
- [GitHub](https://github.com/ErsatzTV/ErsatzTV)

---

## Setup

### Description

This details how to install and set up an ErsatzTV server in a containerised environment.

### References

- [Install ErsatzTV](https://ersatztv.org/docs/user-guide/install)
- [Add Media Items](https://ersatztv.org/docs/user-guide/add-media-items)
- [Streaming Mode](https://ersatztv.org/docs/user-guide/create-channels#streaming-mode)

### Installation

This details the installation steps for ErsatzTV as a Docker container:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [Docker is installed and set up](../courses/container.md#setting-up-docker). The following considerations should be noted:

   - Hardware-accelerated transcoding is highly recommended for the best ErsatzTV experience - setting up an [LXC Container on Proxmox](../courses/container.md#create-lxc-container) and [passing through a (dedicated or integrated) GPU](../courses/hypervisor.md#hardware-passthrough) to it may be the easiest way to set up as the host deployment environment for ErsatzTV, paying careful attention to the following items in your deployment:

     - When deploying with Docker Compose, besides passing through a (dedicated or integrated) GPU to the host deployment environment (i.e. LXC Container), the graphics device should also be passed through to the ErsatzTV container - this is covered in the subsequent deployment steps.
     - Another thing you are likely going to do in your deployment is to pass through a remote or network storage directories to the ErsatzTV container for serving media files - this could be an SMB share that you mount on a Proxmox node host, then passed through to the LXC Container (before again passing through media directories from it to the ErsatzTV container).
     - In both aforementioned cases, it is **important** to ensure that the user the ErsatzTV container will run as has the required permissions to access the graphics device and directories that you are passing through - you might think using `group_add` in the Docker compose file to solve any permission issue (i.e. by adding the ErsatzTV container user to the same group GIDs as the graphics device and media directories) would be the logical solution, but based on our testing however, it might not be (at least not with an "LXC + Docker Container" combo). To verify that this is not an issue:

       - When [passing through the graphics device](../courses/hypervisor.md#hardware-passthrough) to the host deployment environment (i.e. LXC Container on Proxmox), ensure that it is passed through to the UID of the intended ErsatzTV container user (i.e. `0`). Verify that said UID has the rights to the graphics device on the host deployment environment:

          ```sh
          stat -c '%u' /dev/dri/*
          ```

          Sample UID output (i.e. `0`):

          ```
            0
            0
          ```

       - When [passing through a remote or network storage (i.e. SMB share) to the host deployment environment (i.e. LXC Container, from the Proxmox node host)](../courses/container.md#mount-smb-share-on-lxc-container) - which will later be passed through to the ErsatzTV container, ensure that the intended ErsatzTV container user (i.e. `0`) has the rights to the mounted storage on the host deployment environment (i.e. `/mnt/smb`):

          ```sh
          stat -c '%u:%g' <mountpoint>
          ```

          For example:

          ```sh
          stat -c '%u:%g' /mnt/smb
          ```

          Sample UID (i.e. `0`) and GID (i.e. `10000`) output:

          ```
            0:10000
          ```

          Ensure that at least either of these values matches the UID or GID of the intended ErsatzTV container user (i.e. `0`).

   - Either [disable the firewall](firewall.md#disablement) on the host deployment environment or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule):

     - `8409/tcp`

2. [Deploy the ErsatzTV stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/ersatztv`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/ersatztv`): This will be used for the ErsatzTV stack's volume(s).

   - **(Optional)** Remote media directories, on remote mounted storage (i.e. `/mnt/smb/media/movies`, `/mnt/smb/media/tvshows`, etc.): These directories are expected to contain the media files that are meant to be served in ErsatzTV.

   - A Docker compose file for the ErsatzTV stack on the app directory (i.e. `/mnt/smb/docker/ersatztv/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        ersatztv:
          container_name: ${APP_CONTAINER}
          image: docker.io/jasongdove/ersatztv:${APP_VERSION}
          ports:
            - 8409:8409
          #group_add:
          #  - '104' # This needs to be the group id of running `stat -c '%g' /dev/dri/renderD128` on the docker host
          environment:
            - TZ=${APP_TIMEZONE}
          volumes:
            - ${APP_DIR}/config:/config
            #- /path/to/movies:/mnt/movies:ro
            #- /path/to/tvshows:/mnt/tvshows:ro
            #- /path/to/anime:/mnt/anime:ro
            #- /path/to/videos:/mnt/videos:ro
            #- /path/to/music:/mnt/music:ro
          #tmpfs:
          #  - /transcode
          #devices:
          #  - /dev/dri:/dev/dri
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

      Update the following section in the Docker compose file with the actual paths to the directories that store your media files (i.e. Movies, TV Shows, Anime, etc.) - if you are deploying ErsatzTV alongside [Jellyfin](jellyfin.md), mapping the media directories to the same paths as they were mapped in Jellyfin is recommended:

      ```diff
        ersatztv:
          container_name: ${APP_CONTAINER}
          ...
          volumes:
            - ${APP_DIR}/config:/config
      -     #- /path/to/movies:/mnt/movies:ro
      -     #- /path/to/tvshows:/mnt/tvshows:ro
      -     #- /path/to/anime:/mnt/anime:ro
      +     - /mnt/smb/media/movies:/mnt/movies:ro
      +     - /mnt/smb/media/tvshows:/mnt/tvshows:ro
      +     - /mnt/smb/media/anime:/mnt/anime:ro
            #- /path/to/videos:/mnt/videos:ro
            #- /path/to/music:/mnt/music:ro
          ...
      ```

      **(Optional)** If your host deployment environment is capable of hardware transcoding (i.e. has a dedicated or integrated GPU through physical attachment or [hardware passthrough](../courses/hypervisor.md#hardware-passthrough)), uncomment the following section in the Docker compose file to passthrough the graphics device to the ErsatzTV container:

      ```diff
        ersatztv:
          container_name: ${APP_CONTAINER}
          ...
          #group_add:
          #  - '104' # This needs to be the group id of running `stat -c '%g' /dev/dri/renderD128` on the docker host
          ...
      -   #devices:
      -   #  - /dev/dri:/dev/dri
      +   devices:
      +     - /dev/dri:/dev/dri
          ...
      ```

   - An env file for the ErsatzTV stack on the app directory (i.e. `/mnt/smb/docker/ersatztv/.env`):

      ```sh
      SERVICE_NAME=ersatztv
      APP_CONTAINER=ersatztv
      APP_VERSION=develop@sha256:4da5d5e18cbfb125dd9885582ee139666c9a05975d3494d3e9ad4a614311d35c
      APP_TIMEZONE=Asia/Kuala_Lumpur
      APP_DIR=/mnt/smb/docker/ersatztv
      ```

      Replace all of the values in the env file with your own accordingly.

3. Go through the [post-installation setup steps](#post-install-setup) for the ErsatzTV server.

### Post-Install Setup

This details the post-installation steps of the ErsatzTV server for a complete setup:

1. Launch the ErsatzTV server web interface at `http://<ersatztv-server-host>:8409` (i.e. `http://192.168.0.106:8409`) on a web browser.

2. **(Optional)** Enable hardware accelerated transcoding on the used FFmpeg profile:

   - Click the **FFmpeg Profiles** menu option found on the left sidebar.
   - In the **FFmpeg Profiles** view, either update the default profile by clicking its corresponding **Edit Profile** button, or clicking the **ADD PROFILE** button.
   - In the FFmpeg profile form, configure the following:

     - General:

       - Name: Set a unique, descriptive name for the FFmpeg profile (i.e. `1920x1080 hevc ac3`)

     - Video:

       - Format: `hevc`
       - Bitrate: `4000`
       - Buffer Size: `8000`
       - Hardware Acceleration: Expand the dropdown and select the supported acceleration option by your graphics device (i.e. `Vaapi`)
       - VAAPI Driver: Expand the dropdown and select the intended driver (i.e. `RadeonSI`)
       - VAAPI Display: Expand the dropdown and select the intended display mode (i.e. `drm`)
       - VAAPI Device: Expand the dropdown and select the graphics device (i.e. `/dev/dri/renderD128`)

      Note that some of these settings may be missing or different depending on your values - click the **ADD PROFILE** or **SAVE PROFILE** button to submit and apply your changes.

3. Add media sources to the ErsatzTV server by integrating with a media server (i.e. Jellyfin), or locally:

   - Expand the **Media Sources** group found on the left sidebar.
   - To add media sources by integrating with a media server (i.e. Jellyfin):

     - Under the **Media Sources** group, click on the **Jellyfin** menu option.
     - In the **Jellyfin Media Sources** page, click the **CONNECT JELLYFIN** button.
     - In the provided form, configure the following:

       - Address: Fill in the address of the Jellyfin server at `http://<jellyfin-server-host>:8096` (i.e. `http://192.168.0.106:8096`)
       - Api Key: [Add an API key](jellyfin.md#generate-jellyfin-api-key) generated from the Jellyfin server

     - Click the **Edit Libraries** button corresponding to the Jellyfin server.
     - Toggle the **Synchronize** switch corresponding to each library you wish to sync (i.e. Movies, TV Shows, Anime, etc.)
     - Click the **SAVE CHANGES** button to start the synchronisation process.

   - To add media sources locally:

     - Under the **Media Sources** group, click on the **Local** menu option.
     - In the **Local Libraries** view, either update an existing library (i.e. **Movies**) by clicking its corresponding **Edit Library** button, or click the **ADD LOCAL LIBRARY** button.
     - In the provided **Local Library** form, configure the following:

       - Name: Set a unique, descriptive name for the media library (i.e. `Movies`)
       - Media Kind: Expand the dropdown and select the intended content type for the media library (i.e. `Movies`)
       - Path: Enter the path of the media directory inside the ErsatzTV server (i.e. `/mnt/movies`) and click the **ADD PATH** button

        Click the **ADD LOCAL LIBRARY** or **SAVE LOCAL LIBRARY** button to submit and apply your changes.

4. Once your media libraries have been synchronised, create a collection of media you wish to broadcast on the ErsatzTV server:

   - Expand the **Media** group found on the left sidebar and click the menu option that corresponds to the media type you wish to add to the collection (i.e. **Movies**).

   - Select the checkbox corresponding to each media you wish to add to the collection.

   - Click the **ADD TO COLLECTION** button.

   - In the provided **Add To Collection** form, configure the following:

     - Collection: Expand the dropdown and select the `(New Collection)` option if the collection has not been created
     - New Collection Name: Add a unique, descriptive name for the collection (i.e. `Anime`)

      Click the **ADD TO COLLECTION** button to submit the form.

5. Create a channel that is to broadcast your collection of media:

   - **(Optional)** Before creating the channel, create a custom watermark for the channel:

     - Click the **Watermarks** menu option found on the left sidebar.
     - In the **Watermarks** view, click the **ADD WATERMARK** button.
     - In the provided **Watermark** form, configure the following:

       - Name: Set a unique, descriptive name for the watermark (i.e. `MyLogo`)
       - Image: Click the **UPLOAD IMAGE** button and select an image file to upload and use as the watermark
       - Location: Expand the dropdown and select the intended location for the watermark to be displayed (i.e. `Top Right`)

        Click the **ADD WATERMARK** button to submit the form.

   - Click the **Channels** menu option found on the left sidebar.
   - In the **Channels** view, click the **ADD CHANNEL** button.
   - In the provided **Channel** form, configure the following:

     - Name: Set a unique, descriptive name for the channel (i.e. `Anime TV`)
     - Progress Mode: Expand the dropdown and select the intended mode on how the scheduled media programmes progress (i.e. `Always`)
     - Streaming Mode: Expand the dropdown and select the intended streaming method (i.e. `HLS Segmenter` if hardware transcoding is supported, otherwise `HLS Direct`)
     - FFmpeg Profile: Expand the dropdown and select the intended FFmpeg profile or leave as defaut (i.e. `1920x1080 hevc ac3`)
     - **(Optional)** Preferred Audio Language: Expand the dropdown and select the preferred audio language for the channel (i.e. `Japanese`)
     - **(Optional)** Preferred Subtitle Language: Expand the dropdown and select the preferred subtitle language for the channel (i.e. `English`)
     - **(Optional)** Subtitle Mode: Expand the dropdown and select the preferred method of displaying subtitles (i.e. `Any`)
     - **(Optional)** Watermark: Expand the dropdown and select a custom watermark for the channel (i.e. `MyLogo`)

      Click the **ADD CHANNEL** button to submit the form.

6. Create a broadcasting schedule for the media collection:

   - Expand the **Scheduling** group found on the left sidebar and click the **Schedules** menu option.
   - In the **Schedules** view, click the **ADD SCHEDULE** button.
   - In the provided **Schedule** form, configure the following:

     - Name: Set a unique, descriptive name for the broadcast schedule (i.e. `Anime TV`)
     - Keep Multi-Part Episodes Together: Select the corresponding checkbox to enable the described feature
     - Shuffle Schedule Items: Select the corresponding checkbox to enable the described feature

      Click the **ADD SCHEDULE** button to submit the form.

   - In the **Items** view of the schedule you created, click the **ADD SCHEDULE ITEM** button.
   - In the provided form, configure the following:

     - Collection: Expand the dropdown and select the intended media collection (i.e. `Anime`)
     - Playback Order: Expand the dropdown and select the intended playback order (i.e. `Shuffle In Order`)
     - **(Optional)** Watermark: Expand the dropdown and select a custom watermark for the broadcast schedule (i.e. `MyLogo`)
     - **(Optional)** Preferred Audio Language: Expand the dropdown and select the preferred audio language for the broadcast schedule (i.e. `Japanese`)
     - **(Optional)** Preferred Subtitle Language: Expand the dropdown and select the preferred subtitle language for the broadcast schedule (i.e. `English`)
     - **(Optional)** Subtitle Mode: Expand the dropdown and select the preferred method of displaying subtitles (i.e. `Any`)

      Click the **SAVE CHANGES** button to submit the form.

7. Create a playout to start broadcasting according to the created broadcast schedule:

   - Expand the **Scheduling** group found on the left sidebar and click the **Playouts** menu option.
   - In the **Playouts** view, click the **ADD PLAYOUT** button.
   - In the provided **Add Playout** form, configure the following:

     - Channel: Expand the dropdown and select the intended channel to send the broadcast to (i.e. `Anime TV`)
     - Schedule: Expand the dropdown and select the intended broadcast schedule to assign to the playout (i.e. `Anime TV`)

      Click the **ADD PLAYOUT** button to submit the form.

8. **(Optional)** Add the created TV channel(s) to a media streaming server (i.e. Jellyfin):

   - From the top right-hand corner of the web interface of ErsatzTV, do the following:

     - Right click the **M3U** link item, select the **Copy Link** option, and take note of the link.
     - Right click the **XMLTV** link item, select the **Copy Link** option, and take note of the link.

   - Using the values you had copied, [add the created TV channel(s) to your media streaming server](jellyfin.md#add-live-tv-channel-to-jellyfin) (i.e. Jellyfin).
