---
layout: single
title: Xiaomi Electric Scooter Battery not Charging - FIXED
tags: [Xiaomi, Scooter, LiPo, BMS]
category: hardware

header:
    #image: /assets/img/fallback-layout.jpg
    teaser: https://www.allaboutcircuits.com/uploads/articles/TMC5160-stepStick.jpg

excerpt_separator: "<!--more-->"
---

How to fix an electric scooter that won't charge.  
<!--more-->

## TL;DR

My Xiaomi 1S won't charge due to a faulty battery pack. In particular, one of the spot welds on the LiPo cell connecting tabs broke.

**NOTE 1** - The loose tab was not visible from the outside, it became evident only removing the wrapping of the battery pack and checking each tab one by one.
{: .notice--info}  

**NOTE 2** - The loose tab was not visible from the outside, it became evident only removing the wrapping of the battery pack and checking each tab one by one.
{: .notice--info}  

## What happened?

I've got my Xiaomi Mi 1S electric scooter back in 2021 and been using it as my daily commute ever since. It has seen some 1500km so far, requiring little to no maintenance (a couple of flat tires).
Few days ago, I hooked up the battery charger, without paying attention that the LED on the charger didn't change to red (stayed on green). I came back the next day to a scooter that I believed was fully charged, just to find out that was not the case on my way to the office.
I barely made it to the office and as I got there I plugged the scooter once again. There I realized that the charger wasn't charging the battery at all.

## Troubleshooting

I've been riding forThe driver supports both **SPI and UART** communication protocols. It provides a set of console commands to configure the driver and to run the stepper motor in **velocity or position mode, using the internal ramp generator**. Source code can be downloaded from [GitHub](https://github.com/cooked/zephyr-trinamic).

- implement the full Sensor API to set/get driver parameters
- implement step/dir mode

## FIX

``` bash
&spi2 {
    cs-gpios = <&gpiob 12 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;

    tmc0: tmc5160@0 {
        compatible = "trinamic,tmc5160";
        status = "okay";
        reg = <0x00>;
        spi-max-frequency = <1000000>;
    };
};
```

## Resources
