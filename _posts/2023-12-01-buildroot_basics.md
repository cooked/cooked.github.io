---
layout: single
title: Buildroot basics

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Buildroot, Linux, Kconfig]

header:
    teaser: /assets/img/buildroot-rpi/buildroot-cover.png
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

See [8.5. Building out-of-tree](https://buildroot.org/downloads/manual/manual.html#_building_out_of_tree)

```bash
cd buildroot
make O=../<buildroot-output> menuconfig # exit without saving
cd ../<buildroot-output>
```

**Note!** A relative path here, is interpreted relative to the main Buildroot source directory.
{: .notice--info}

When using out-of-tree builds, the Buildroot .config and temporary files are also stored in the output directory.  
Buildroot generates a Makefile wrapper in the output directory

## External folder

This first part is optional if you already have the ext folder.

```bash
mkdir <buildroot-out-of-tree-folder>
cd <buildroot-out-of-tree-folder>
touch external.desc
touch Config.in 
touch external.mk
```
Let Buildroot know about the ext folder setting the BR2_EXTERNAL variable:

```bash
make BR2_EXTERNAL=/path/to/ext/folder menuconfig # exit without saving
```

**Note!** If building **out-of-tree** run the above commands from the Buildroot output folder.
{: .notice--success}

**Note!** A relative path is interpreted relative to the main Buildroot source directory, not to the Buildroot output directory.
{: .notice--info}

See [9.2. Keeping customizations outside of Buildroot](https://buildroot.org/downloads/manual/manual.html#outside-br-custom).


## Work with config

``` bash
# list configurations
make list-defconfigs

# load one of the available configurations (then edit it)
make rpi4_defconfig
make menuconfig

# save at the location specified in .config
make savedefconfig
# or save overriding the location
make BR2_DEFCONFIG=<whatever/path/to/file_defconfig> savedefconfig
```

## Custom users

See Buildroot manual, [9.6. Adding custom user accounts](https://buildroot.org/downloads/manual/manual.html#customize-users).
https://buildroot.org/downloads/manual/manual.html#customize-users

**Watch out!** Each line MUST be terminated and MUST be terminated right, which is each line having a trailing **\n**, and the last line being an empty one.
{: .notice--info}

## Control the (re)build

```bash
# build current configuration
make
```

```bash
# rebuild a package 
rm -rf output/build/<package_name>
make
```

```bash
# clean only the target  
rm -rf output/target
find output/ -name ".stamp_target_installed" -delete
rm -f output/build/host-gcc-final-*/.stamp_host_installed
```



## Build and Export the toolchain

See [6.1. Cross-compilation toolchain](https://buildroot.org/downloads/manual/manual.html#_cross_compilation_toolchain) (specifically 6.1.3).

If you need to build application for the system that is built by Buildroot you need a cross-compilation toolchain (and possibly the target rootfs).  
The compilation toolchain that comes with your system runs on and generates code for the processor in your host system. As your embedded system has a different processor, you need a cross-compilation toolchain - a compilation toolchain that runs on your host system but generates code for your target system (and target processor).

``` bash
cd ~/buildroot
make sdk
```

```bash
cd ~
cp buildroot/output/images/arm-buildroot-linux-gnueabihf_sdk-buildroot.tar.gz .
tar -xvf arm-buildroot-linux-gnueabihf_sdk-buildroot.tar.gz
mv arm-buildroot-linux-gnueabihf_sdk-buildroot rpi-buildroot-sdk
```

Then fix the simlinks running the relocator script:

```bash
cd rpi-buildroot-sdk
./relocate-sdk.sh
```
