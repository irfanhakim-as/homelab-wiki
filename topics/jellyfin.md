# Jellyfin

## Description

Jellyfin is a Free Software Media System that puts you in control of managing and streaming your media. It is an alternative to the proprietary Emby and Plex, to provide media from a dedicated server to end-user devices via multiple apps.

## Directory

- [Jellyfin](#jellyfin)
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
    - [Configuring Jellyfin](#configuring-jellyfin)
    - [Adding Plugins to Jellyfin](#adding-plugins-to-jellyfin)
    - [Customising Jellyfin Appearance](#customising-jellyfin-appearance)
    - [Recommended Configurations](#recommended-configurations)
  - [Usage](#usage)
    - [Description](#description-3)
    - [References](#references-3)
    - [Adding a User to Jellyfin](#adding-a-user-to-jellyfin)

## References

- [Jellyfin](https://jellyfin.org)
- [GitHub](https://github.com/jellyfin/jellyfin)
- [Wikipedia](https://en.wikipedia.org/wiki/Jellyfin)

---

## Setup

### Description

This details how to install and set up a Jellyfin server in a containerised environment.

### References

- [Using Docker Compose](https://jellyfin.org/docs/general/installation/container/#using-docker-compose)
- [Hardware Acceleration](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration)
- [linuxserver/jellyfin](https://docs.linuxserver.io/images/docker-jellyfin)
- [Understanding PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid)
- [Setup Wizard Walkthrough](https://jellyfin.org/docs/general/post-install/setup-wizard)

### Installation

This details the installation steps for Jellyfin as a Docker container:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [Docker is installed and set up](../courses/container.md#setting-up-docker). The following considerations should be noted:

   - Hardware-accelerated transcoding is highly recommended for the best Jellyfin experience - setting up an [LXC Container on Proxmox](../courses/container.md#create-lxc-container) and [passing through a (dedicated or integrated) GPU](../courses/hypervisor.md#hardware-passthrough) to it may be the easiest way to set up as the host deployment environment for Jellyfin, paying careful attention to the following items in your deployment:

     - First of all, when deploying with Docker Compose, consider the user that the Jellyfin container will run as - we recommend _emulating_ as a non-root, [service user](linux.md#create-user) that you are using on the host deployment environment (i.e. LXC Container).
     - When deploying with Docker Compose, besides passing through a (dedicated or integrated) GPU to the host deployment environment (i.e. LXC Container), the graphics device should also be passed through to the Jellyfin container - this is covered in the subsequent deployment steps.
     - Another thing you are likely going to do in your deployment is to pass through a remote or network storage directories to the Jellyfin container for serving media files - this could be an SMB share that you mount on a Proxmox node host, then passed through to the LXC Container (before again passing through media directories from it to the Jellyfin container).
     - In both aforementioned cases, it is **important** to ensure that the user the Jellyfin container will run as has the required permissions to access the graphics device and directories that you are passing through - you might think using `group_add` in the Docker compose file to solve any permission issue (i.e. by adding the Jellyfin container user to the same group GIDs as the graphics device and media directories) would be the logical solution, but based on our testing however, it might not be (at least not with an "LXC + Docker Container" combo). To address this issue:

       - When [passing through the graphics device](../courses/hypervisor.md#hardware-passthrough) to the host deployment environment (i.e. LXC Container on Proxmox), ensure that it is passed through to the UID of the intended Jellyfin container user (i.e. `1000`). Verify that said UID has the rights to the graphics device on the host deployment environment:

          ```sh
          stat -c '%u' /dev/dri/*
          ```

          Sample UID output (i.e. `1000`):

          ```
            1000
            1000
          ```

       - When [passing through a remote or network storage (i.e. SMB share) to the host deployment environment (i.e. LXC Container, from the Proxmox node host)](../courses/container.md#mount-smb-share-on-lxc-container) - which will later be passed through to the Jellyfin container, take note of the GID that has the rights to the mounted storage on the host deployment environment (i.e. `/mnt/smb`):

          ```sh
          stat -c '%g' <mountpoint>
          ```

          For example:

          ```sh
          stat -c '%g' /mnt/smb
          ```

          Sample GID (i.e. `10000`) output:

          ```
            10000
          ```

       - Finally, based on the UID (i.e. `1000`) and GID (i.e. `10000`) values, set the Jellyfin container user UID and GID accordingly using the following variables in your deployment's env file. For example:

          ```diff
            ...
          - APP_USER_UID=0
          - APP_USER_GID=0
          + APP_USER_UID=1000
          + APP_USER_GID=10000
            ...
          ```

   - Either [disable the firewall](firewall.md#disablement) on the host deployment environment or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule):

     - `8096/tcp`
     - `7359/udp`
     - `1900/udp`

2. [Deploy the Jellyfin stack with Docker Compose](../courses/container.md#docker-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/jellyfin`) or a remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/jellyfin`): This will be used for the Jellyfin stack's volume(s) - note that this might take up a huge amount of space, depending on the number of media files you will be serving.

   - **(Optional)** Remote media directories, on remote mounted storage (i.e. `/mnt/smb/media/movies`, `/mnt/smb/media/tvshows`, etc.): These directories are expected to contain the media files that are meant to be served in Jellyfin.

   - A Docker compose file for the Jellyfin stack on the app directory (i.e. `/mnt/smb/docker/jellyfin/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        jellyfin:
          container_name: ${APP_CONTAINER}
          image: docker.io/linuxserver/jellyfin:${APP_VERSION}
          ports:
            - 8096:8096
            #- 8920:8920 # optional
            - 7359:7359/udp # optional
            - 1900:1900/udp # optional
          #group_add:
          #  - '104' # This needs to be the group id of running `stat -c '%g' /dev/dri/renderD128` on the docker host
          environment:
            - PUID=${APP_USER_UID}
            - PGID=${APP_USER_GID}
            - TZ=${APP_TIMEZONE}
          volumes:
            - ${APP_DIR}/config:/config
            #- /path/to/movies:/mnt/movies:ro
            #- /path/to/tvshows:/mnt/tvshows:ro
            #- /path/to/anime:/mnt/anime:ro
            #- /path/to/videos:/mnt/videos:ro
            #- /path/to/music:/mnt/music:ro
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

      Update the following section in the Docker compose file with the actual paths to the directories that store your media files (i.e. Movies, TV Shows, Anime, etc.):

      ```diff
        jellyfin:
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

      **(Optional)** If your host deployment environment is capable of hardware transcoding (i.e. has a dedicated or integrated GPU through physical attachment or [hardware passthrough](../courses/hypervisor.md#hardware-passthrough)), uncomment the following section in the Docker compose file to passthrough the graphics device to the Jellyfin container:

      ```diff
        jellyfin:
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

   - An env file for the Jellyfin stack on the app directory (i.e. `/mnt/smb/docker/jellyfin/.env`):

      ```sh
      SERVICE_NAME=jellyfin
      APP_CONTAINER=jellyfin
      APP_VERSION=version-10.10.6ubu2404
      APP_USER_UID=0
      APP_USER_GID=0
      APP_TIMEZONE=Asia/Kuala_Lumpur
      APP_DIR=/mnt/smb/docker/jellyfin
      ```

      Replace all of the values in the env file with your own accordingly.

3. Go through the [post-installation setup steps](#post-install-setup) for the Jellyfin server.

### Post-Install Setup

This details the post-installation steps of the Jellyfin server for a complete setup:

1. Launch the Jellyfin server web interface at `http://<jellyfin-server-host>:8096` (i.e. `http://192.168.0.106:8096`) on a web browser.

2. When prompted for your **Preferred display language**, expand the dropdown and select the intended language for your client (i.e. `English`).

3. Set up an administrator account for managing the Jellyfin server by configuring the following in the provided form:

   - Username: Set a unique, descriptive username for the Jellyfin user (i.e. `admin`)
   - Password: Set a secure password for the Jellyfin admin user
   - Password (confirm): Confirm the password you have set for the Jellyfin admin user

    Submit the form by clicking the **Next** button.

4. Set up the media libraries on the Jellyfin server for each media directory you have passed through to the Jellyfin stack:

   - Click the **Add Media Library** button.
   - In the **Add Media Library** form, configure the following:

     - Content type: Expand the dropdown and select the intended content type for the media library (i.e. `Movies`)
     - Display name: Set a unique, descriptive name for the media library or leave as default (i.e. `Movies`)
     - Under the **Library Settings** section:

       - Enable the library: Select the corresponding checkbox to enable the media library
       - Enable real time monitoring: Select the corresponding checkbox to enable real time monitoring for the media library
       - Metadata downloaders: Select the checkboxes corresponding to the metadata sources you wish to use for the media library and rank them in order of priority
       - Image fetchers: Select the checkboxes corresponding to the image sources you wish to use for the media library and rank them in order of priority

     - Under the **Trickplay** section:

       - **(Optional)** Enable trickplay image extraction: Select the corresponding checkbox to enable previews while scrubbing through your media content

     - Under the **Subtitle Downloads** section:

       - Subtitle downloaders: Select the checkboxes corresponding to the subtitle sources you wish to use for the media library and rank them in order of priority
       - Only download subtitles that are a perfect match for video files: Select the corresponding checkbox to enable the described feature
       - Save subtitles into media folders: Select to enable the corresponding checkbox to store subtitles downloaded by Jellyfin alongside their corresponding media file

     - Reverting back to the **Folders** section, click its corresponding **Add** button and configure the following in the **Select Path** form:

       - Folder: Enter the path of the media directory inside the Jellyfin server (i.e. `/mnt/movies`)

        Submit the form by clicking the **Ok** button.

   - Click the **Ok** button to finish adding the media library.

5. Set the preferred metadata language server-wide by configuring the following form:

   - Language: Expand the dropdown and select the preferred language for metadata on the Jellyfin server (i.e. `English`)
   - Region: Expand the dropdown and select the preferred region for metadata on the Jellyfin server (i.e. `United States`)

6. Set up remote access for the Jellyfin server by configuring the following form:

   - Allow remote connections to this server: Select the corresponding checkbox to allow remote access to the Jellyfin server
   - Enable automatic port mapping: Deselect the corresponding checkbox to disable automatic port mapping on your router

7. Once you are in the **You're Done!** view, click the **Finish** button to complete the Jellyfin server setup.

8. **(Optional)** Proceed to configure the Jellyfin server according to the [recommended configuration options](#recommended-configurations).

---

## Configuration

### Description

This details how to configure a Jellyfin server as well as some recommended configuration options.

### References

- [Plugins](https://jellyfin.org/docs/general/server/plugins)
- [CSS Customization](https://jellyfin.org/docs/general/clients/css-customization)

### Configuring Jellyfin

This details how to configure a Jellyfin server through the web interface:

1. Launch the Jellyfin server web interface at `http://<jellyfin-server-host>:8096` (i.e. `http://192.168.0.106:8096`) on a web browser.

2. In the **Login** page, either click on the profile icon corresponding to the Jellyfin admin user or click the **Manual Login** button.

3. In the provided form, fill in the admin user credentials, and click the **Sign In** button.

4. From the **Home** view of the Jellyfin server, click on the user profile icon from the top right corner of the web interface to get to the **Settings** page.

5. In the **Settings** page, click any of the available menu option according to what you wish to configure:

   - Menu options under the **Username** (i.e. `admin`) section offer settings specific to the user and may need re-configuration on a per-device basis.
   - Menu options under the **Administration** section offer settings specific to the Jellyfin server and apply to all users.

    In most cases, settings you would wish to configure for the server, as an admin, should be in the **Dashboard** menu option under the **Administration** section.

6. Assuming that you are in the **Dashboard** menu, click on any of the available menu options found on the left sidebar to configure the selected groups of settings.

7. In the selected group of settings, make your configurations accordingly, and save your changes by clicking the **Save** button.

8. Some configuration changes, such as [adding plugins](#adding-plugins-to-jellyfin), may require a restart of the Jellyfin server to take effect - do so accordingly when prompted by heading back to the **Dashboard** view and clicking the **Restart** button.

### Adding Plugins to Jellyfin

This details how to add additional plugins to extend the functionality of a Jellyfin server:

1. [Configure](#configuring-jellyfin) the **Dashboard** menu option under the **Administration** section as an admin user.

2. Click on the **Catalog** menu option found on the left sidebar.

3. **(Optional)** Add additional repositories for a larger selection of plugins to install:

   - In the **Catalog** view, click on the **Settings** button found on the top left corner, next to the view title.
   - In the **Repositories** section, click on the **Add** button, and configure the following:

     - Repository Name: Add a unique, descriptive name for the plugin repository (i.e. `Sample Plugins`)
     - Repository URL: Add the full URL path to the plugin repository's manifest file (i.e. `https://sample-plugins.example.com/manifest.json`)

      Click the **Save** button to finish adding the repository.

   - Back in the **Repositories** view, click the **Back** button once you are done.

4. In the **Catalog** view, scroll through the available plugins of various kinds, such as:

   - General
   - Metadata
   - Notifications

    Once you have found the plugin you wish to install, click on the plugin accordingly.

5. In the selected plugin's view, click its corresponding **Install** button to install the plugin.

6. Once the plugin is installed, if prompted to restart the Jellyfin server, head back to the **Dashboard** view and click the **Restart** button.

### Customising Jellyfin Appearance

This details how to customise the appearance of the Jellyfin web interface using CSS:

1. [Configure](#configuring-jellyfin) the **Dashboard** menu option under the **Administration** section as an admin user.

2. Click on the **General** menu option found on the left sidebar.

3. In the General **Settings** view, scroll down to the **Branding** section.

4. Under the **Branding** section, configure the **Custom CSS code** setting with your own CSS-based tweaks. **Alternatively**, you could add in or import [custom designs made by the community](https://jellyfin.org/docs/general/clients/css-customization/#community-themes) into this field. For example:

    ```css
    @import url('https://one-custom-design.example.com/theme.css');
    @import url('https://another-custom-design.example.com/theme.css');
    ```

5. Click the **Save** button to save and apply your changes.

### Recommended Configurations

This details some recommended configuration options for a complete Jellyfin server setup:

1. The following are a list of recommended [plugins to install](#adding-plugins-to-jellyfin) for enhancing the functionality of a Jellyfin server:

   - Intro Skipper [[Intro Skipper](https://manifest.intro-skipper.org/manifest.json)]: Automatically detect and skip intro/credit sequences in Jellyfin
   - Merge Versions [[Jellyfin Plugin Manifest](https://raw.githubusercontent.com/danieladov/JellyfinPluginManifest/master/manifest.json)]: This plugin scans all your movies or TV shows and groups every repeated media files in one version
   - Playback Reporting [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]: Collect and show user playback statistics, such as total time watched, media watched, time of day watched, and time of week watched
   - Session Cleaner [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]: This plugin allows automatic cleaning of leftover sessions
   - Transcode Killer [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]: Plugin to automatically kill transcoded streams

2. Recommended metadata sources for media libraries of various types on Jellyfin:

   - [Install the following plugins](#adding-plugins-to-jellyfin) from their respective repositories if they are not already installed:

     - AniDB [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]
     - AniList [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]
     - Fanart [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]
     - TheTVDB [[Jellyfin Stable](https://repo.jellyfin.org/files/plugin/manifest.json)]

   - [Configure the Jellyfin server](#configuring-jellyfin)'s media **Libraries** according to their content type, then enable and rank the following metadata sources in the presented order, for the following settings:

     - Movies:

       - Metadata downloaders (Movies):
         - TheMovieDb
         - TheTVDB
         - AniDB
         - AniList
         - The Open Movie Database
       - Image fetchers (Movies):
         - TheMovieDb
         - TheTVDB
         - AniDB
         - AniList
         - The Open Movie Database
         - Embedded Image Extractor
         - Fanart
         - Screen Grabber

     - TV Shows:

       - Metadata downloaders (TV Shows):
         - TheTVDB
         - TheMovieDb
         - AniDB
         - AniList
         - The Open Movie Database
         - Missing Episode Fetcher
       - Metadata downloaders (Seasons):
         - TheTVDB
         - TheMovieDb
         - AniDB
       - Metadata downloaders (Episodes):
         - TheTVDB
         - TheMovieDb
         - AniDB
         - The Open Movie Database
       - Image fetchers (TV Shows):
         - TheTVDB
         - TheMovieDb
         - AniDB
         - AniList
         - Fanart
       - Image fetchers (Seasons):
         - TheTVDB
         - TheMovieDb
         - AniDB
         - AniList
         - Fanart
       - Image fetchers (Episodes):
         - TheTVDB
         - TheMovieDb
         - The Open Movie Database
         - Embedded Image Extractor
         - Screen Grabber

3. **(Optional)** [Customise the appearance](#customising-jellyfin-appearance) of the Jellyfin server with [Scyfin's CSS theme](https://github.com/loof2736/scyfin):

    ```css
    @import url('https://cdn.jsdelivr.net/gh/loof2736/scyfin@latest/CSS/scyfin-theme.css');
    @import url('https://cdn.jsdelivr.net/gh/loof2736/scyfin@latest/CSS/disable-static-drawer.css');
    ```

4. **(Optional)** Recommended [per-user](#adding-a-user-to-jellyfin), per-client [configurations](#configuring-jellyfin):

   - Display:

     - Under the **Libraries** section, enable the **Backdrops** option
     - Under the **Next Up** section, enable the **Enable Rewatching in Next Up** option

   - Home:

     - Home screen ordering:

       - Section 1: `Continue Watching`
       - Section 2: `Continue Listening`
       - Section 3: `Continue Reading`
       - Section 4: `Next Up`
       - Section 5: `Recently Added Media`
       - Section 6: `Live TV`
       - Section (Last): `My Media`

     - Library Order:

       - Movies
       - TV Shows
       - Anime
       - Videos
       - Live TV

     - Default screen for each media library: `Suggestions`

   - Subtitles:

     - Preferred subtitle language: `English`
     - Subtitle mode: `Always Play`
     - Always burn in subtitle when transcoding: Check the corresponding checkbox to enable the described option - this helps to work around a bug that causes desynchronised subtitles when transcoding
     - Text size: `Extra Large`

   - Media library view configurations:

     - View: `Poster`
     - Sort By: `Date Added` (Movies) or `Date Episode Added` (TV Shows)
     - Sort Order: `Descending`

---

## Usage

### Description

This details some common usage steps for a Jellyfin server.

### References

- [Managing Users](https://jellyfin.org/docs/general/server/users/adding-managing-users)

### Adding a User to Jellyfin

This details how to add a user to the Jellyfin server:

1. [Configure](#configuring-jellyfin) the **Dashboard** menu option under the **Administration** section as an admin user.

2. Click on the **Users** menu option found on the left sidebar.

3. In the **Users** view, click the **Add User** button found on the top left corner, next to the view title.

4. In the **Add User** form, configure the following:

   - Name: Set a unique, descriptive name for the Jellyfin user (i.e. `john`)
   - Password: Set a secure password for the Jellyfin user
   - Enable access to all libraries: Select the corresponding checkbox if you wish to enable access to all existing and future media libraries on the Jellyfin server
   - Libraries: **Alternatively**, select the checkboxes corresponding to the media libraries you wish to give the user access to

    Click the **Save** button to submit the form and create the user account.
