# Backups

## Description

Backups are a critical part of any homelab setup. This course covers how to set up a backup solution for your homelab, including the backup server, backup client, and common backup operations.

## Directory

- [Backups](#backups)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Backup Setup](#backup-setup)
    - [Borg](#borg)
  - [Usage](#usage)
    - [Triggering a Backup](#triggering-a-backup)
    - [Listing Backups](#listing-backups)
    - [Restoring from Backup](#restoring-from-backup)

## References

- [Backup](https://en.wikiversity.org/wiki/Backup)

---

## Backup Setup

This section details all topics pertaining to the setup and configuration of a backup solution.

### Borg

This details how to set up and use Borg for backing up data from any mounted storage and database dumps to a remote Borg server:

- [Server](../topics/borg.md#borg-server)
- [Client](../topics/borg.md#borg-client)

---

## Usage

This details some common usage steps for backups.

### Triggering a Backup

This details how to trigger an on-demand backup outside of the regular schedule:

- [Borg](../topics/borg.md#running-a-manual-backup)

### Listing Backups

This details how to list the available backups in a backup repository:

- [Borg](../topics/borg.md#listing-archives)

### Restoring from Backup

This details how to restore data from a backup:

- [Borg](../topics/borg.md#restoring-from-backup)
