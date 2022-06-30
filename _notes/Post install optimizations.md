---
title: Post install optimizations
tags: #linux
toc: true
season: summer
---
Links: [[Linux]]
# Post install optimizations

### Editing mkinitcpio.conf for faster boot times
Replace base and udev with systemd for faster boots and set compression algorithm to zstd and compression level to -2 because compression ratio increase isn't worth the increased latency

(bellow isnt the whole file, just the parts that needs changes)
```ini
HOOKS="systemd autodetect modconf block keyboard keymap consolefont filesystems"

COMPRESSION="zstd"

COMPRESSION_OPTIONS=(-2)
```
