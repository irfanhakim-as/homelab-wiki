# sSMTP

## Description

sSMTP is a programme which delivers email from a local computer to a configured mailhost (mailhub). It is not a mail server (like feature-rich mail server `sendmail`) and does not receive mail, expand aliases or manage a queue.

## Directory

- [sSMTP](#ssmtp)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setup](#setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)

## References

- [ArchWiki](https://wiki.archlinux.org/title/SSMTP)

---

## Setup

### Description

This details the installation and setup process for sSMTP on a Linux system.

### References

- [Installation](https://wiki.archlinux.org/title/SSMTP#Installation)

### Steps

1. [Install](package-manager.md#install-software) the `ssmtp` package using the system's package manager (i.e. `apt`).

2. Backup the existing sSMTP configuration file:

    ```sh
    sudo cp /etc/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf.bak
    ```

3. Update the network configuration file:

    ```sh
    sudo nano /etc/ssmtp/ssmtp.conf
    ```

    Add the following configuration to the end of the file:

    ```sh
    root=<email-address>

    mailhub=smtp.gmail.com:465

    rewriteDomain=gmail.com

    TLS_CA_FILE=/etc/ssl/certs/ca-certificates.crt
    UseTLS=Yes
    UseSTARTTLS=No

    AuthUser=<email-address>
    AuthPass=<email-password>
    AuthMethod=LOGIN
    ```

    Replace `<email-address>` and `<email-password>` with your own email address (i.e. `user@example.com`) and password (i.e. [app password](https://support.google.com/mail/answer/185833)) respectively. You may need to configure these a little differently according to your own email provider.

4. Ensure sSMTP has been configured correctly by sending a test email:

    ```sh
    echo -e "Subject: Test Email\n\nThis is a test email" | sudo sendmail <receiver-email>
    ```

    Replace `<receiver-email>` with the email address you wish to receive the test email from (i.e. `admin@example.com`).
