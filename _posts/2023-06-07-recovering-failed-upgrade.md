---
layout: post
title: "Recovering a Failed Upgrade with an Encrypted LUKS Partition"
---

So a few weeks ago I decided to do the responsible thing and upgrade my Ubuntu distro from 20.04 to 22.04 because it's been long enough.
As you might guess from the title, things did not go quite as planned and I had to force shutdown my laptop while attempting the upgrade.
You might guess correctly, dear reader, that this caused some issues.
Booting my laptop up displayed a kernel panic.
Trying to switch to the recovery kernel also resulted in kernel panic.
Then I started to panic.
The rest of this blog is to hopefully help you in a similar situation.
I can't take credit for being the source of any wisdom here, but this is a rough summary of my steps for recovering my encrypted LUKS partition data and reviving my upgrade.


# Backing up Data

So this actually should have been step #0 if you aren't a dummy like me.
In my hubris I decided I didn't need no dang backup because I knew what I was doing and this was a simple upgrade.
Wrong wrong wrong wrong. DO IT ANYWAYS. I preach the [3-2-1 rule](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/) and I acknowledge the hypocrisy.

Do this first if you haven't started upgrading anything.


# Recovering Data to Backup

I first wanted to recover whatever important data I had on my disk in the first place. To get that data, I created a live USB with Ubuntu to boot from, which I already had lying around.
You can create one easily enough (if you have another computer to do it) using something like [Ubuntu's startup disk creator](https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview).

Once you have a live USB to boot from, boot from the USB and mount the encrypted LUKS partition.
I have to thank this [post](https://unix.stackexchange.com/questions/445652/how-to-mount-and-de-encrypt-a-luks-encrypted-partition-to-recover-files) for all the steps, but I'll reiterate it below.

```
# open the LUKS container
sudo cryptsetup open /dev/nvme1n1p3 luksrecoverytarget --type luks

# enter the password to decrypt

# next goal is to mount the drive that's mapped to /dev/mapper/luksrecoverytarget

# sync logical volumes
sudo vgscan

# display logical volumes
sudo lvdisplay # find the one that's yours by lv path

# make a directory to mount to
sudo mkdir /mnt/recoverytarget

# mount the logical volume
sudo mount <LV_PATH> /mnt/recoverytarget
```

At this point you should have access to your disk at `/mnt/recoverytarget`, and you can copy your data safely out to something more...recoverable.

Once you're done, clean up with:
```shell
# unmount the drive
sudo umount /mnt/recoverytarget

# change the logical volume
sudo vgchange -an

# close LUKS partition
sudo cryptsetup close luksrecoverytarget
```

You probably _should_ do a fresh install after this, while watching everything carefully.
It'll help avoid random things breaking for seemingly no reason in the future, but if you're masochist or want to really keep whatever you had, you can try recovering the failed upgrade.


# Fixing the Failed Upgrade

When I tried booting it would kernel panic, and the recovery kernel also resulted in a kernel panic as well.
This was a bad situation that I think was caused by carelessly abusing `autoremove` to clear up space in `/boot`, as the upgrade couldn't have continued with the space I had.
According to this [askubuntu response by lumbric](https://askubuntu.com/questions/567730/gave-up-waiting-for-root-device-ubuntu-vg-root-doesnt-exist), this possibly removed all the useful `cryptsetup` related functions that would have helped me unlock my LUKS partition. 

To summarize that post, I also had to again take out the liveboot Ubuntu drive and create a chroot environment, where I could install the cryptsetup package.
Below are the relevant commands copied from the post. Something additional I did was run `apt update` and `apt upgrade` while in the chroot environment, which helped finish the stuck upgrade to 22.04 as well as fix the faulty kernel issue.

```bash
# find root partition
sudo fdisk -l

# unencrypt partition
#   Note: replace /dev/nvme0n1p3 with your disk
#         replace "nvme0n1p3_crypt" with the correct name 
#         check by running this in chroot:
#         $ cat /etc/crypttab | cut -f1 -d " "
#         nvme0n1p3_crypt
sudo cryptsetup luksOpen /dev/nvme0n1p3 nvme0n1p3_crypt

# mount root partition
sudo vgscan 
sudo vgchange -ay
sudo mount /dev/mapper/xubuntu--vg-root /mnt

# prepare chroot environment
sudo mount /dev/nvme0n1p2 /mnt/boot/   # replace nvme0n1p2 with your boot partition!
sudo mount -o rbind /dev/ /mnt/dev/
sudo mount -t proc proc /mnt/proc/
sudo mount -t sysfs sys /mnt/sys/

# make dns available in chroot
sudo cp /etc/resolv.conf  /mnt/etc/resolv.conf 

# enter chroot
sudo chroot /mnt /bin/bash

# finish the update and upgrade process
apt update
apt upgrade

# re-install missing packages
apt install cryptsetup lvm2

# re-generate  (this might be done also by apt in the step before, I'm not sure)
update-initramfs -u -k all

# Leave chroot environment - not sure if the following is really necessary...
exit

# Write buffers to disk
sudo sync

# Unmount file systems
sudo umount /mnt/sys
sudo umount /mnt/proc
sudo umount /mnt/boot
```

Now, when rebooting I was able to drop into an initramfs shell, which is way better than the kernel panic I had before.
If you're here, you can unlock the LUKS partition directly with cryptsetup directly in initramfs and continue with the boot process as normal.

```bash
(initramfs) cryptsetup luksOpen /dev/nvme1n1p3 ubuntu
            Enter passphrase: *****
(initramfs) exit
```

This is because initramfs doesn't know quite know where your `ubuntu-vg-root` is located in the partition by default, so you have to provide that.
Make sure the UUID is correct in `/etc/crypttab`, and shows something like `nvme1n1p3_crypt UUID=<disk with LUKS UUID> none luks`.
You can find this with `blkid`:

```shell
$ sudo blkid
> ...
> /dev/nvme1n1p3: UUID="<the UUID you're looking for>" TYPE="crypto_LUKS" PARTUUID="<whatever part UUID1234>"
> ...
```

This [link](https://askubuntu.com/questions/1268943/after-upgrade-to-20-04-luks-doesnt-open-my-disk-on-boot-cryptroot-crypttab-is) and [this link](https://gist.github.com/samuelcolvin/43c5ed2807e7db004b1058d0c9bfb068) helped me with this part.

However, I like using the grub bootloader instead of initramfs each time to unlock my partition.
I had to install grub to my boot partition with `grub-install /dev/nvme1n1p2`, though replace the partition with your actual boot partition.
This didn't show me grub options, but it did at least drop me into screen to unlock the LUKS partition.
To get the grub menu back, edit the grub conf at `/etc/default/grub`, change `GRUB_TIMOUT` to something other than 0, and add `GRUB_TIMEOUT_STYLE=menu`.
This should bring back your familiar grub options.

And that should (hopefully) fix it!
Hopefully this has been helpful for providing resources debugging your failed Ubuntu upgrade.
Send me an email if there are corrections or questions.
