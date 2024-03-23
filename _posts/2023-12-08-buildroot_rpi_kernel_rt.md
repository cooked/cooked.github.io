---
layout: single
title: Raspberry Pi Linux Real-Time Kernel with Buildroot
description: RPi x-compile

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Raspberry Pi, Buildroot, Linux, Kernel, Real-Time, RT, PREEMPT_RT]

header:
    teaser: /assets/img/buildroot-rpi/buildroot-cover.png
---

How to use Buildroot to create a minimal Linux system with fully preemptible kernel.

## TL;DR

Jump straight to the Buildroot [config file](https://github.com/cooked/buildroot-ext/tree/master/configs).

## Introduction
As a developer, I’ve often encountered situations where I needed a real-time-capable system. Whether it’s for work or personal projects, having a responsive and predictable system is crucial. Recently, I delved into the world of real-time Linux kernels, specifically using the PREEMPT-RT patch. In this blog post, I’ll share my journey and provide step-by-step instructions for patching the Linux kernel with PREEMPT-RT using Buildroot.

## Real-time kernel

Real-time (RT) systems are designed to meet strict timing requirements, ensuring that tasks are executed within specific time constraints. These systems are critical for applications such as industrial automation, robotics, automotive control, and audio/video streaming.

The PREEMPT-RT patch enhances the standard Linux kernel, making it suitable for real-time applications. By enabling full preemption, the kernel can respond promptly to high-priority tasks, reducing latency and ensuring predictable behavior. This is achieved by enabling threaded interrupt handlers, making locking primitives preemptible, and minimizing non-preemptible code sections.

## Implementation

Succesfully applying the Linux Kernel patch with Buildroot is a matter of properly picking and configuring the kernel source itself and a compatible PREEMPT_RT patch source.

For the specific case of the Raspberry these are the most important config changes:

1. replace the tarball specified in the [defconfig shipped with Buildroot](https://github.com/buildroot/buildroot/blob/master/configs/raspberrypi4_defconfig) for the Raspberry Pi kernel source from their GitHub repo
2. specify a compatible kernel patch, to be sourced from the [The Linux Foundation - PREEMPT_RT patch versions](https://wiki.linuxfoundation.org/realtime/preempt_rt_versions)
3. align the kernel headers version the toolchain will use at compile time

### Raspberry Pi kernel source 

Using the Raspberry Pi kernel has the clear advange that it comes with any customization for the hardware we're targeting. In particular, we can still make use of the intree DTS that would not be available patching a vanilla kernel.

```bash
BR2_LINUX_KERNEL_INTREE_DTS_NAME="bcm2711-rpi-4-b"
```

However, when browsing the [Raspberry Pi kernel repository](https://github.com/raspberrypi/linux) it can be seen that the descriptions of the Pi are not defined for all the branches and tags the kernel sources. When available (e.g., in the **stable_** tags but not in the **rpi-x.x.x** tags), the corresponding device tree files are found in

```bash
arch/arm/boot/dts
```

In the end, I picked the [**stable_20211118**](https://github.com/raspberrypi/linux/tree/stable_20211118) tag of the Raspberry Pi Linux kernel.

What is still left to know at this point is what kernel version the selected tag will build. This information can be found in the main [Makefile](https://github.com/raspberrypi/linux/blob/stable_20211118/Makefile) which, for the given tag looks like

```bash
VERSION = 5
PATCHLEVEL = 10
SUBLEVEL = 63
```

In hindsight, the reason to roll back to an older stable tag is that I couldn't find a proper PREEMPT_RT patch for the later kernel that Buildroot was defaulting to. 
{: .notice--info}

### PREEMPT_RT patch source 

Once the kernel version is known, a proper PREEMPT_RT patch can be identified from the official list at [The Linux Foundation - PREEMPT_RT patch versions](https://wiki.linuxfoundation.org/realtime/preempt_rt_versions).

**Matching version** - The ideal patch would have version, patchlevel and sublevel all matching. In my case **I was able to match only version and patchlevel**, then I picked the latest patch sublevel older than the kernel sublevel, and with a stable RT version (i.e., not rc, or release candidate), which points to [**5.10.59-rt52**](https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.10/older/patch-5.10.59-rt52.patch.xz).
{: .notice--info}

```bash
# selected PREEMPT_RT patch
5.10.59-rt52
```

### Buildroot defconfig file

The following snippet shows the most important changes to the standard Raspberry Pi 4 defconfig file, to build the RT kernel. The complete defconfig file is available here.

```bash
# source kernel from Raspberry Pi GitHub (at an explicti tag)
BR2_LINUX_KERNEL_CUSTOM_GIT=y
BR2_LINUX_KERNEL_CUSTOM_REPO_URL="https://github.com/raspberrypi/linux.git"
BR2_LINUX_KERNEL_CUSTOM_REPO_VERSION="stable_20211118"

# point to a compatible patch version
BR2_LINUX_KERNEL_PATCH="https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.10/older/patch-5.10.59-rt52.patch.xz"

# keep the RT flag in a fragment file, for modularity
BR2_LINUX_KERNEL_CONFIG_FRAGMENT_FILES="$(BR2_EXTERNAL)/board/raspberrypi-rt/linux.config"

# align the toolchain header files
BR2_PACKAGE_HOST_LINUX_HEADERS_CUSTOM_5_10=y
```

Keeping the RT flag in a separate file has the advantage that we don't have to issue the **make linux-config** command manually. Buildroot automatically applies the RT config flag, only after the kernel has been patched (i.e., only when the RT config flag is actually available, since before the patch it is not). The kernel fragment file is a one liner:

```bash
# enable full preemption
CONFIG_PREEMPT_RT=y
```

**Buildroot** - If you don't have Buildroot installed you can follow the setup steps (and a shortlist of useful commands) described in [Buildroot basics]({{ site.baseurl }}/buildroot_basics)
{: .notice--info}

[![](../assets/img/buildroot-rpi/buildroot-rpi-kernel.png)](../assets/img/buildroot-rpi/buildroot-rpi-kernel.png)

[![](../assets/img/buildroot-rpi/buildroot-rpi-kernel-toolchain.png)](../assets/img/buildroot-rpi/buildroot-rpi-kernel-toolchain.png)

## References

- [SO - Where do I find the version of a Linux kernel source tree?](https://stackoverflow.com/a/12151781) 
- [SO - How to grep commits based on a certain string?](https://stackoverflow.com/questions/1337320/how-to-grep-commits-based-on-a-certain-string)
- [SO - How to find a commit by its hash?](https://stackoverflow.com/questions/14167335/how-to-find-a-commit-by-its-hash)
