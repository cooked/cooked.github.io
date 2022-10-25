---
layout: single
title: Zephyr RTOS SPI slave addressing 
tags: [Zephyr RTOS, SPI]

header:
    teaser: https://user-images.githubusercontent.com/50737216/114847513-1117b900-9dde-11eb-989d-ea8da77ef906.png

#excerpt_separator: "<!--more-->"
---

How to select a slave device on the SPI bus using default addressing or custom chip-select pins.  

## Context

The SPI API of Zephyr RTOS does a great job in streamlining the mundane task of interfacing to a sensor connected on the SPI bus.

## Custom chip-select pin

TODO:
Here's another way, not officially documented:  

- do not specify any CS pin in the SPI node
- specify CS as gpios in the device/driver node
- in the driver explicitly operate the CS before and after transceiving on the SPI bus

## Custom slave address

This can use the default cs approach
TODO: this is the case where the slave possibly has one or more pin to hardwire a slave address (based on those pin levels..2 pins up to 4 slaves on the same ) 


## Reference

- [SPI - Zephyr Project Documentation](https://docs.zephyrproject.org/latest/hardware/peripherals/spi.html)
