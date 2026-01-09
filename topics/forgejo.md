# Forgejo

## Description

Forgejo is software for hosting a forge using the Git version control system to aid with software development.

## Directory

- [Forgejo](#forgejo)
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
    - [Configuring Forgejo](#configuring-forgejo)
    - [Recommended Configurations](#recommended-configurations)
  - [Usage](#usage)
    - [Description](#description-3)
    - [References](#references-3)
    - [Adding a User to Forgejo](#adding-a-user-to-forgejo)
    - [Cloning Existing Git Repository to Forgejo](#cloning-existing-git-repository-to-forgejo)
    - [Repository Mirroring from Forgejo to GitHub](#repository-mirroring-from-forgejo-to-github)

## References

- [Forgejo](https://forgejo.org)
- [Wikipedia](https://en.wikipedia.org/wiki/Forgejo)

---

## Setup

### Description

This details how to install and set up an Forgejo server in a containerised environment.

### References

- [Installation with Docker](https://forgejo.org/docs/latest/admin/installation/docker)

### Installation

This details the installation steps for Forgejo as a containerised application:

1. On a [preconfigured Linux machine](linux.md#configuration) running on a [virtual machine](../courses/vm.md#creating-a-virtual-machine-from-a-template), bare metal device (i.e. [Raspberry Pi](raspberry-pi.md)), or perhaps an [LXC Container](../courses/container.md#create-lxc-container); ensure that [a Container Runtime is installed and set up](../courses/container.md#setting-up-a-container-runtime). For simplicity, the following considerations should be noted:

   - Either [disable the firewall](firewall.md#disablement) on the system or [allow access to the following port(s) and corresponding protocol(s)](firewall.md#adding-allow-rule):

     - `3000/tcp`
     - `222/tcp`

2. [Deploy the Forgejo stack with Compose or Portainer](../courses/container.md#container-runtime-usage) after preparing the following items:

   - A local app directory, on local storage (i.e. `/home/myuser/.local/share/docker/forgejo`): This will be used for at least the database volume directory which specifically requires local storage.

   - **(Optional)** A remote app directory, on remote mounted storage (i.e. `/mnt/smb/docker/forgejo`): This can be used for anything else that supports remote storage such as the Forgejo configuration volume directory.

   - A Compose file for the Forgejo stack on either app directory (i.e. `/mnt/smb/docker/forgejo/docker-compose.yml`):

      ```yaml
      name: ${SERVICE_NAME}
      services:
        server:
          container_name: ${FORGEJO_CONTAINER}
          image: codeberg.org/forgejo/forgejo:${FORGEJO_VERSION}
          ports:
            - 3000:3000
            - 222:22
          environment:
            - USER_UID=${HOST_USER_UID}
            - USER_GID=${HOST_USER_GID}
            - FORGEJO__database__DB_TYPE=postgres
            - FORGEJO__database__HOST=db:5432
            - FORGEJO__database__NAME=${DB_NAME}
            - FORGEJO__database__USER=${DB_USER}
            - FORGEJO__database__PASSWD=${DB_PASSWORD}
          volumes:
            - ${REMOTE_APP_DIR}/config:/data
            - /etc/timezone:/etc/timezone:ro
            - /etc/localtime:/etc/localtime:ro
          depends_on:
            - db
          networks:
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

        db:
          container_name: ${FORGEJO_DB_CONTAINER}
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
            - default
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true

      networks:
        default:
      ```

      If you are not using a remote app directory, replace `${REMOTE_APP_DIR}` with `${LOCAL_APP_DIR}` accordingly.

   - An env file for the Forgejo stack on either of the app directory (i.e. `/mnt/smb/docker/forgejo/.env`):

      ```sh
      SERVICE_NAME=forgejo
      FORGEJO_CONTAINER=forgejo
      FORGEJO_DB_CONTAINER=forgejo-db
      FORGEJO_VERSION=13.0
      HOST_USER_UID=1000
      HOST_USER_GID=1000
      REMOTE_APP_DIR=/mnt/smb/docker/forgejo
      POSTGRES_VERSION=16.3-bookworm
      DB_USER=postgres
      DB_PASSWORD=secret
      DB_NAME=forgejo-db
      LOCAL_APP_DIR=/home/myuser/.local/share/docker/forgejo
      ```

      Omit any variable(s) you may not need (i.e. `REMOTE_APP_DIR`) and replace the rest of the values with your own accordingly.

3. Go through the [post-installation setup steps](#post-install-setup) to complete the Forgejo server setup.

### Post-Install Setup

This details the post-installation steps of the Forgejo server for a complete setup:

1. Launch the Forgejo server web interface at `http://<forgejo-server-host>:3000` (i.e. `http://192.168.0.106:3000`) on a web browser.

2. In the initial onboarding **Initial configuration** view, configure the following settings in the provided form:

   - General settings:

     - **(Optional)** Disable self-registration: Select to enable the checkbox if you wish to disable self-registration of new users

   - **(Optional)** Email settings:

     - SMTP host: Set the SMTP server address of the sender email account (i.e. `smtp.gmail.com`)
     - SMTP port: Set the SMTP mail server port (i.e. `465`)
     - Send email as: Set the name and email address of the sender email account (i.e. `"Forgejo" <forgejo@example.com>`)
     - SMTP username: Set the email address of the sender email account (i.e. `forgejo@example.com`)
     - SMTP password: Set the (app) password of the sender email account
     - Require email confirmation to register: Select to enable the checkbox if you wish to require email verification for user self-registration
     - Enable email notifications: Select to enable the checkbox if you wish to enable email notifications

   - Server and third-party settings:

     - **(Optional)** Hide email addresses by default: Select to enable the checkbox if you wish to hide user email addresses by default

   - Administrator and account settings:

     - Administrator username: Set a unique, descriptive username for the Forgejo admin user (i.e. `admin`)
     - Email address: Set the email address of the Forgejo admin user (i.e. `admin@example.com`)
     - Password: Set a secure password for the Forgejo admin user
     - Confirm password: Confirm the password you have set for the Forgejo admin user

    You may leave the other settings as default and submit the form by clicking the **Install Forgejo** button.

3. Configure the Forgejo server based on the [recommended configuration options](#configuration).

---

## Configuration

### Description

This details how to configure a Forgejo server as well as some recommended configuration options.

### References

- [Push To Create](https://forgejo.org/docs/latest/user/push-to-create)
- [Reverse proxy](https://forgejo.org/docs/latest/admin/setup/reverse-proxy)

### Configuring Forgejo

This details how to configure Forgejo through its configuration file (`app.ini`):

1. Update the Forgejo configuration file (`app.ini`) from the host machine, or the Forgejo server itself:

   - To update the config file from the host machine:

      ```sh
      nano <path-to-app-dir>/config/gitea/conf/app.ini
      ```

      For example:

      ```sh
      nano /mnt/smb/docker/forgejo/config/gitea/conf/app.ini
      ```

   - **Alternatively**, to update the config file from the Forgejo server (i.e. container):

      ```sh
      nano /data/gitea/conf/app.ini
      ```

2. Define your configuration options, bearing in mind its `ini` syntax and configuration specifications, to the configuration file (`app.ini`):

   - Typically, adding a configuration option to an existing _group_ in the config file would look something like so:

      ```diff
        [some_group]
        some_option = some_value
      + my_option = my_value
      ```

      For example:

      ```ini
        [some_group]
        some_option = some_value
        my_option = my_value
      ```

   - Adding a configuration option to a new _group_ in the config file meanwhile would look something like so:

      ```diff
        [some_group]
        some_option = some_value
      +
      + [new_group]
      + my_option = my_value
      ```

      For example:

      ```ini
        [some_group]
        some_option = some_value

        [new_group]
        my_option = my_value
      ```

   - Lastly, to update an existing configuration option in the config file would look something like so:

      ```diff
        [some_group]
      - some_option = some_value
      + some_option = my_value
      ```

      For example:

      ```ini
        [some_group]
        some_option = my_value
      ```

3. Apply your pending configuration changes by deploying the Forgejo server or restarting the server if it is already running.

### Recommended Configurations

This details some recommended configuration options for a Forgejo setup:

1. **(Optional)** [Configure Forgejo](#configuring-forgejo) to enable Push to create, a feature that allows users to push to a repository that does not yet exist in Forgejo:

   - Add the following configuration options under the `repository` group in your Forgejo instance's configuration file (`app.ini`):

     - ENABLE_PUSH_CREATE_USER: Set this to `true` if you want to allow users to create repositories in their own user account
     - ENABLE_PUSH_CREATE_ORG: Set this to `true` if you want to allow users to create repositories in organisations they are members of
     - DEFAULT_PUSH_CREATE_PRIVATE: Set this to `true` to set the default visibility of pushed repositories to private

2. **(Optional)** [Configure Forgejo](#configuring-forgejo) to point to your [domain](../courses/network.md#registering-subdomains) (non-proxied) if you wish to [set up a reverse proxy](../courses/network.md#reverse-proxy) for the Forgejo instance:

   - First and foremost, prior to updating the following configurations, the following considerations should be noted:

     - Your Forgejo server domain should point directly to your public IP, not proxied through Cloudflare or similar services (i.e. DNS-only mode), as proxying can interfere with Git operations over SSH and cause connection issues.

     - Since the reverse proxy only covers the HTTP and HTTPS protocols, set up [port forwarding](../courses/network.md#port-forwarding) for SSH (typically port `22`) to allow Git operations over SSH to reach your Forgejo instance:

       - Service name: Set any suitable name that best describes the service or rule (i.e. `forgejo-ssh`)
       - Device IP address: Enter the private IP address of the Forgejo server (i.e. `192.168.0.106`)
       - Internal port: Enter the port number you have allocated and allowed for the Forgejo server or leave as default (i.e. `222`)
       - External port: Enter the default port for SSH (i.e. `22`)
       - Protocol: `TCP`
       - Enabled: Toggle or check this box to enable the port forwarding rule

   - Update the following configuration options under the `server` group on your Forgejo instance's configuration file (`app.ini`):

     - `DOMAIN`: Set this to the domain you had set up and configured for the Forgejo web server (i.e. `forgejo.example.com`)
     - `SSH_DOMAIN`: Set this to the domain you had set up and configured for the Forgejo SSH server - in most cases this should be the same as `DOMAIN` (i.e. `forgejo.example.com`)
     - `ROOT_URL`: Set this to the URL of the Forgejo web server (i.e. `https://forgejo.example.com/`)

   - **(Optional)** Update the following configuration option under the `security` group on your Forgejo instance's configuration file (`app.ini`) to establish a trusted network range for the reverse proxy:

     - `REVERSE_PROXY_TRUSTED_PROXIES`: Set this to the network range of your local network (i.e. `192.168.0.0/24`)

---

## Usage

### Description

This details some common usage steps for a Forgejo server.

### References

- [Repository Mirrors](https://forgejo.org/docs/latest/user/repo-mirror)
- [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- [Creating a personal access token (classic)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic)

### Adding a User to Forgejo

This details how to create user accounts that should have access to the Forgejo server:

1. Launch the Forgejo server web interface at `http://<forgejo-server-host>:3000` (i.e. `http://192.168.0.106:3000`) on a web browser.

2. Logged in as the admin user, click the profile icon found on the top right corner of the web interface to expand the **Profile and settings** menu.

3. Click the **Site administration** menu option.

4. In the **Admin settings** page, expand the **Identity & access** group, and click the **User accounts** menu option.

5. In the **Manage user accounts** page, click the **Create user account** button.

6. In the **Create user account** form, configure the following options:

   - Authentication source: Expand the dropdown and select the intended authentication source for the user (i.e. `Local`)
   - User visibility: Expand the dropdown and select the intended account visibility for the user (i.e. `Public`)
   - Username: Set a unique, descriptive username for the user (i.e. `john`)
   - Email address: Set the email address of the user (i.e. `john@example.com`)
   - Password: Set a secure password for the user
   - Require user to change password: Select to enable the checkbox if you wish to force the user to change the pre-assigned password after their initial login
   - Notify about registration via email: Select to enable the checkbox if you wish to notify the user via email about their registration

    Click the **Create user account** button to submit the form and finish creating the user account.

### Cloning Existing Git Repository to Forgejo

This details how to clone an existing Git repository to the Forgejo server:

1. From the Forgejo server web interface, create a new, empty repository as the intended target for the Git repository to be cloned to. If you intend to authenticate to the Forgejo server via SSH, please ensure that you have added your public SSH key to the Forgejo user account accordingly.

2. Assuming the existing source Git repository has been cloned locally on your machine, get into the directory of the repository:

    ```sh
    cd /path/to/repository
    ```

    For example:

    ```sh
    cd myrepo
    ```

3. List the current remote(s) configured for the repository:

    ```sh
    git remote
    ```

    Sample output:

    ```
      origin
    ```

4. In the existing Git repository, add the Forgejo server as a remote repository:

   - If the current remote is named `origin`, rename it to something more descriptive (i.e. `github`):

      ```sh
      git remote rename origin <name>
      ```

      For example:

      ```sh
      git remote rename origin github
      ```

   - Add a new remote named `origin` pointing to the target repository on the Forgejo server via SSH or HTTPS:

      SSH:

      ```sh
      git remote add origin ssh://git@forgejo.example.com/myuser/myrepo.git
      ```

      HTTPS:

      ```sh
      git remote add origin https://forgejo.example.com/myuser/myrepo.git
      ```

      Replace the links to the target repository (i.e. `forgejo.example.com/myuser/myrepo.git`) accordingly.

   - Verify that there should now be (at least) two remotes configured for the local repository:

      ```sh
      git remote
      ```

      Sample output:

      ```
        github
        origin
      ```

5. Push the source repository to the target repository on the Forgejo server:

   - Ensure that the current upstream is pointing to the source repository remote (i.e. `github`):

      ```sh
      git branch -vv
      ```

      Sample output:

      ```
        * master 032d53c [github/master] some commit
      ```

   - Push the source repository to the target repository remote on the Forgejo server (i.e. `origin`):

      ```sh
      git push --mirror origin
      ```

   - Change the current upstream to point to the target repository remote (i.e. `origin`) and branch (i.e. `master`):

      ```sh
      git branch --set-upstream-to=origin/master
      ```

      Ensure the current upstream has been updated:

      ```sh
      git branch -vv
      ```

      Sample output:

      ```
        * master 032d53c [origin/master] some commit
      ```

6. **(Optional)** If you wish for local commits to be pushed to both remotes by default going forward, configure the following:

   - Check the current remote targets for the local repository:

      ```sh
      git remote -v
      ```

      Sample output:

      ```
        github  git@github.com:myuser/myrepo.git (fetch)
        github  git@github.com:myuser/myrepo.git (push)
        origin  ssh://git@forgejo.example.com/myuser/myrepo.git (fetch)
        origin  ssh://git@forgejo.example.com/myuser/myrepo.git (push)
      ```

   - Set the `push` target for the `origin` remote to both its current `push` target, and the other intended remote's `push` target (i.e. `github`):

      ```sh
      git remote set-url --add --push origin ssh://git@forgejo.example.com/myuser/myrepo.git
      git remote set-url --add --push origin git@github.com:myuser/myrepo.git
      ```

   - Check the current remote targets again for the local repository:

      ```sh
      git remote -v
      ```

      Sample output:

      ```
        github  git@github.com:myuser/myrepo.git (fetch)
        github  git@github.com:myuser/myrepo.git (push)
        origin  ssh://git@forgejo.example.com/myuser/myrepo.git (fetch)
        origin  ssh://git@forgejo.example.com/myuser/myrepo.git (push)
        origin  git@github.com:myuser/myrepo.git (push)
      ```

   - With this configuration, commits pushed to the `origin` remote will be pushed to both remotes going forward.

    **Alternatively**, skip this step if you wish to set up [repository mirroring](#repository-mirroring-from-forgejo-to-github) instead.

### Repository Mirroring from Forgejo to GitHub

This details how to set up repository mirroring from the Forgejo server to GitHub:

1. To set up repository mirroring to GitHub from a Forgejo server, you will need to choose a method to authenticate to the GitHub server; either via SSH or HTTPS. If you choose the latter, please ensure that you have created a GitHub personal access token (classic) with the following scopes:

   - `public_repo`
   - `workflow`

2. The source repository in this exercise will be the Forgejo server repository. The target repository on GitHub must already be created and should meet either one of the two following criterias:

   - The repository should either be completely empty (i.e. no commits, branches, or tags)
   - The repository should already exist as an exact copy of the Forgejo server repository

3. Configure repository mirroring on the source repository:

   - On the Forgejo server web interface, navigate to the source repository page.

   - In the page of the source repository, click its corresponding **Settings** button.

   - In the **Repository** settings page, configure the **Mirror settings** form like so:

     - Git remote repository URL: Set this to either the SSH (i.e. `git@github.com:myuser/myrepo.git`) or HTTPS (i.e. `https://github.com/myuser/myrepo.git`) URL of the target repository on GitHub, depending on the authentication method of choice
     - Authorization:
       - **(HTTPS)** Username: Set the username of the GitHub user that owns the target repository
       - **(HTTPS)** Password: Set the configured personal access token of the GitHub user
       - **(SSH)** Use SSH authentication: Select to enable this checkbox if you instead choose to authenticate to the target GitHub repository via SSH
     - Sync when commits are pushed: Select to enable this checkbox if you wish for the mirror to be updated as soon as there are changes

      Click the **Add push mirror** button to submit the form.

   - **(SSH)** If you have chosen to authenticate to the target GitHub repository via SSH, please ensure that you have added the configured mirror's automatically generated public SSH key to the GitHub user account accordingly:

     - On the source repository, under the same **Mirror settings** section, click the **Copy public key** link corresponding to the configured mirror to obtain the public SSH key that you need.

4. Going forth, pushes to the source repository on the Forgejo server will be mirrored automatically to the target repository on GitHub.
