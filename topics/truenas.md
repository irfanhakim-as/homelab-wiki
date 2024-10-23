# TrueNAS

## Description

TrueNAS is a family of network-attached storage (NAS) products produced by iXsystems, incorporating both FOSS, as well as commercial offerings.

## Directory

- [TrueNAS](#truenas)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Backup Configuration](#backup-configuration)
    - [Description](#description-1)
    - [References](#references-1)
    - [Manual](#manual)

## References

- [TrueNAS](https://www.truenas.com)
- [Wikipedia](https://en.wikipedia.org/wiki/TrueNAS)

---

## Backup Configuration

### Description

This details the process of backing up the TrueNAS system configuration.

### References

- [Backing Up System Configurations](https://www.truenas.com/docs/core/coretutorials/systemconfiguration/usingconfigurationbackups/#backing-up-system-configurations)

### Manual

1. Launch the TrueNAS web interface on a web browser.

2. On the left-hand side of the web interface, expand the **System** group and click the **General** menu item.

3. In the General view, click the **Save Config** button.

4. In the **Save Configuration** form, configure the following:

   - Export Secret Seed: Check this box to enable it
   - Root Password: Enter the administrator password of the TrueNAS server

    Click the **Save** button.

5. After the system configuration backup has been downloaded, store the archive in a secure location.
