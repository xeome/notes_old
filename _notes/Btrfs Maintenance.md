---
title: Btrfs Maintenance
tags: #linux
toc: true
season: summer
date updated: 2022-07-05 00:42
---

Links: [[Linux]], [[Post install optimizations]]

# Btrfs Maintenance

Scrubbing reads all data and metadata from devices and verifies checksums. It is not mandatory, but it may detect problems with faulty hardware early because it touches data that may not be in use and causes bit rot.

If there is data/metadata redundancy, such as DUP or RAID1/5/6 profiles, scrub can automatically repair the data if a good copy is available.

You can use `sudo btrfs scrub start /` to start the scan and `sudo btrfs scrub status /` to check the status of it.  
TODO: add more maintenance functions