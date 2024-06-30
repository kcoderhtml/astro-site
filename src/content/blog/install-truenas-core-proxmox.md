---
title: Install TrueNAS Core on Proxmox
slug: install-truenas-core-proxmox
date: July 10, 2023
---

# Install TrueNAS Core on Proxmox

![photo of a hardrive](https://assets.vrite.io/64974cb888e8beebeb2c925b/mSouawQas8oyw6g5VsNlH.jpeg)

## Introduction

> Note: I have since found out that running TrueNAS in a VM passing through the drives may not be a very good solution as it essentially just creates a large virtual disk that is the size of the drive you are passing through. Because of this I will not be using this setup in my homelab and will instead create a large ZFS pool on Proxmox. However if you are fine with those downsides, then have fun and enjoy the tutorial.

To install [TrueNAS Core](https://www.truenas.com/download-truenas-core/#) on [Proxmox](https://www.proxmox.com/en/proxmox-ve) you need three things:

1.  A copy of [Proxmox](https://www.proxmox.com/en/proxmox-ve) — A complete, open-source server management platform for enterprise virtualization.
    
2.  A [TrueNAS CORE](https://www.truenas.com/download-truenas-core/#) ISO — World’s #1 NAS Operating System
    
3.  HDDs — You can use whatever you want, but I will be using three Barracuda ES 750GB drives
    

## Install TrueNAS Core

Sign-in to Proxmox and upload your ISO to the local storage or, download the file directly from the link using the built-in ISO fetcher.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/Ww212rUDQ_Ms9P2WhJMwz.png)

Next to create the VM, the only thing that needs to be changed from the defaults is the memory, which I set to `8192 MB` (8 GB).

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/bDT9VdIMMG1LWvv1RwNKl.png)

Now finish creating the VM and click on the VM after it is created. Go to options and enable start at boot.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/ImxOHWJNuRL3yiF12cQfe.png)

Next, we need to pass through the physical drives to the VM. Open a terminal on the Proxmox server (use the built-in terminal or ssh in) and run the following command. Only run the part after the #.
```bash
root@thespia:~# lsblk -o +MODEL,SERIAL

NAME                           MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT MODEL                   SERIAL
sda                              8:0    0 698.6G  0 disk            ST3750330NS             9QK2GT8R
sdb                              8:16   0 698.6G  0 disk            ST3750640NS             3QD0AYE0
sdc                              8:32   0 698.6G  0 disk            ST3750640NS             3QD0BQ5G
sdd                              8:48   1 111.8G  0 disk            Hitachi_HTS543212L9SA02 090130FBEB00LGGJ35RF
├─sdd1                           8:49   1  1007K  0 part
├─sdd2                           8:50   1   512M  0 part /boot/efi
└─sdd3                           8:51   1 111.3G  0 part
    ├─pve-swap                   253:0    0     8G  0 lvm  [SWAP]
    ├─pve-root                   253:1    0  37.8G  0 lvm  /
    ├─pve-data_tmeta             253:2    0     1G  0 lvm
    │ └─pve-data-tpool           253:4    0  49.6G  0 lvm
    │   ├─pve-data               253:5    0  49.6G  1 lvm
    │   ├─pve-vm--100--cloudinit 253:6    0     4M  0 lvm
    │   └─pve-vm--101--cloudinit 253:7    0     4M  0 lvm
    └─pve-data_tdata             253:3    0  49.6G  0 lvm
    └─pve-data-tpool           253:4    0  49.6G  0 lvm
        ├─pve-data               253:5    0  49.6G  1 lvm
        ├─pve-vm--100--cloudinit 253:6    0     4M  0 lvm
        └─pve-vm--101--cloudinit 253:7    0     4M  0 lvm
sde                              8:64   0 465.8G  0 disk            WDC_WD5000AAKS-65YGA0   WD-WCAS83511331
├─sde1                           8:65   0 465.8G  0 part
└─sde9                           8:73   0     8M  0 part
zd0                            230:0    0    32G  0 disk
├─zd0p1                        230:1    0   100M  0 part
├─zd0p2                        230:2    0    16M  0 part
├─zd0p3                        230:3    0  31.4G  0 part
└─zd0p4                        230:4    0   522M  0 part
zd16                           230:16   0    80G  0 disk
├─zd16p1                       230:17   0     1M  0 part
└─zd16p2                       230:18   0    80G  0 part
zd32                           230:32   0     4M  0 disk
zd48                           230:48   0    80G  0 disk
├─zd48p1                       230:49   0     1M  0 part
└─zd48p2                       230:50   0    80G  0 part
zd64                           230:64   0     1M  0 disk
zd80                           230:80   0    32G  0 disk
```

In my server `sda, sdb, and sdc` are my drives. I can tell because they have no partitions and are `698.6G`. Next, based on the serial numbers of the disks, find the `dev/disk/by-id` of the drive.
```bash
root@thespia:~# ls /dev/disk/by-id/

ata-Hitachi_HTS543212L9SA02_090130FBEB00LGGJ35RF
ata-Hitachi_HTS543212L9SA02_090130FBEB00LGGJ35RF-part1
ata-Hitachi_HTS543212L9SA02_090130FBEB00LGGJ35RF-part2
ata-Hitachi_HTS543212L9SA02_090130FBEB00LGGJ35RF-part3
ata-ST3750330NS_9QK2GT8R
ata-ST3750640NS_3QD0AYE0
ata-ST3750640NS_3QD0BQ5G
ata-WDC_WD5000AAKS-65YGA0_WD-WCAS83511331
ata-WDC_WD5000AAKS-65YGA0_WD-WCAS83511331-part1
ata-WDC_WD5000AAKS-65YGA0_WD-WCAS83511331-part9
dm-name-pve-root
dm-name-pve-swap
dm-name-pve-vm--100--cloudinit
dm-name-pve-vm--101--cloudinit
dm-uuid-LVM-i2jw2DEc8aJxdhf3mg7sAcAbc57lfeNL967xBhsO2KsTDqSJ5KB9pGqef5HjQJHk
dm-uuid-LVM-i2jw2DEc8aJxdhf3mg7sAcAbc57lfeNLQ6hkWGll1H38yFz0ty3RmmJPSRSbj1sa
dm-uuid-LVM-i2jw2DEc8aJxdhf3mg7sAcAbc57lfeNLrSofGgZtL41un6baoCpRHunOrbJeMTeO
dm-uuid-LVM-i2jw2DEc8aJxdhf3mg7sAcAbc57lfeNLWPjyk8d4ik2D6KIcp2zaugdFsHB4TNOM
lvm-pv-uuid-pRECVX-zqKA-evrD-PNof-sTYg-zNrD-WUelFe
usb-Generic-_Multi-Card_20120926571200000-0:0
wwn-0x5000c50015a53388
wwn-0x5000cca562c751e4
wwn-0x5000cca562c751e4-part1
wwn-0x5000cca562c751e4-part2
wwn-0x5000cca562c751e4-part3
wwn-0x50014ee2ab77b23f
wwn-0x50014ee2ab77b23f-part1
wwn-0x50014ee2ab77b23f-part9
```

In my case, the ID of the drives I want are `ata-ST3750330NS_9QK2GT8`, `ata-ST3750640NS_3QD0AYE0`, and `ata-ST3750640NS_3QD0BQ5G`.

Now find your VM\_ID, mine is 102.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/gwjgFbI5IrnJSTLTB0PeX.png)

