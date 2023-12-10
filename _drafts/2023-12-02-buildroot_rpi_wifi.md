---
layout: single
title: Wireless network for Raspberry Pi 4 with Buildroot

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Raspberry Pi, Buildroot, Linux, wlan, iwd, dropbear]

header:
    teaser: /assets/img/buildroot_rpi_logo.jpg
---

Enable wireless network (and SSH access) on a Buildroot minimal Linux system for the Raspberry Pi 4.

## Introduction

On each board Buildroot ships configuration for, only *eth0* is configured by default. The Raspberry Pi boards, on the other hand, are generally capable of wireless connectivity.  
Trying to enable *wlan0* proved not to be so straight forward, as many of the articles I've found are now a bit outdated. But eventually it boils down to few little settings.

## TL;DR

Jump to the configuration file for this build...

## RPI firmware

```bash
BR2_PACKAGE_BRCMFMAC_SDIO_FIRMWARE_RPI=y
BR2_PACKAGE_BRCMFMAC_SDIO_FIRMWARE_RPI_WIFI=y
BR2_PACKAGE_RPI_FIRMWARE=y
BR2_PACKAGE_RPI_FIRMWARE_VARIANT_PI4=y
```
The drivers alone however, will not get us quite there. If you 

## Enable MDEV

As described in [this post](https://rohitsw.wordpress.com/2016/12/17/building-a-linux-filesystem-on-raspberry-pi-3/) the WiFi components on the Pi come up a little late, so we need to modprobe the WiFi driver **brcmfmac**, manually, or use an hotplug mechanism that would automate that: enter [MDEV](https://git.busybox.net/busybox/tree/docs/mdev.txt).

- **System Configurations -> /dev management -> Dynamic using Devtmpfs + mdev** 

will set the following value:

```bash
BR2_ROOTFS_DEVICE_CREATION_DYNAMIC_MDEV=y
```



## Network packages

```bash
# network interfaces manager
BR2_PACKAGE_IWD=y

# SSH support
BR2_PACKAGE_DROPBEAR=y
```

Here iwd is preferred to the wpa_supplicant. 
see reason here:


## References

- [Rohit's Rants - Building a Linux Filesystem on Raspberry Pi 3](https://rohitsw.wordpress.com/2016/12/17/building-a-linux-filesystem-on-raspberry-pi-3/) : notes about using MDEV and adding RPI firmware. A lot of the other settings apart from that are not useful as of today (November 2023). Also, wpa_supplicant which requires few more steps than **iwd**, seems outdated. 
- [Raspberry Pi Forum - Enabling wlan on Raspberry Pi 3 custom linux](https://forums.raspberrypi.com/viewtopic.php?t=159034) : forum thread with the link to the previous blog
- [SO - How to connect to Wifi on start-up using Buildroot?](https://stackoverflow.com/questions/71426700/how-to-connect-to-wifi-on-start-up-using-buildroot) : hint on using **iwd** in place of wpa_supplicant