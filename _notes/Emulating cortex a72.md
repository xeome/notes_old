---
title: Emulating cortex a72
tags: #linux
toc: true
season: summer
date updated: 2022-08-21 14:35
---

# Starting out (Preparing for emulation)

- Make a sort of Project directory

```sh
$ mkdir rpi_image
$ cd rpi_image
```

- Download Debian RasPi4 Image & decompress

```sh
$ wget https://raspi.debian.net/tested/20220808_raspi_4_bookworm.img.xz
$ xz --decompress 20220808_raspi_4_bookworm.img.xz
```

- Find starting sector number via `fdisk`

```sh
$ fdisk -l 20220808_raspi_4_bookworm.img
```

```
Disk 20220808_raspi_4_bookworm.img: 1,95 GiB, 2097152000 bytes, 4096000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf7e489e2

Device         Boot  Start     End Sectors  Size Id Type
20220808_raspi_4_bookworm.img1        8192  819199  811008  396M  c W95 FAT32
20220808_raspi_4_bookworm.img2      819200 4095999 3276800  1,6G 83 Linux
```

- Before we mount the image to do some stuff, we need to get an offset in order to correctly mount.<br>In second partition `$something.img2`, find the `Start` number. In my case, it's `819200`. Multiply it by `512`, which in my case, it equals to `419430400`.
- Create a mount directory:

```sh
$ mkdir /mnt/raspi4
```

- We can now mount image:

```sh
$ sudo mount -o offset=419430400 20220808_raspi_4_bookworm.img /mnt/raspi4
```

- Now we can extract kernel and initrd from image:<br>(NOTE: We are cd'd into rpi_image directory)

```sh
$ cp /mnt/raspi4/vmlinuz .
$ cp /mnt/raspi4/initrd.img .
```

- Finally, we need to edit `fstab` for slicker(?) mounting via QEMU<br>BIG NOTE: Use your favorite text editor, smh.

```
$ nano /mnt/raspi4/etc/fstab
```

```sh
# The root file system has fs_passno=1 as per fstab(5) for automatic fsck.
LABEL=RASPIROOT / ext4 rw 0 1
# All other file systems have fs_passno=2 as per fstab(5) for automatic fsck.
LABEL=RASPIFIRM /boot/firmware vfat rw 0 2
```

- Replace `LABEL=RASPIROOT` with `/dev/vda2`

- Replace `LABEL=RASPIFIRM` with `/dev/vda1`

- File should look something like this

```sh
# The root file system has fs_passno=1 as per fstab(5) for automatic fsck.
/dev/vda2 / ext4 rw 0 1
# All other file systems have fs_passno=2 as per fstab(5) for automatic fsck.
/dev/vda1 /boot/firmware vfat rw 0 2
```

- Now we can convert image to qcow2

```sh
qemu-img convert -f raw -O qcow2 20220808_raspi_4_bookworm.img rpi.qcow2
```

# Emulation Time!

- We can finally start making our launch script.

```sh
$ nano rpistart.sh
```

**rpistart.sh**

```sh
#!/bin/bash
screen -mS raspberry-pi-4 \
sudo qemu-system-aarch64 \
-M virt \
-m 4096 -smp 4 \
-cpu cortex-a72 \
-kernel vmlinuz \
-initrd initrd.img \
-append "root=/dev/vda2 panic=1 rootfstype=ext4 rw" \
-hda rpi.qcow2 \
-no-reboot \
-nographic
```

- Paste the above into the script and save.

### Some info about script

- - _Since QEMU doesn't natively support Raspberry Pi 4(b), our only option is to virtualize Cortex A72 (Which is CPU used in Raspberry Pi 4(b))._
- - `-nographic` because _who needs graphics._
- - `screen` is used 'cuz _why not_.

<br>
* Make script executable
```sh
$ chmod +x rpistart.sh
```
* **And you should\* be able to run QEMU instance.** 
* LAUNCH!
```sh
$ ./rpistart.sh
```
# Some wacky reality.
You have booted into nice Debian. Oh, btw, username is passwordless `root`.<br>But there are two problems:
* Not enough space!
* No internet!

## Not enough space!

Very simple. Just:

- poweroff ze VM.

```sh
VM$ poweroff
```

- In our `rpi_image` directory, we can resize `qcow2` image via:

```sh
$ qemu-img resize rpi.qcow2 +4G
```

```sh
# Side note: You can also set exact size by getting rid of that + sign.
```

- Now we need to boot into VM once again:

```sh
$ ./rpistart.sh
```

- We have resized the image capacity, but not partition size. We can do that with:

```sh
VM$ resize2fs /dev/vda2
```

- You can check final result via

```sh
VM$ df -H
```

## No internet!

We gonna create interfaces! Just kidding, I have no idea what I'm doing, but this works, so ig it works.

- You can check available network interfaces via:

```sh
VM$ ip addr
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
```

- Notice, that `enp0s1` has zero IP addresses. We gotta fix that! _By creating another file._

```sh
VM$ nano /etc/network/interfaces.d/enp0s1
```

and paste this in:<br><br>
**enp0s1**

```sh
auto enp0s1

iface enp0s1 inet dhcp
```

- Reboot (Although config won't actually let you reboot.)

```sh
VM$ poweroff
```

```sh
$ ./rpistart.sh
```

- And voilá!
