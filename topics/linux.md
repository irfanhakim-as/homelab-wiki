# Linux

## Description

Linux is both an open-source Unix-like kernel and a generic name for a family of open-source Unix-like operating systems based on the Linux kernel, an operating system kernel first released on September 17, 1991, by Linus Torvalds.

## Directory

- [Linux](#linux)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Ubuntu](#ubuntu)
    - [Description](#description-1)
    - [References](#references-1)
    - [Minimum Requirements](#minimum-requirements)
    - [Installation](#installation)
    - [Configuration](#configuration)
      - [Template VM](#template-vm)
      - [Individual VM](#individual-vm)
    - [Adding a New Disk](#adding-a-new-disk)
    - [Expanding a Disk Partition](#expanding-a-disk-partition)
  - [Debian](#debian)
    - [Description](#description-2)
    - [References](#references-2)
    - [Minimum Requirements](#minimum-requirements-1)
    - [Installation](#installation-1)
    - [Configuration](#configuration-1)
      - [Template VM](#template-vm-1)
      - [Individual VM](#individual-vm-1)
    - [Adding a New Disk](#adding-a-new-disk-1)
    - [Expanding a Disk Partition](#expanding-a-disk-partition-1)
  - [Rocky Linux (RHEL)](#rocky-linux-rhel)
    - [Description](#description-3)
    - [References](#references-3)
    - [Minimum Requirements](#minimum-requirements-2)
    - [Installation](#installation-2)
    - [Configuration](#configuration-2)
      - [Template VM](#template-vm-2)
      - [Individual VM](#individual-vm-2)
    - [Adding a New Disk](#adding-a-new-disk-2)
    - [Expanding a Disk Partition](#expanding-a-disk-partition-2)

## References

- [Linux](https://en.wikipedia.org/wiki/Linux)
- [Linux distribution](https://en.wikipedia.org/wiki/Linux_distribution)
- [List of TCP and UDP port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)

---

## Ubuntu

### Description

This details the installation and configuration of Ubuntu as the operating system of choice for a virtual machine.

### References

- [Ubuntu 20.04.6 LTS (Focal Fossa)](https://releases.ubuntu.com/focal)

### Minimum Requirements

- CPU: `1`
- Memory: `1GB`
- Storage: `10GB`

### Installation

1. Go through the installation wizard as per usual.

2. In the **Profile setup** section:

    - Your name: Set a name suitable for your homelab.
    - Your server's name: Set a unique name for said server, without the domain name. i.e. `ubuntu-server`.
    - Pick a username: Set a username suitable for your homelab.
    - Choose a password: Set a secure, minimum 12 character password.
    - Confirm your password: Confirm the password you have set.

3. In the **SSH Setup** section:

    - Install OpenSSH server: `Enabled`
    - Import SSH identity: `Disabled`

4. Complete the installation.

### Configuration

#### Template VM

1. Login to the server using the service account credentials you have set during installation.

2. Update the system:

    ```sh
    sudo apt update && sudo apt upgrade -y
    ```

3. Install the following packages:

    ```sh
    sudo apt install -y \
        curl \
        git \
        htop \
        nano \
        net-tools \
        openssh-server \
        python3 \
        python3-pip \
        wget
    ```

    On a VMware ESXi host, install the following package:

    ```sh
    sudo apt install -y open-vm-tools
    ```

    On a Proxmox host, install the following package:

    ```sh
    sudo apt install -y qemu-guest-agent
    ```

    > [!TIP]  
    > On Proxmox, you may also need to enable the `qemu-guest-agent` service yourself.

4. Clean up the system:

    ```sh
    sudo apt autoremove -y && sudo apt clean
    ```

5. Set up the VM's firewall.

   - Enable the firewall:

        ```bash
        sudo ufw enable
        ```

   - Check the firewall status:

        ```bash
        sudo ufw status numbered
        ```

   - Allow new SSH port:

        ```bash
        sudo ufw allow <port>/tcp
        ```

        > [!TIP]  
        > Replace `<port>` with the new port number you wish to use.

6. Update the VM's SSH configuration.

   - Update the SSH configuration file:

        ```sh
        sudo nano /etc/ssh/sshd_config
        ```

   - Change the default SSH port:

        ```sh
        #Port 22
        ```

       - Uncomment the `Port` line if it is commented.
       - Set the `Port` value to the new port number you have selected in the previous step.

   - Disable root login:

        ```sh
        #PermitRootLogin prohibit-password
        ```

       - Uncomment the `PermitRootLogin` line if it is commented.
       - Set the `PermitRootLogin` value to `prohibit-password` (good security) or `no` (best security).

   - Disable password authentication:

        ```sh
        #PasswordAuthentication yes
        ```

       - Uncomment the `PasswordAuthentication` line if it is commented.
       - Set the `PasswordAuthentication` value to `no`.

   - [Enable the SSH service](ssh.md#enable-remote-access) (in case it is not yet enabled).

   - Restart the SSH service:

        > [!WARNING]  
        > Make sure you have [copied your public SSH key(s)](ssh.md#copy-ssh-keys) to the server before doing this!

        ```sh
        sudo systemctl restart ssh
        ```

7. Reboot the VM to apply all changes:

    ```sh
    sudo reboot now
    ```

#### Individual VM

1. Login to the server using the service account credentials you have set during installation.

2. Set static IP address.

3. Update the system hostname.

4. Update the system.

### Adding a New Disk

TODO

### Expanding a Disk Partition

TODO

---

## Debian

### Description

This details the installation and configuration of Debian 12 as the operating system of choice for a virtual machine.

### References

- [Debian CD](https://cdimage.debian.org/debian-cd/current/amd64/bt-cd)
- [Debian 12 "Net Install" Installation Walkthrough](https://youtu.be/gddlhr9ST9Y)
- [How to fix debian sudo command not found](https://linuxhint.com/how-to-fix-debian-sudo-command-not-found)
- [How To Set Up a Firewall with UFW on Debian 12](https://www.cyberciti.biz/faq/set-up-a-firewall-with-ufw-on-debian-12-linux)

### Minimum Requirements

- CPU: `1`
- Memory: `512MB`
- Storage: `10GB`

### Installation

1. Pick the **Install** option when given the option.

2. Go through the installation wizard as per usual.

3. In the **Configure the network** section:

    - Hostname: Set a unique name for said server, without the domain name. i.e. `debian-server`.
    - Domain name: Leave this blank or add in your domain name i.e. `example.com`.

4. In the **Set up users and passwords** section:

    - Root password: Set a secure, minimum 12 character password for the `root` user.
    - Re-enter root password to verify: Confirm the password you have set for the `root` user.
    - Full name for the new user: Set a name suitable for your homelab.
    - Username for your account: Set a username suitable for your homelab.
    - Choose a password for the new user: Set a secure, minimum 12 character password **different** from the password set for the `root` user.
    - Re-enter password to verify: Confirm the password you have set.

5. In the **Partition disks** section:

    - Partitioning method: Select the **Guided - use entire disk** option.
    - Select disk to partition: Select the disk you wish to use for the installation.
    - Partitioning scheme: Select the **All files in one partition (recommended for new users)** option.
    - Overview of the new partitions: Select the **Finish partitioning and write changes to disk** option.
    - Write the changes to disks: Select the **Yes** option.

6. When asked to scan extra installation media, select **No**.

7. In the **Debian archive mirror country** selection section: Select the country closest to you.

8. In the **Debian archive mirror** selection section: Select the mirror you think is best i.e. `deb.debian.org`.

9. Leave the **HTTP proxy information** section blank and select **Continue**.

10. Participate in the package usage survey: Select **No**.

11. In the **Choose software to install** section:

    - Debian desktop environment: `Disabled`
    - GNOME: `Disabled`
    - SSH server: `Enabled`
    - Standard system utilities: `Enabled`
    - Select the **Continue** option.

12. When asked to **Install the GRUB boot loader to your primary drive**, select **Yes**.

13. For the **Device for boot loader installation**, select the disk you have installed Debian on i.e. `/dev/sda`.

14. Complete the installation.

### Configuration

#### Template VM

1. Login to the server using the service account credentials you have set during installation.

2. Switch to the root user:

    ```sh
    su -
    ```

    Enter the root password when prompted.

3. Update the system:

    ```sh
    apt update && apt upgrade -y
    ```

4. Install `sudo`:

    ```sh
    apt install -y sudo
    ```

5. Add your user to the `sudo` group:

    ```sh
    usermod -aG sudo <username>
    ```

    > [!NOTE]  
    > Replace `<username>` with the username you have set during installation.

6. Verify that the user has been added to the `sudo` group:

    ```sh
    id <username>
    ```

    > [!NOTE]  
    > Replace `<username>` with the username you have set during installation.

    Example output:

    ```sh
    uid=1000(<username>) gid=1000(<username>) groups=1000(<username>),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
    ```

7. Log out of the root user and back into your service user account by pressing <kbd>Ctrl + D</kbd>.

8. Log out of the server by pressing <kbd>Ctrl + D</kbd> once again and log back in to apply the changes.

9. Install the following packages:

    ```sh
    sudo apt install -y \
        curl \
        git \
        htop \
        nano \
        net-tools \
        openssh-server \
        python3 \
        python3-pip \
        ufw \
        wget
    ```

    On a VMware ESXi host, install the following package:

    ```sh
    sudo apt install -y open-vm-tools
    ```

    On a Proxmox host, install the following package:

    ```sh
    sudo apt install -y qemu-guest-agent
    ```

    > [!TIP]  
    > On Proxmox, you may also need to enable the `qemu-guest-agent` service yourself.

10. Clean up the system:

    ```sh
    sudo apt autoremove -y && sudo apt clean
    ```

11. Set up the VM's firewall.

   - Enable the firewall:

        ```bash
        sudo ufw enable
        ```

   - Check the firewall status:

        ```bash
        sudo ufw status numbered
        ```

   - Allow new SSH port:

        ```bash
        sudo ufw allow <port>/tcp
        ```

        > [!TIP]  
        > Replace `<port>` with the new port number you wish to use.

12. Update the VM's SSH configuration.

   - Update the SSH configuration file:

        ```sh
        sudo nano /etc/ssh/sshd_config
        ```

   - Change the default SSH port:

        ```sh
        #Port 22
        ```

       - Uncomment the `Port` line if it is commented.
       - Set the `Port` value to the new port number you have selected in the previous step.

   - Disable root login:

        ```sh
        #PermitRootLogin prohibit-password
        ```

       - Uncomment the `PermitRootLogin` line if it is commented.
       - Set the `PermitRootLogin` value to `prohibit-password` (good security) or `no` (best security).

   - Disable password authentication:

        ```sh
        #PasswordAuthentication yes
        ```

       - Uncomment the `PasswordAuthentication` line if it is commented.
       - Set the `PasswordAuthentication` value to `no`.

   - [Enable the SSH service](ssh.md#enable-remote-access) (in case it is not yet enabled).

   - Restart the SSH service:

        > [!WARNING]  
        > Make sure you have [copied your public SSH key(s)](ssh.md#copy-ssh-keys) to the server before doing this!

        ```sh
        sudo systemctl restart ssh
        ```

13. Reboot the VM to apply all changes:

    ```sh
    sudo reboot now
    ```

#### Individual VM

1. Login to the server using the service account credentials you have set during installation.

2. Set static IP address.

3. Update the system hostname.

4. Update the system.

### Adding a New Disk

TODO

### Expanding a Disk Partition

TODO

---

## Rocky Linux (RHEL)

### Description

This details the installation and configuration of Rocky Linux (RHEL) as the operating system of choice for a virtual machine.

### References

- [Download Rocky](https://rockylinux.org/download)
- [How To Install EPEL Repository (Repo) on an RHEL 8](https://www.cyberciti.biz/faq/install-epel-repo-on-an-rhel-8-x)
- [How to Install VMware Tools on CentOS/RHEL 8 VMware Virtual Machines](https://www.cloudhosting.lv/eng/faq/How-to-Install-VMware-Tools-on-CentOS-RHEL-8-VMware-Virtual-Machines)
- [firewalld for Beginners](https://docs.rockylinux.org/guides/security/firewalld-beginners)
- [SELinux security](https://docs.rockylinux.org/guides/security/learning_selinux)
- [error while starting sshd service on CentOS/RHEL](https://www.thegeekdiary.com/error-bind-to-port-2222-on-0-0-0-0-failed-permission-denied-error-while-starting-sshd-service-on-centos-rhel)

### Minimum Requirements

- CPU: `1`
- Memory: `1GB`
- Storage: `10GB`

### Installation

1. Pick the **Install Rocky Linux 8** option when given the option.

2. Go through the installation wizard as per usual.

3. In the **Network & Hostname** section, select the network interface you wish to use (i.e. `Ethernet (ens192)`), toggle the switch to **ON** to enable it, and click the **Done** button.

4. In the **Time & Date** section, update the **Region** and **City**, and click **Done**.

5. In the **Installation Destination** section, select the disk you wish to use for the installation, ensure that it has a checkmark, and click the **Done** button.

6. In the **Root Password** section, enter a secure password for the root user with a minimum of 12 characters, and click the **Done** button.

7. In the **User Creation** section:

    - Full name: Enter a name suitable for your homelab.
    - User name: Enter a username suitable for your homelab.
    - Make this user administrator: `Enabled`
    - Require a password to use this account: `Enabled`
    - Password: Enter a secure password for the user with a minimum of 12 characters.
    - Confirm password: Confirm the password you have set.
    - Click the **Done** button.

8. Click the **Begin Installation** button.

9. Click the **Reboot System** button once the installation is complete.

### Configuration

#### Template VM

1. Login to the server using the service account credentials you have set during installation.

2. Update the system:

    ```sh
    sudo yum update -y
    ```

3. Enable the EPEL repository:

    ```sh
    sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    ```

4. Install development tools:

    ```sh
    sudo yum groupinstall -y 'Development Tools'
    ```

5. Install the following packages:

    ```sh
    sudo yum install -y \
        curl \
        git \
        htop \
        nano \
        net-tools \
        openssh-server \
        policycoreutils-python-utils \
        python3 \
        python3-pip \
        wget
    ```

    > [!TIP]  
    > The `policycoreutils-python-utils` package is required for the `semanage` command, which is used to manage SELinux policy.

    On a VMware ESXi host, install the following package:

    ```sh
    sudo yum install -y open-vm-tools
    ```

    On a Proxmox host, install the following package:

    ```sh
    sudo yum install -y qemu-guest-agent
    ```

    > [!TIP]  
    > On Proxmox, you may also need to enable the `qemu-guest-agent` service yourself.

6. Clean up the system:

    ```sh
    sudo yum autoremove -y
    ```

7. Set up the VM's firewall.

   - Enable the firewall

        ```sh
        sudo systemctl enable --now firewalld
        ```

   - Check the firewall status

        ```sh
        sudo firewall-cmd --list-all
        ```

   - Disallow the default SSH port

        ```sh
        sudo firewall-cmd --remove-service=ssh
        ```

   - Allow the new SSH port

        ```sh
        sudo firewall-cmd --add-port=<port>/tcp
        ```

   - Save the firewall rules permanently

        ```sh
        sudo firewall-cmd --runtime-to-permanent
        ```

   - Reload the firewall

        ```sh
        sudo firewall-cmd --reload
        ```

        > [!TIP]  
        > Replace `<port>` with the new port number you wish to use.

8. Update the VM's SSH configuration.

   - Update the SSH configuration file:

        ```sh
        sudo nano /etc/ssh/sshd_config
        ```

   - Change the default SSH port:

        ```sh
        #Port 22
        ```

       - Uncomment the `Port` line if it is commented.
       - Set the `Port` value to the new port number you have selected in the previous step.

   - Disable root login:

        ```sh
        #PermitRootLogin yes
        ```

       - Uncomment the `PermitRootLogin` line if it is commented.
       - Set the `PermitRootLogin` value to `prohibit-password` (good security) or `no` (best security).

   - Disable password authentication:

        ```sh
        PasswordAuthentication yes
        ```

       - Uncomment the `PasswordAuthentication` line if it is commented.
       - Set the `PasswordAuthentication` value to `no`.

   - [Enable the SSH service](ssh.md#enable-remote-access) (in case it is not yet enabled).

   - Restart the SSH service:

        > [!WARNING]  
        > Make sure you have [copied your public SSH key(s)](ssh.md#copy-ssh-keys) to the server before doing this!

        ```sh
        sudo systemctl restart ssh
        ```

9. Reboot the VM to apply all changes:

    ```sh
    sudo reboot now
    ```

#### Individual VM

1. Login to the server using the service account credentials you have set during installation.

2. Set static IP address.

3. Update the system hostname.

4. Update the system.

### Adding a New Disk

TODO

### Expanding a Disk Partition

TODO
