# Backups

## Description

Backups are a critical part of any homelab setup. This course covers how to set up a backup solution for your homelab, including the backup server, backup client, and common backup operations.

## Directory

- [Backups](#backups)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Setting Up a Backup Server](#setting-up-a-backup-server)
    - [Borg Server Setup](#borg-server-setup)
  - [Setting Up a Backup Client](#setting-up-a-backup-client)
    - [Borg Client Setup](#borg-client-setup)
  - [Usage](#usage)
    - [Listing Backups](#listing-backups)
    - [Restoring from Backup](#restoring-from-backup)

## References

- [BorgBackup](https://www.borgbackup.org)

---

## Setting Up a Backup Server

This section details all topics pertaining to the setup and configuration of a backup server.

### Borg Server Setup

This details how to set up a Borg backup server to receive backups from Borg clients:

- [Docker](../topics/borg.md#install-borg-server-on-docker)

---

## Setting Up a Backup Client

This section details all topics pertaining to the setup and configuration of a backup client.

### Borg Client Setup

This details how to set up a Borg backup client to back up data to a Borg server on a schedule:

- [Docker](../topics/borg.md#install-borg-client-on-docker)
- [Helm](../topics/borg.md#install-borg-client-on-helm)

---

## Usage

This details some common usage steps for backups.

### Listing Backups

This details how to list the available backups in a backup repository:

- [Borg](../topics/borg.md#listing-archives)

### Restoring from Backup

This details how to restore data from a backup:

- [Borg](../topics/borg.md#restoring-from-backup)
