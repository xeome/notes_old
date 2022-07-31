---
title: JomOS Settings
tags: #linux
toc: true
season: summer
date updated: 2022-07-31 20:01
---

# JomOS Settings

Documentation for configuration tweaks of JomOS

## /etc/sysctl.d/99-JomOS-settings.conf

### vm.swappiness

The swappiness sysctl parameter represents the kernel's preference (or avoidance) of swap space. Swappiness can have a value between 0 and 100, the default value is 60.

A low value causes the kernel to avoid swapping, a higher value causes the kernel to try to use swap space. Using a low value on sufficient memory is known to improve responsiveness on many systems.

This value is automatically calculated using your ram amount

### vm.vfs_cache_pressure

The value controls the tendency of the kernel to reclaim the memory which is used for caching of directory and inode objects (VFS cache).

Lowering it from the default value of 100 makes the kernel less inclined to reclaim VFS cache (do not set it to 0, this may produce out-of-memory conditions)

This value is automatically calculated using your ram amount

### vm.page-cluster

refer to <https://xeome.github.io/notes/Zram#page-cluster-values-latency-difference>

### vm.dirty_ratio

Contains, as a percentage of total available memory that contains free pages and reclaimable pages, the number of pages at which a process which is generating disk writes will itself start writing out dirty data (Default is 20).

### vm.dirty_background_ratio

Contains, as a percentage of total available memory that contains free pages and reclaimable pages, the number of pages at which the background kernel flusher threads will start writing out dirty data (Default is 10).

### Network tweaks (TODO)

### kernel.nmi_watchdog

Disabling NMI watchdog will speed up your boot and shutdown, because one less module is loaded. Additionally disabling watchdog timers increases performance and lowers power consumption
