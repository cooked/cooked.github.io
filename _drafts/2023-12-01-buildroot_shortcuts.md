---
layout: single
title: Buildroot shortcuts cheatsheet

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Buildroot, Linux, Kconfig]

#header:
#    teaser: /assets/img/buildroot_rpi_logo.jpg
---

Some of the most useful takeaways from the [Buildroot official manual](https://buildroot.org/downloads/manual/manual.html).

## Get Buildroot

Just the kick-off to start the whole journey with the Buildroot latest stable release:

```bash
git clone https://github.com/buildroot/buildroot.git
# checkout latest stable release 
cd buildroot
git fetch --all --tags
git checkout <yyyy.mm>
```

The steps above will put you in a detached head, which is fine if you're "just using" buildroot for your builds, without contributing back.

## Build out-of-tree

```bash
cd buildroot
make O=../<buildroot-output> menuconfig # exit without saving
cd ../<buildroot-output>
```

## External folder

This first part is optional if you already have the ext folder.

```bash
mkdir <buildroot-out-of-tree-folder>
cd <buildroot-out-of-tree-folder>
touch external.desc
touch Config.in 
touch external.mk
```

## Work with config

``` bash
# list configurations
make list-defconfigs

# save configurations (specifying the path make sure to override whatever path is defined in the current .config)
make BR2_DEFCONFIG=<whatever/path/to/file_defconfig> savedefconfig
```

## Custom users

See manual 
https://buildroot.org/downloads/manual/manual.html#customize-users

add the user table as indicated… The user folder like for example /home/pi is created automatically by Buildroot.

IMPORTANT - Eeach line MUST be terminated and the termination MUST be right. This means, if the tables contains only 

TODO: also pay attention to create the user table with nano or another basic editor (do not use VScode)

http://lists.busybox.net/pipermail/buildroot/2017-October/204137.html

## Control the (re)build

```bash
# build current configuration
make
```

TODO: read manual and summarize

## Export the toolchain


``` bash
cd ~/buildroot
make sdk
```
