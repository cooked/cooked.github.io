---
layout: single
title: Zephyr RTOS minimal development environment 
tags: [Zephyr OS]

header:
    teaser: https://docs.zephyrproject.org/3.1.0/_images/west-mr-model.png

#excerpt_separator: "<!--more-->"
---


When I started developing for the Zephyr OS the first time I followed the project's [Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html). It is easy to follow and will get you started quickly.  
But now I needed something different!  

Here's another way, not officially documented, to create the minimal development environment I was looking for (only the necessary tools and source code are installed).

## Goal

Setup a custom Zephyr development environment with minimal footprint (<1GB incl SDK and workspace)

## Minimal Zephyr SDK  

A full Zephyr SDK, installed by the steps of the [Getting Started Guide, contains the toolchains for all the supported architectures.
most of which I (and probably you) will never use.
In total they end up using some **5 to 6GB** (with the latest versions of the SDK being also the more space hungry).

On the other end, a minimal SDK starts with an almost empty folder. Any needed toolchain is added manually. This way the disk used can be as low as **100-200MB**.  

Here how to do it (on Linux).

### SDK bundle  

Download the desired version of the "minimal bundle" from the [Zephyr SDK release page](https://github.com/zephyrproject-rtos/sdk-ng/releases):  

``` bash
# replace 0.15.1 with the desired version
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.15.1/zephyr-sdk-0.15.1_linux-x86_64_minimal.tar.gz
tar xvf zephyr-sdk-0.15.1_linux-x86_64_minimal.tar.gz
```

### Toolchain(s)  

Most of my projects run non ARM microcontrollers so I will install just the toolchain for **arm-zephyr-eabi** :

``` bash
cd zephyr-sdk-0.15.1
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.15.1/toolchain_linux-x86_64_arm-zephyr-eabi.tar.gz
tar xvf toolchain_linux-x86_64_arm-zephyr-eabi.tar.gz
rm toolchain_linux-x86_64_arm-zephyr-eabi.tar.gz
```


## Minimal Zephyr source (via west Manifest)

Another area of optimization is the codebase any Zephyr workspace is initialized to. By default (see Getting Started Guide) the source code for zephyr, bootloader, tools and all modules are installed, using the *manifest file* from the remote default Zephyr repository 
The entire codebase totals at around **3.7GB**.

A valid alternative to the default manifest, is a custom one, eventually stored locally in the workspace 

NOTE: the manifest shall be **stored in a workspace subfolder** (e.g. zephyrproject/manifest)

The manifest will include for download only the source code for the modules that we really need (and of course the core Zephyr source)

Since I develop for STM32 a minimal manifest in my case looks like this:

``` yaml
manifest:

  remotes:
    - name: zephyr
      url-base: https://github.com/zephyrproject-rtos

  projects:
    - name: zephyr
      remote: zephyr
      repo-path: zephyr
      revision: main
      import:
        name-allowlist:
          - cmsis
          - hal_stm32
```

## Reference

- [Getting Started Guide - Zephyr Project Documentation](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)
- [West Manifests- Zephyr Project Documentation](https://docs.zephyrproject.org/latest/develop/west/manifest.html)