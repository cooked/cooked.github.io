---
layout: single
title: Odoo - Free ERP on the Raspberry Pi
tags: [ERP, CRM, Raspberry Pi]
category: embedded

#header:
    #image: /assets/img/fallback-layout.jpg
#    teaser: https://www.allaboutcircuits.com/uploads/articles/TMC5160-stepStick.jpg

---

Odoo is a (partially) free ERP tool that lets you manage your small project like a pro.

## Overview

An Enterprise Resource Planning (ERP) is a piece software that organizations use to manage day-to-day business activities such as accounting, procurement, project management, risk management and compliance, and supply chain operations.
Historically, these software have been expensive and not really viable other than for large companies. When OpenERP (Odoo predecessor) became available things changed quite a bit, since now a free tool with good functionality was available for the occasional user.
The project, that has grown ever since and changed name to Odoo, now offers a mix of paid modules to appeal a larger and more premium user base, along with the foundational free modules (e.g Manufacturing and Inventory).

## Installation

Installing Odoo on the Raspberry Pi 4 is a matter of running a couple of commands, once you've got the SD card with the OS configured.
I normally prepare a basic system with the following steps:

- flash Raspberry OS 64bit Lite, using Imager (TODO: note to create default user !!!) (see [link]())
- add the /boot/ssh empty file, to enable SSH in headless mode (see [link]())
- add the wpa_supplicant.conf to allow the Pi to connect to your wifi (see [link]())

Then we SSH to the Raspberry and we run the following commands to update the system, add dependencies and install Odoo:

``` bash
sudo apt update
sudo apt upgrade
sudo apt install postgresql -y
sudo apt install odoo-14 -y

# check Odoo service is running
sudo systemctl status odoo
```

Odoo is now available at the address **http://"raspberry_ip":8069**

## Manufacturing

TODO:

## Inventory

TODO:

## Resources

[https://linuxhint.com/install-odoo-raspberry-pi-os/](https://linuxhint.com/install-odoo-raspberry-pi-os/)
