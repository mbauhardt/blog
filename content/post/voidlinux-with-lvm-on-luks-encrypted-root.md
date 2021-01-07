---
title: "Voidlinux With Lvm on Luks Encrypted Root"
date: 2021-01-07T08:51:37+01:00
series: ["void-linux"]
---

## Introduction
This guide gives you an idea how to install voidlinux onto LVM with
luks encrypted root partition but with a separate unencrypted efi and
boot partition. I'm using *x86_64-musl* instead glibc. Instead */dev/sda1*,
my SSD is a NVM based one, so the device i'm using is */dev/nvme0n1*.
The laptop has 32G RAM.

First, starting the live installer, login as root and do the following.

## Filesystem

		$ wipeout --all /dev/nvme0n1
		$ cfdisk /dev/nvme0n1

The goal is to have four partitions.

* EFI -			/dev/nvme0n1p1, EFI System, 1G
* BOOT -		/dev/nvme0n1p2, Linux Filesystem, 1G
* Void -		/dev/nvme0n1p3, Linux Filesystem, 300G
* Free partition for another OS e.g. NetBSD

Let's create filesystems

		$ mkfs.vfat -nBOOT -F32 /dev/nvme0n1p1
		$ mkfs.ext2 -L grub /dev/nvme0n1p2
		$ cryptsetup luksFormat --type=luks -s=512 /dev/nvme0n1p3
		$ cryptsetup open /dev/nvme0n1p3 void-vg
		$ vgcreate void-vg /dev/mapper/void-vg
		$ lvcreate --name root-lv -L 52G void-vg
		$ lvcreate --name swap-lv -L 48G void-vg
		$ lvcreate --name home-lv -L 180G  void-vg
		$ mkfs.xfs -L root /dev/void-vg/root-lv
		$ mkfs.xfs -L home /dev/void-vg/home-lv
		$ mkswap /dev/void-vg/swap-lv


## EFI and Boot partition

		$ mkdir /mnt/efi
		$ mount -o rw,noatime /dev/nvme0n1p1 /mnt/efi
		$ mkdir /mnt/boot
		$ mount -o rw,noatime /dev/nvme0n1p2 /mnt/boot


## Setup Chroot and Installing Base System

		$ mount /dev/void-vg/root-lv /mnt
		$ for dir in dev proc sys run; do mkdir -p /mnt/$dir ; mount --rbind /$dir /mnt/$dir ; mount --make-rslave /mnt/$dir ; done
		$ mkdir -p /mnt/home
		$ mount /dev/void-vg/home-lv /mnt/home
		$ XBPS_ARCH=x86_64-musl xbps-install -Sy -R https://alpha.de.repo.voidlinux.org/current/musl -r /mnt base-system cryptsetup grub-x86_64-efi lvm2


## Chroot

		$ chroot /mnt
		$ chown root:root /
		$ chmod 755 /
		$ passwd root
		$ echo voidvm > /etc/hostname
		$ echo "LANG=en_US.UTF-8" > /etc/locale.conf
		$ echo "en_US.UTF-8 UTF-8" >> /etc/default/libc-locales

Edit */etc/fstab*

		# <file system>          <dir>      <type>     <options>                <dump>  <pass>
		tmpfs                    /tmp       tmpfs      defaults,nosuid,nodev    0       0
		/dev/void-vg/root-lv     /          xfs        defaults                 0       0
		/dev/void-vg/home-lv     /home      xfs        defaults                 0       0
		/dev/void-vg/swap-lv     swap       swap       defaults                 0       0
		/dev/nvme0n1p1           /efi       vfat       defaults                 0       0
		/dev/nvme0n1p2           /boot      ext2       defaults                 0       0


## Configure and Install GRUB

Add the following line to */etc/default/grub*

		GRUB_ENABLE_CRYPTODISK=y

Find the UUID of the root device

		$ blkid -o value -s UUID /dev/nvme0n1p3

Edit the */etc/default/grub* and edit line *GRUB_CMDLINE_LINUX_DEFAULT=*
and append at the end the following

		rd.lvm.vg=voidvm rd.luks.uuid=<UUID>

Make sure the UUID matches the one for the *nvme0n1p3*.

Edit the */etc/default/grub* and edit line *GRUB_CMDLINE_LINUX_DEFAULT=*
and append at the end the following

		resume=/dev/mapper/void--vg-swap--lv

Install Grub

		$ grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id="Void Linux"

Regenerate the configs

		$ xbps-reconfigure -fa


## Reboot

		# exit the chroot
		$ exit
		$ reboot


## Resources

* https://docs.voidlinux.org/installation/guides/fde.html
* https://gist.github.com/gbrlsnchs/9c9dc55cd0beb26e141ee3ea59f26e21


