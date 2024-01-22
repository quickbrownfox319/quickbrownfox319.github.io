---
layout: post
title: "Updating VM Size with XCP-ng and Xen Orchestra"
---

This is a relatively short post, more like notes to myself about how to increase the size of a VM with XCP-ng and Xen Orchestra.

The reason I had to learn how to increase the size of a VM was because for some reason I made an Ubuntu server template with a virtual disk size of only 10 GB disk when I was first starting out.
Then I used that to make all my other VMs...derp.
I didn't need to use that much space at first, but over the years updates, logs, and old kernels eat up disk space and eventually things start crashing.

Let me be reemphasize, 10 GB is really not enough space to start off with when running critical infrastructure.
Luckily this is just for home learning and not a service that has zero tolerance for any down time.

Anyways this is for Ubuntu, so steps should be adjusted for whichever distro you're running with.


# Prerequisites

Back your stuff up!! Seriously, don't just wing it.

Bad things happen.

I have rolling monthly backups so in case something goes wrong I can restore things pretty quickly.
Back it up, and double check you backed it up.


# Increasing the Size of the VM

1. In Xen Orchestra, shut down the VM you're trying to increase the size of, as you can't change the size of the VDI while it's running.
2. Click on Disks, and under size, give it the total space amount you want it to have. This changes the "physical" size of the disk, but next you have to extend the file system size so it knows that it has more space to use.
3. Boot the VM now, and after logging in via console or SSH, check which partitions need to be updated with `lsblk` or `fdisk -l`. It's probably going to be something like `/dev/xvda2`, where `/dev/xvda1` is the boot partition.
4. Resize the disk with the handy tool resize2fs if it's installed, i.e. `resize2fs /dev/xvda2`.
5. Check your newly adjusted VM with `df -h` - it should reflect how much space you're using with your updated disk size.

Pretty easy and straightforward!


# Increasing the VDI Size without XOA

However, what happens if XOA is running in a VM that needs to get updated? This creates a sort of chicken and egg problem that I had the displeasure of dealing with, but ultimately just takes a few extra steps.

1. Connect to the host running XCP-ng. We'll be relying on `xe` commands.
2. Make note of the VM name and VM's UUID. In XOA, this can be in the URL, e.g. `http://xen-orchestra.homelab.lan/#/vms/<VM UUID>`, or on the host you can look through `xe vm-list` for the matching VM name.
3. Get the VM's disk UUID, `xe vm-disk-list uuid=<VM UUID>`. This should show the VDI UUID, similar to below:

```
Disk 0 VBD:
uuid ( RO)             : <VBD UUID>
    vm-name-label ( RO): xen-orchestra
       userdevice ( RW): 0


Disk 0 VDI:
uuid ( RO)             : <VDI UUID>
       name-label ( RW): xen-orchestra 0
    sr-name-label ( RO): Local storage
     virtual-size ( RO): 53687091200

```

Alternatively, you can try finding the VDI's UUID with `xe vdi-list`, but if you have a lot of backups and other VMs this can be kind of noisy.

3. Stop the VM, whether in XOA or via the host through the CLI if you know the command. THIS WILL KILL YOUR XOA GUI CONNECTION, obviously.
4. Expand the VDI size, with `xe vdi-resize uuid=<VDI UUID> disk-size=<new size for the VDI>`. Size can be denoted with `GiB`.
5. Start the VM again with either the VM name or VM UUID, i.e. `xe vm-start uuid=<VM UUID>`.
6. Follow steps to resize the filesystem from earlier. If you don't have `resize2fs`, you can use `fdisk`, similar to steps from [here from Citrix](https://support.citrix.com/article/CTX125405/how-to-extend-the-virtual-disk-size-of-a-xenvm).

And that should be it!


Additional resources:
* xe command line reference - https://docs.xcp-ng.org/appendix/cli_reference/
* Blog post for how to do this with LVM - https://rinzler.dk/posts/increasing-disk-xcp-ng-ubuntu/
