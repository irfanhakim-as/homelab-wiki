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
  - [Copy SSH keys](#copy-ssh-keys)
    - [Description](#description-2)
    - [Steps](#steps)
  - [Remote access](#remote-access)
    - [Description](#description-3)
    - [References](#references-1)
    - [Enable remote access](#enable-remote-access)
    - [Remotely access using SSH](#remotely-access-using-ssh)

## References

- [Secure Shell](https://wiki.archlinux.org/title/Secure_Shell)

---

## Setup

### Description

This details how to install and setup SSH on your system.

### Installation

1. Install SSH according to your Linux distribution.

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

## Copy SSH keys

### Description

This details how to copy your SSH keys to a remote server.

### Steps

1. Copy your SSH keys to the remote server using `ssh-copy-id`.

    ```sh
    ssh-copy-id -i <public-key-path> <remote-user>@<ip-address>
    ```

    If the remote server's SSH port is not the default (i.e. `22`), you could add the `-p` flag to specify the port number:

    ```sh
    ssh-copy-id -i <public-key-path> -p <port-number> <remote-user>@<ip-address>
    ```

2. Alternatively, you could also do so without `ssh-copy-id`:

    ```sh
    cat <public-key-path> | ssh <remote-user>@<ip-address> -p <port> 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
    ```

3. Verify that the key has been copied successfully by [remotely accessing](#remotely-access-using-ssh) the remote server using SSH.

    If your public key has been copied successfully, you should not be prompted for a password.

---

## Remote access

### Description

This details how to enable remote access to your machine and to access another machine using SSH.

### References

- [OpenSSH](https://wiki.archlinux.org/title/OpenSSH)

### Enable remote access

1. Enable the SSH service according to your Linux distribution.

    Arch Linux:

    ```sh
    sudo systemctl enable --now sshd.service
    ```

    Debian/Ubuntu:

    ```sh
    sudo systemctl enable --now ssh.service
    ```

    Rocky Linux (RHEL):

    ```sh
    sudo systemctl enable --now sshd.service
    ```

2. Once the SSH service is active, said machine should now be able to be remotely accessed via another device using SSH.

### Remotely access using SSH

1. Connect to the another machine (with SSH service enabled) using SSH.

    ```sh
    ssh <remote-user>@<ip-address> -p <port>
    ```

    > [!TIP]  
    > You could omit the `-p` flag if the remote server's SSH port is the default (i.e. `22`).

2. Enter the password of the remote user when prompted.

    > [!NOTE]  
    > If you have copied your public SSH key to the remote server, you should not be prompted for a password.