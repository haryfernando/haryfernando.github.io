---
layout: post
lang: en
title: Virtualbox Raw Disk Access
---
My current laptop is dual boot OS and normally, I use linux as my daily OS. 

<!-- more -->

Sometimes there are some applications that need to be run on Microsoft Windows. 
So I use virtualbox and I create new virtualbox machine along with virtualbox disk.

Later I realize that why don't I use the Microsoft Windows from the disk. 
You know, my laptop came with original Windows 7 Home Edition. After that, I install fedora 19 until now it became fedora 21.

The using of physical drive in virtualbox is called virtualbox raw disk access. You can google them, hehe.

Here is my disk partition layout :

    Disk /dev/sda: 232.9 GiB, 250059350016 bytes, 488397168 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x76692ca8

    Device     Boot     Start       End   Sectors   Size Id Type
    /dev/sda1            2048  30715903  30713856  14.7G 83 Linux
    /dev/sda2  *     30716280 114602791  83886512    40G  7 HPFS/NTFS/exFAT
    /dev/sda3       114604032 480004095 365400064 174.2G  7 HPFS/NTFS/exFAT
    /dev/sda4       480006142 488396799   8390658     4G  5 Extended
    /dev/sda5       480006144 488396799   8390656     4G 82 Linux swap / Solaris

I know, it's an old laptop, the disk is only 250G, but it's been great to have it. 

Okay, let's cut to the chase.

Here are my steps to create virtualbox machine with raw disk

First, create bootloader :

    dd if=/dev/sda2 of=boot.mbr bs=512 count=1


create virtualbox :

    VBoxManage createvm --name windows7 --ostype Windows7_64 --register


add settings, such as memory, NIC :

    VBoxManage modifyvm "windows7" --memory 1024 --acpi on --boot1 disk --nic1 nat

set IDE controller use ICH6 :

    VBoxManage storagectl "windows7" --name "IDE Controller" --add ide --controller ICH6


create the vmdk :

    VBoxManage internalcommands createrawvmdk -filename /home/hary/windows7.vmdk -rawdisk /dev/sda -partitions 2 -mbr /home/hary/boot.mbr -relative


add the storage to our virtualbox :

    VBoxManage storageattach windows7 --storagectl "IDE Controller"  --port 0 --device 0 --type hdd --medium /home/hary/windows7.vmdk


We will need ISO/DVD of Windows installer. 

Mount it and make sure it to become the first boot of our virtualbox.

Start the virtualbox and boot to DVD, then open Windows command line by pressing `SHIFT+F10`. 

After that, type:

    bootrec /FixMbr
    reg load HKLM\Computer_System C:\Windows\system32\config\system
    regedit

Go to :

    HKEY_LOCAL_MACHINE\Computer_System\MountedDevices

and write down the value of `"\DosDevices\C:"`. 

For example :

    ab cd ef gh

Then type:

    diskpart
    DISKPART> select disk 0
    DISKPART> uniqueid disk id=ghefcdab
    DISKPART> exit

Now reboot to our virtualbox, enjoy.