Run the following command, replacing the VM\_ID and DISK\_ID with yours.

```bash
# qm set VM_ID -scsi1 /dev/disk/by-id/DISK_ID

root@thespia:~# qm set 102 -scsi1 /dev/disk/by-id/ata-ST3750330NS_9QK2GT8R
update VM 102: -scsi1 /dev/disk/by-id/ata-ST3750330NS_9QK2GT8R
root@thespia:~# qm set 102 -scsi2 /dev/disk/by-id/ata-ST3750640NS_3QD0AYE0
update VM 102: -scsi2 /dev/disk/by-id/ata-ST3750640NS_3QD0AYE0
root@thespia:~# qm set 102 -scsi3 /dev/disk/by-id/ata-ST3750640NS_3QD0BQ5G
update VM 102: -scsi3 /dev/disk/by-id/ata-ST3750640NS_3QD0BQ5G
```

Here is how it appears in Proxmox:

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/QBpmqflEHmPiUHbd8JVk2.png)

If everything went well, then you can start your VM now. After it finishes booting up, you will get the screen below. Make sure Install/Upgrade is selected and hit enter.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/UFqhrRdD3GkP1_No5lWaj.png)

You will then get this screen, use space to select the first drive and hit enter.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/xD5QxmFtHxw10p624FgwM.png)

Hit enter one last time and enter your password.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/MZy3mN1cXBaicgVolVYs5.png)![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/KWq2P7Iok9LThOF5Xoj6l.png)

Select BIOS, as this is the default mode for Proxmox VMs.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/RfXwEGx6oug1vVF3UZuCj.png)

After about five to ten minutes, the installation process will finish and the VM will ask you to remove installation media and reboot.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/mFEH-FHY10H7NUAvYi0aE.png)![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/JPXkEQJgBmeATEE40HHpr.png)

Select the installation media and remove it with the top button, go back to the console and hit enter, which will take you back to the main menu. On the main menu, select reboot with the arrow keys and hit enter.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/IfvdMuF6AVU_f0-_rngqq.png)

Once the machine restarts, it will display an IP address in the console.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/K8yoM1FcPpGGhxOnE7_DX.png)

Upon connecting to the IP address, you will get this screen. Use the root username and the password, previously configured, to login.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/ghvCsvwAJMudUCUGvcnCu.png)

Once logged in, I updated the system using the button on the home screen.

![undefined](https://assets.vrite.io/64974cb888e8beebeb2c925b/nrBop3a9ilvuc7h-0WPEG.png)

I chose not to save the configuration file when prompted, proceeded to install the updates, and rebooted.

I hope you enjoyed the tutorial! My inspiration to make this came from watching [“How to run TrueNAS on Proxmox?”](https://www.youtube.com/watch?v=M3pKprTdNqQ) by [Christian Lempa](https://www.youtube.com/@christianlempa). I encourage you to watch his video if you want a video guide to installing TrueNAS on Proxmox.
