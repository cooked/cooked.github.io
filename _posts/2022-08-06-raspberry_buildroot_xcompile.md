---
layout: single
title: Raspberry Pi cross-compile using Buildroot 
description: RPi x-compile

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Raspberry Pi, Buildroot, OpenCV, Qt5]

header:
    teaser: /assets/img/buildroot_rpi_logo.jpg
---

This article provides a simple but robust workflow to develop embedded applications using cross-compilation.
These notes are tailored to applications based on OpenCV and Qt5, that will run on a Raspberry Pi 4.  
Setting up the packages for computer vision and UI, as well as their dependencies, already make a compelling case for building an automated workflow that is simple and repeatable.

## Introduction

Developing applications for embedded systems means having to deal with hardware constraints that seriously hinder the possibility of local development (local here refers to the development done onto the embedded device itself).
A smarter alternative is instead to setup a development machine, where all the phases of the coding happen, that connects to the embedded device only to deploy and run the application after it has been built.

This workflow is referred to as **cross-compilation** and to work properly it relies on two key aspects:

- the cross-compilation **toolchain**, the set of tools (compiler, linker, etc.) available on the development machine (i.e. host) and needed to build the source code into binary files for the embedded device (target). Since the CPU architecture of the host and the target might not be the same, the standard toolchain of the host, which is designed to produce binaries for its own architecture, doesn't work for cross-compilation. Instead, for example, a development x86 machine used to develop for Raspberry will use a toolchain that runs on x86 but compiles for ARM.

- the **target filesystem**, the set of libraries and other relevant resources representing the target environment of the embedded device in which the cross-compiled application will run. For example,

As described above, and when compared to local development, cross-compiling might seems more convoluted to set up and it might end up being more difficult to maintain.

## Goal

The overarching goal is to have a (mostly) automated and consistent way of generating

## State of the art

Currently there are few other notable ways to achieve the same result, few of which I've used in the past.

TODO: list methods:

- qengineering
- abishek

## TL;DR

Refer to the buildroot configuration in

Run the following

```bash
# build the raspberry img
make

```

## Buildroot

### Install

``` bash
cd ~
git clone https://github.com/buildroot/buildroot.git
cd buildroot
```

### Configure

Qt creator needs:

- sftp (use openSSH instead of dropbox)
- rsync

### Setup SDK and dev env

``` bash
cd ~/buildroot
make sdk
```

- copy the sdk into home
- config the sdk (run relocate.sh)

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

## Host computer

### Configure QtCreator

### Qt project deploy
