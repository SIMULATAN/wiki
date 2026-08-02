---
title: RAID
description: 
published: true
date: 2026-08-02T16:11:22.250Z
tags: 
editor: markdown
dateCreated: 2026-08-02T16:11:22.250Z
---

# GRUB Setup using Software RAID
When using Software RAID, the BIOS still only sees two physical disks and attempts to boot from either. Therefore, installing on the mapped `mdXXX` disk will leave the system unbootable.

To fix this:

1. Boot into systemrescue (use a normal mode, we will need to mount the mapped RAID partitions)
2. Mount the necessary file systems and install GRUB:
   ```bash
   lsblk
   # mount your rootdisk to read the configurations & use the appropriate libraries
   # find out the correct name of your root partition using above command
   mount /dev/mdXXX /mnt
   # check the fstab, mount all necessary partitions under `/mnt`
   # for example, if using a separate `/boot` partition, mount it as `/mnt/boot`
   cat /mnt/etc/fstab
   mount ... /mnt/...

   # mount necessary system stuff from the host to `/mnt` to prepare chroot
   mount --bind /dev /mnt/dev
   mount --bind /dev/pts /mnt/dev/pts
   mount -t proc /proc /mnt/proc
   mount -t sysfs /sys /mnt/sys

   chroot /mnt
   
   # in chroot, install GRUB to all disks your root RAID partition is made of
   > grub-install /dev/sda
   > grub-install /dev/sdb
   > exit

   # make sure changes were written to the disk
   sync
   # unmount everything manually to prevent pending writes and other woes
   umount /mnt/dev/pts
   umount /mnt/dev
   umount /mnt/proc
   umount /mnt/sys
   umount /mnt

   reboot
   ```
3. After the reboot, in case the system refuses to boot initially: don't panic. Short press the power button, wait until the device powers off and remove the recovery USB. Power it on again - it should work now.