---
layout: default
title: A minimal development environment for Zephyr OS
#description: Prot
---

When I started developing for the Zephyr OS the first time I followed the project's [Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html). It is fairly complete, easy to follow and will get you started in a breeze.  
But now I needed something different!  

Being a bit more familiar with Zephyr's framework I've come to realize that there is another way to put together a development environment that better suits my taste. The dev env created is minimal in the sense that only the necessary tools and source code are installed.

## The big picture  

all the following is on linux

## zephyr full

cd zephyrproject
west init
west update

this will use the manifest from the remote default zephyr project repo and will download everything (all modules for all silicons, all hals, etc..)
size of the sources (bootloader+modules+tools+zephyr): 3.7GB

zephyr SDK full
0.14.2 = 5.1GB
0.15.0 = 5.8GB


## zephyr minimal

local manifest,only zephyr+st hal and cmsis
size: 

https://docs.zephyrproject.org/latest/develop/west/manifest.html