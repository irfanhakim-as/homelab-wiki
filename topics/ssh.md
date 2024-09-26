# SSH

## Description

Secure Shell (SSH) is a cryptographic network protocol for operating network services securely over an unsecured network.

## Directory

- [SSH](#ssh)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [Installation](#installation)
    - [Generate SSH keys](#generate-ssh-keys)
  - [Configuration](#configuration)
    - [Description](#description-2)
    - [Steps](#steps)
    - [Custom SSH Port](#custom-ssh-port)
    - [Disable Root Login](#disable-root-login)
    - [Disable Password Authentication](#disable-password-authentication)
  - [Copy SSH keys](#copy-ssh-keys)
    - [Description](#description-3)
    - [Steps](#steps-1)
  - [Remote access](#remote-access)
    - [Description](#description-4)
    - [References](#references-1)
    - [Enable remote access](#enable-remote-access)
    - [Remotely access using SSH](#remotely-access-using-ssh)

## References

- [Secure Shell](https://wiki.archlinux.org/title/Secure_Shell)

---

## Setup

### Description

This details how to install and setup SSH on a Linux system.

### Installation

1. Install SSH according to the system's Linux distribution.

    Arch Linux:

    ```sh
    sudo pacman -Syyu
    sudo pacman -S openssh
    ```

    Debian/Ubuntu:

    ```sh
    sudo apt update
    sudo apt install openssh-server
    ```

    Rocky Linux (RHEL):

    ```sh
    sudo yum update
    sudo yum install openssh-server
    ```

### Generate SSH keys

1. Generate SSH keys of the type `ed25519` with the following command:

    ```sh
    ssh-keygen -t ed25519
    ```

    > [!TIP]  
    > Besides the `ed25519` key type, there are several alternative types such as `ecdsa` and `rsa`.

    Go through the SSH key generation process, step by step.

2. When asked for where the key file should be saved, enter a valid path or hit <kbd>Enter</kbd> to accept the default (i.e. `~/.ssh/id_ed25519`).

3. When asked for a passphrase, enter a secure passphrase which will be required each time the key is used or hit <kbd>Enter</kbd> to set an empty passphrase. Do the same when prompted for a confirmation.

4. Once the key has been generated, take note of the paths to the newly generated keys. For example:

    - Private key: `~/.ssh/id_ed25519`
    - Public key: `~/.ssh/id_ed25519.pub`

---

## Configuration

### Description

This details the process of updating the system's SSH configuration and includes some recommended configuration options.

### Steps

1. Backup the original SSH configuration file:

    ```sh
    sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
    ```

2. Update the SSH configuration file:

    ```sh
    sudo nano /etc/ssh/sshd_config
    ```

3. Make any necessary changes and save the SSH configuration file.

4. Once **absolutely** confident and ready to apply the changes, [restart](systemd.md#restart-service) the SSH service `sshd.service`.

    > [!WARNING]  
    > Applying changes to the SSH configuration file without being prepared for the changes to take effect is highly discouraged.

### Custom SSH Port

This changes the system's SSH port from its default port `22` for better security.

1. In the SSH config file, search for the `Port` parameter.

2. Based on the default `Port` line:

    ```diff
    #Port 22
    +Port <port-number>
    ```

   - Comment the original `Port` line if it is not already commented to override it.

   - Add a new `Port` line underneath the original with `<port-number>` being the new port number you had chosen (i.e. `2222`).

   - Example changes:

        ```
        #Port 22
        Port 2222
        ```

### Disable Root Login

This disables logging in as root via SSH.

1. In the SSH config file, search for the `PermitRootLogin` parameter.

2. Based on the default `PermitRootLogin` line:

    ```diff
    #PermitRootLogin prohibit-password
    +PermitRootLogin no
    ```

   - Comment the original `PermitRootLogin` line if it is not already commented to override it.

   - Add a new `PermitRootLogin` line with its value set to `prohibit-password` (good security) or `no` (best security).

   - Example changes:

        ```
        #PermitRootLogin prohibit-password
        PermitRootLogin no
        ```

### Disable Password Authentication

This disables authenticating users with their passwords via SSH, requiring the use of SSH keys instead.

1. In the SSH config file, search for the `PasswordAuthentication` parameter.

2. Based on the default `PasswordAuthentication` line:

    ```diff
    #PasswordAuthentication yes
    +PasswordAuthentication no
    ```

   - Comment the original `PasswordAuthentication` line if it is not already commented to override it.

   - Add a new `PasswordAuthentication` line with its value set to `no` to only allow user authentication using SSH keys.

   - Example changes:

        ```
        #PasswordAuthentication yes
        PasswordAuthentication no
        ```

---

## Copy SSH keys

### Description

This details how to copy your SSH keys to a remote system.

### Steps

1. [Generate SSH keys](#generate-ssh-keys) on the local system if they do not already exist.

2. Copy your SSH keys to the remote server using `ssh-copy-id`:

   - Copy the public key to the remote server:

        ```sh
        ssh-copy-id -i <public-key-path> <remote-user>@<ip-address>
        ```

     - `<public-key-path>`: Replace this with the path to your public key (i.e. `~/.ssh/id_ed25519.pub`)
     - `<remote-user>`: Replace this with the username of the remote system (i.e. `myuser`)
     - `<ip-address>`: Replace this with the IP address of the remote system (i.e. `192.168.0.86`)

   - If the remote server's SSH port is not the default (i.e. `22`), you could add the `-p` flag to specify the port number:

        ```sh
        ssh-copy-id -i <public-key-path> -p <port-number> <remote-user>@<ip-address>
        ```
     - `<port-number>`: Replace this with the SSH port number of the remote system (i.e. `2222`)

3. **Alternatively**, you could also do so without `ssh-copy-id`:

    ```sh
    cat <public-key-path> | ssh <remote-user>@<ip-address> -p <port-number> 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
    ```

4. Verify that the key has been copied successfully by [remotely accessing](#remotely-access-using-ssh) the remote server using SSH.

    If your public key has been copied successfully, you should not be prompted for a password.

---

## Remote access

### Description

This details how to enable remote access to your machine and to access another machine using SSH.

### References

- [OpenSSH](https://wiki.archlinux.org/title/OpenSSH)

### Enable remote access

1. Ensure SSH is [installed](#installation) on both the local and remote system.

2. [Start and enable](systemd.md#enable-service) the SSH service `sshd.service`.

3. Once the SSH service is active, said machine should now be able to be remotely accessed via another device using SSH.

### Remotely access using SSH

1. Connect to the another machine (with SSH service enabled) using SSH.

    ```sh
    ssh <remote-user>@<ip-address> -p <port-number>
    ```

    > [!TIP]  
    > You could omit the `-p` flag and value if the remote server's SSH port is the default (i.e. `22`).

2. Enter the password of the remote user when prompted.

    > [!NOTE]  
    > If you have [copied](#copy-ssh-keys) over your public SSH key to the remote server, you should not be prompted for a password.
