---
layout: post
title: Full Disk Encryption In FreeBSD 9.1
---

{{page.title}}

<p class="meta">24 Jan 2013</p>

Introduction 

We'll be using [GELI](https://en.wikipedia.org/wiki/Geli_%28software%29) as the encryption and [UFS](https://en.wikipedia.org/wiki/Unix_File_System) as the file system, all inside a 64GB <abbr title="Solid-state drive">SSD</abbr>. Please note that when I say full disk encryption it means everything but the /boot partition. For this article and in my case you'll be using the **/dev/ada0** disk. Two partitions; a 10G /boot and the rest of the disk as **/**. Why no swap you ask? In most recent machines there is enough RAM to not need any, top that with using a <abbr title="Solid-state drive">SSD</abbr> and it is an easier choice to make.

Take a look at the [Release notes](http://www.freebsd.org/releases/9.1R/announce.html), [Handbook](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/), [Wiki](https://wiki.freebsd.org/), and [Forums](http://forums.freebsd.org/) for more information regarding FreeBSD. I also suggest taking a glance at the following man pages ( [tunefs](http://www.freebsd.org/cgi/man.cgi?query=tunefs&apropos=0&sektion=0&manpath=FreeBSD%2B9.0-RELEASE&arch=default&format=ascii), [newfs](http://www.freebsd.org/cgi/man.cgi?query=newfs&apropos=0&sektion=0&manpath=FreeBSD%2B9.0-RELEASE&arch=default&format=ascii), [geli](http://www.freebsd.org/cgi/man.cgi?query=geli&apropos=0&sektion=0&manpath=FreeBSD%2B9.0-RELEASE&arch=default&format=ascii), [gpart](http://www.freebsd.org/cgi/man.cgi?query=gpart&apropos=0&sektion=0&manpath=FreeBSD%2B9.0-RELEASE&arch=default&format=ascii) ) for the commands we'll be using, just in case there is something extra your system/setup requires.

Procedure 

[Download](http://www.freebsd.org/where.html), [prepare the installation media](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/bsdinstall-pre.html), then boot into it. The next few steps are really straight forward and the handbook does a pretty good job explaining its options. In the "Welcome" menu choose "Install", choose your keyboard layout, then type in your hostname; *pcname.localhost.home* or something resembling that format. Choose the optional system components as you wish, the default options are fine. Finally, when it comes to the "Partitioning", select "Shell". Inside the shell you'll be typing the commands below. 

Partitioning 

    # gpart show   
    # gpart destroy -F ada0   
    # gpart create -s gpt ada0   
    # gpart create -s gpt ada0   
    # gpart add -s 128 -t freebsd-boot ada0   
    # gpart add -s 10G -t freebsd-ufs ada0   
    # gpart add -t freebsd-ufs ada0   
    # gpart bootcode -b /boot/pmbr -p /boot/gptboot -i 1 ada0   
    # newfs -E -t -U -m 0 -j /dev/ada0p2   
    # geli init -bl 256 /dev/ada0p3   
    # set your password # geli attach /dev/ada0p3   
    # newfs -E -t -U -m 0 -j /dev/ada0p3.eli   
    # mount /dev/ada0p3.eli /mnt   
    # mkdir /mnt/bootdir   
    # mount /dev/ada0p2 /mnt/bootdir   
    # exit   


Most of the difficult parts are completed, now we just wait for bsdinstall to extract and install the base, kernel, ports, etc. After this step, you will be asked for a root password, network configuration, among other things. On the "Final Configuration" window, select "Exit" and wait for the next screen, it may take a while depending on your system specifications. Select "Yes" on "Manual Configuration". The following few steps are very important, we want to make sure the /boot partition is linked to the right place and the configuration files to be right. 

    # mv boot bootdir/   
    # ln -fs bootdir/boot   


Edit the file "/boot/loader.conf" and enter the following, please make sure to find out if your processor supports <abbr title="Advanced Encryption Standard (AES) Instruction Set">AES-NI</abbr>. Newer Intel and AMD processors support this. If yours do not, remove the second line shown below. 

    vfs.root.mountfrom=”ufs:/dev/ada0p3.eli”   
    aesni_load=”YES”   
    geom\_eli\_load=”YES”   


Edit the file "/etc/fstab" and enter the following: 

    # Device    Mountpoint    FStype    Options    Dump    Pass   
    /dev/ada0p3.eli   /  ufs  rw  0  0   
    /dev/ada0p2   /bootdir  ufs  rw  1  1   


After you save the above file and exit, type in the command line: 

    # exit 

Select "Reboot". Make sure to boot from your hard drive instead of the install media. And we are finished, after you log into your system, type the following commands to make sure that trim is enabled. Although FreeBSD is now installed, there is much more to be done, including the installation of Xorg, your favorite <abbr title="Desktop Environment">DE</abbr>, <abbr title="Window Manager">WM</abbr> or office suite. For those questions, the FreeBSD handbook and forum are the place to go.

    # tunefs -p /dev/ada0p2   
    # tunefs -p /dev/ada0p3   


Sources: [1](http://namor.userpage.fu-berlin.de/howto_fbsd9_encrypted_ufs.html) and [2](https://www.dan.me.uk/blog/2012/05/05/full-disk-encryption-in-freebsd-9-x-well-almost/).
