---
layout: single
title: Trinamic TMCs drivers in Zephyr RTOS
description: Add support for Trinamic TMCs stepper drivers to Zephyr OS

tags: [Zephyr RTOS, Trinamic, TMC5160, TMC5130]

header:
    #image: /assets/img/fallback-layout.jpg
    teaser: https://www.allaboutcircuits.com/uploads/articles/TMC5160-stepStick.jpg

excerpt_separator: "<!--more-->"

last_modified_at: 2023-05-27
---

Introduction to the Zephyr RTOS driver for the TMC51xx series of motor ICs from [TRINAMIC](https://www.trinamic.com/).

<!--more-->

## Dependencies

A fork of the [TMC-API](https://github.com/cooked/TMC-API) is used, for reusability of the header files and definitions published and maintained by [Trinamic (now ADI)](https://www.analog.com/en/lp/001/trinamic-support.html).
The fork is needed since Zephyr-related PRs have been repeatedly rejected by the company (see [#31](https://github.com/analogdevicesinc/TMC-API/pull/31), [#52](https://github.com/analogdevicesinc/TMC-API/pull/52)).

When the [zephyr-trinamic](https://github.com/cooked/zephyr-trinamic) module is used in an application, the TMC-API dependency is pulled-in automatically via the [project manifest](https://github.com/cooked/zephyr-trinamic/blob/master/west.yml).

The fork simply add few CMake and config files to enable to build it in Zephyr:

- [CMakeLists.txt](https://github.com/cooked/TMC-API/blob/zephyr/CMakeLists.txt)
- [Kconfig.trinamic](https://github.com/cooked/TMC-API/blob/zephyr/Kconfig.trinamic)
- [zephyr/module.yml](https://github.com/cooked/TMC-API/blob/zephyr/zephyr/module.yml)

## State of the art

The driver supports both **SPI and UART** communication protocols, on top of the **STEP/DIR** signal to control speed and direction rotation. It provides a set of console commands to configure the driver and to run the stepper motor in **velocity or position mode, using the internal ramp generator**. Source code can be downloaded from [GitHub](https://github.com/cooked/zephyr-trinamic).

### TODOs

- implement the full Sensor API to set/get driver parameters

## STEP/DIR interface

The driver relies on the MCU's hardware timers and Zephyr's [PWM API](https://docs.zephyrproject.org/latest/hardware/peripherals/pwm.html) to generate the step pulses to rotate a stepper motor.
In the devicetree (or overlay) file, at least one PWM node (child of a timer node) shall be enabled.  
With reference to the [STM32 PWM binding](https://docs.zephyrproject.org/latest/build/dts/api/bindings/pwm/st%2Cstm32-pwm.html#dtbinding-st-stm32-pwm), the PWM period is defined in the *pwms* cell of the TMC (see snippet below).

``` bash
# STEP/DIR configuration for the ST Nucleo F103

&timers1 {
    status = "okay";
    st,prescaler = <100>;

    stepper0: pwm {
        status = "okay";
        # PA8 is the step pin, which is attached to TIM1 channel 1 on the F103 
        pinctrl-0 = < &tim1_ch1_pwm_pa8 >;
        pinctrl-names = "default";
    };
};

&spi2 {
    #...
    tmc_0: tmc5160@0 {
        #...
        # STEP - pwms cell linked to the pwm defined above, using channel 1 with period 10000ns and in normal polarity
        pwms = <&stepper0 1 10000 PWM_POLARITY_NORMAL>;
        # DIR - a simple gpio
        dir-gpios = <&gpioc 7 GPIO_ACTIVE_HIGH>;    
        #...
    };
};
```

The driver change the motor speed setting the PWM period and direction:  

``` c

struct tmc_config *cfg = dev->config;

// period (in ns) of the input speed (given in RPM)
uint32_t period_ns = 1e9 / ( abs(speed) * RPM_TO_PPS);

// set period w/ 50% duty cycle, CW
if(speed>0) {
    gpio_pin_set_dt(&cfg->dir,0);
    pwm_set_dt(&cfg->step, period_ns, (uint32_t)(period_ns/2));
// set period w/ 50% duty cycle, CCW
} else if(speed<0) {
    gpio_pin_set_dt(&cfg->dir,1);
    pwm_set_dt(&cfg->step, period_ns, (uint32_t)(period_ns/2));
// stop motion
} else {
    pwm_set_pulse_dt(&cfg->step, 0);
}

```

## SPI interface

Interfacing with the TMC via SPI is pretty straightforward.  

**SPI speed** is set to 1MHz to be on the safe side, but can be increased up to 4MHz, according to the datasheet.
{: .notice--info}  

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

## UART interface

Interface via UART is a bit more tricky but serves better the case of several TMC drivers connected on the same bus. Here 
the UART on the "master" (a Nucleo F103RB board) is configured as "single-wire" and uses DMA.  

``` bash
&usart1 {
    dmas = <&dma1 4 0x440>,
            <&dma1 5 0x480>;
    dma-names = "tx", "rx";
    single-wire;
    pinctrl-0 = <&usart1_tx_pa9>;

    tmc0: tmc5160 {
        compatible = "trinamic,tmc5160";
        status = "okay";
    };
};
```

On the ST Nucleo F103RB board the UART2 is used for the consol/debug. Select a different UARTx for the TMC driver.  
{: .notice--warning}  

## Shell commands

The driver provides a simple set of commands (below is a subset) that can be used to run a motor or just peek into drivers registers.

``` bash

# Run motor (address) 0 at 10 rpm
> tmc 0 run 10

# Stop the motor
> tmc 0 run 0

# Rotate motor fwd by 2 turns and back by 1 (incremental positioning)
> tmc 0 turn 2
> tmc 0 turn -1

# Rotate motor to 0 turns (absolute positioning)
> tmc 0 move 0

# Read  current position from the register
> tmc 0 get xactual
```

## Sample applications

Sample applications for connecting a ST Nucleo F103RB board to a TMC5160 via [SPI and UART are provided](https://github.com/cooked/zephyr-trinamic/tree/master/samples) as a starting point.  

## Development

### Workspace  

Load the VSCode [workspace provided](https://github.com/cooked/zephyr-trinamic) for a quick start with development.  

### Project setup  

``` bash
cd <repo root>
west init -l manifest
west update
```

The provided **west manifest** is kept small on purpose. It works out-of-the-box for STM32 silicons and shall be modified if you're targeting a different one. See [Zephyr manifest](https://github.com/zephyrproject-rtos/zephyr/blob/main/west.yml) for the complete list of modules
{: .notice--info}

### Build

The project comes with a short list of [**VSCode build tasks**](https://github.com/cooked/zephyr-trinamic/blob/master/.vscode/tasks.json) for the sample applications, that can be accesses via **<<CTRL+SHIFT+B>>**.  

## Hardware  

The TMC5160 driver has been tested with the following hardware:
- [Nucleo-64 STM32F103RB](https://www.st.com/en/evaluation-tools/nucleo-f103rb.html) board
- Watterott [SilentStepStick TMC5160](https://shop.watterott.com/SilentStepStick-TMC5160-Stepper-motor-driver) (SPI only, UART pins not available)
- TRINAMIC [TMC5160-BOB](https://www.trinamic.com/support/eval-kits/details/tmc5160-bob/) breakout board
- TRINAMIC [TMC5160-EVAL](https://www.trinamic.com/support/eval-kits/details/tmc5160-eval/) 
- own custom boards based on TMC5130 chip

**POWER-UP SEQUENCE** 1) apply motor voltage VM, 2) apply digital IO voltage VCC_IO. The driver IC is **reset cycling VCC_IO**
{: .notice--info}

### NUCLEO/TMC5160-StepStick (SPI)

**HW MOD** Watterott StepStick have the STEP/DIR interface enabled by default. To enable the TMC5160 internal ramp generator apply this mod **TODO**
{: .notice--warning}

**UART** Watterott StepStick does not support UART interface 
{: .notice--danger}

| **NUCLEO-F103RB** | **TMC5160 board** | Description |
|---|---|---|
|  | GND | Ground |
|  | VM | Motor Supply Voltage (10-35V) |
| CN6 pin 4 (3V3) | VIO | Logic Supply Voltage (3.3-5V) |
| GND | EN | Enable Motor Outputs (GND=on, VIO=off) |
| CN10, PB15 (SPI2_MISO) | SDO | MISO - Serial Data Output |
| CN10, PB14 (SPI2_MOSI) | SDI | MOSI - Serial Data Input |
| CN10, PB13 (SPI2_CLK) | SCK | SCLK - Serial Clock Input |
| CN10, PB12 (SPI2_CLK) | CS | SS - Chip Select Input (no internal pu resistor) |
| CN9, PA8 (TIM1_CH1) | STEP | STEP - Step Input) |
| CN5, PC7 | DIR | DIR - Direction Input) |
|  | M1A/B, M2A/B | Motor Coils |

![nucleo-ss-wiring-spi](/assets/img/nucleo-tmc5160.png){: .align-center}

### NUCLEO/TMC5160-BOB (SPI)

**UART** Trinamic BOB board does not support UART interface **(TODO see schematic)**
{: .notice--danger}

The table of connections is the same as for the StepStick above, while the wiring diagram is provided below.

![nucleo-bob-wiring-spi](/assets/img/nucleo-tmc5160bob-spi.png){: .align-center}

### NUCLEO/TMC5160-EVAL (UART)

![nucleo-bob-wiring-spi](/assets/img/nucleo-tmc5160eval-uart.png){: .align-center}
