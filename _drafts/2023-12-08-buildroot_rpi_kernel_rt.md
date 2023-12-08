---
layout: single
title: Raspberry Pi Linux Real Time Kernel with Buildroot 
description: RPi x-compile

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Raspberry Pi, Buildroot, Linux, Kernel, Real-Time, RT]

header:
    teaser: /assets/img/buildroot_rpi_logo.jpg
---

How to use Buildroot to build a minimal Linux system with fully preemptible real-time kernel.

## Introduction

Many times in the past I found myself in need of a real-time capable system. Being that for work or spare time projects it didn't matter, the goal was always to find a quick (automatable) and reproducible way to (patch and) build the Linux kernel. 
Fast-forward 2023 I think I got the recipe right, and since I will jump to another project too soon, these notes are here for future reference.

## TL;DR

Refer to the buildroot configuration in

Run the following

```bash
# build the raspberry img
make
```

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

then fix the simlinks running the relocator script

```bash
cd rpi-buildroot-sdk
./relocate-sdk.sh
```
