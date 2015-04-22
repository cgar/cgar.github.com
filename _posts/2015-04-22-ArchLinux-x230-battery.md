---
layout: post
title: Battery life, Thinkpad x230, and Arch Linux
---

{{page.title}}

<p class="meta">22 Apr 2015</p>

Introduction

Ever since buying a Thinkpad x230 some years ago, I have neglected tweaking and
monitoring its power usage. Mostly thanks to having it plugged in.The laptop
itself has gone through a couple of operating systems including; OpenBSD,
FreeBSD, Ubuntu, and Arch Linux for now. To begin the journey of power saving
via Arch Linux, here is some info regarding the current battery.

    $ acpi -V
    Battery 0: Discharging, 97%, **03:33:17** remaining
    Battery 0: design capacity 4791 mAh, last full capacity 3081 mAh = 64%
    Adapter 0: off-line
    Thermal 0: ok, 40.0 degrees C
    Thermal 0: trip point 0 switches to mode critical at temperature 103.0 degrees C
    Cooling 0: x86_pkg_temp no state information available
    Cooling 1: intel_powerclamp no state information available
    Cooling 2: Processor 0 of 10
    Cooling 3: Processor 0 of 10
    Cooling 4: Processor 0 of 10
    Cooling 5: Processor 0 of 10

Turning On Audio Power Saving

    $ sudo vim /etc/modprobe.d/audio_powersave.conf
    options snd_hda_intel power_save=1

Turning Off Bluetooth via udev rules

    $ sudo vim /etc/udev/rules.d/50-bluetooth.rules
    SUBSYSTEM=="rfkill", ATTR{type}=="bluetooth", ATTR{state}="0"

Disabling The <abbr title="Non-maskable interrupt">NMI</abbr> watchdog
    
    $ sudo vim /etc/sysctl.d/disable_watchdog.conf
    kernel.nmi_watchdog = 0

Set Kernel at Laptop Mode 5
    
    $ sudo vim /etc/sysctl.d/laptop.conf
    vm.laptop_mode = 5


Disable Wake-on-LAN
    
    # Show the interfaces
    $ ip link show 
    $ sudo vim /etc/udev/rules.d/70-wifi-powersave.rules
    ACTION=="add", SUBSYSTEM=="net", KERNEL=="**wlp3s0**", RUN+="/usr/bin/iw dev %k set power_save on"
    $ sudo vim /etc/udev/rules.d/70-disable_wol.rules
    ACTION=="add", SUBSYSTEM=="net", KERNEL=="**enp0s25**", RUN+="/usr/bin/ethtool -s %k wol d"

Install and enable laptop-mode-tools

    $ curl -O https://aur.archlinux.org/packages/la/laptop-mode-tools/laptop-mode-tools.tar.gz
    $ tar -zxf laptop-mode-tools.tar.gz
    $ cd laptop-mode-tools
    $ makepkg -si
    $ sudo systemctl enable laptop-mode.service

The final result:

    $ acpi -V
    Battery 0: Discharging, 98%, **05:14:50** remaining
    Battery 0: design capacity 4748 mAh, last full capacity 3053 mAh = 64%
    Adapter 0: off-line
    Thermal 0: ok, 45.0 degrees C
    Thermal 0: trip point 0 switches to mode critical at temperature 103.0 degrees C
    Cooling 0: x86_pkg_temp no state information available
    Cooling 1: intel_powerclamp no state information available
    Cooling 2: Processor 0 of 10
    Cooling 3: Processor 0 of 10
    Cooling 4: Processor 0 of 10
    Cooling 5: Processor 0 of 10

Source: [1](https://wiki.archlinux.org/index.php/Power_management/Energy_saving).
