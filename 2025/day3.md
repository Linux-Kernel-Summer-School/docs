# Day 3 – GPIO, Interrupts, and Custom Input Drivers

Slides: [day3_slides](./day3_slides)

## Description

This lab session focuses on working with General Purpose Input/Output (GPIO) interfaces in Linux, emphasizing interrupt handling and the creation of custom input drivers. The objective is to implement and test kernel modules that respond to GPIO-triggered hardware events, interact with user-space applications, and demonstrate full-stack integration using a terminal-based Python game. By the end of this lab, one should be able to configure GPIOs, handle interrupts in the kernel, and connect kernel drivers with real-time user-space applications.

---

## Topics Covered
- Linux GPIO subsystem
- Kernel interrupt handling (IRQs)
- Using `devm_request_irq()` and `gpiod_to_irq()`
- Debouncing and edge-triggered interrupts
- Communicating from kernel space to user space
- Integrating kernel drivers with user-space applications (e.g., Python)

---

## Linux and Board Setup

Please use the following [cheatsheet](./cheatsheet.md)!

1. Before starting the lab exercises, perform this action **once** at the beginning.
   - [Update linux repo](./cheatsheet.md#Updating the Linux kernel repository)
   - [Update the lkss-utils repo](./cheatsheet.md#Updating the utilities repository)

2. To test a module after each update, follow these steps:
  - [Enable and compile a kernel module](./cheatsheet.md#Enabling a kernel module)
  - [Install the kernel modules to rootfs](./cheatsheet.md#Installing the kernel modules in the rootfs)
  - [Boot the board](./cheatsheet.md#Booting the board)

---
## Lab Exercises

Before starting this lab, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB3**.

**HINT**: Use `ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make menuconfig `

Before diving into the software-related tasks, we need to set up our hardware environment.

The schematics for this setup can be found here:
[LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)

Please check the bottom-right corner of each page for the name of the circuit. For today's lab, we'll be using LKSS LAB3 Circuits (see page 3).

For testing purposes, begin with the Button-LED circuit to verify that pressing the button successfully turns on the LED.

### Exercise 1: Handling Button Presses with GPIO and Interrupts

**Hardware environment**: Set up `GPIO Button` circut from LKSS LAB3 Circuits (see page 3) from [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)

**Objective**: Implement a Linux kernel module that detects button presses using GPIO interrupts and logs the events to the kernel message buffer.

**Steps**:
1. Enable the `button` node in the device tree file `imx93-11x11-frdm.dts` by changing its status from `disabled` to `okay`.
This file can be found in `~/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts`
2. In function `button_irq_handler()` count how many times the button was pressed and print the info each time.
3. Build and Load
- Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.
- Check if the driver has been successfully loaded by running: `dmesg | grep button`.
4. Test

---

### Exercise 2: Controlling an LED with a Button

**Hardware environment**: Set up `GPIO Button and LED` circut from LKSS LAB3 Circuits (see page 3) from [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)

**Objective**: Extend the button driver to toggle an LED each time the button is pressed.

**Steps**:
1. Enable the `button-led` node in the device tree file `imx93-11x11-frdm.dts` by changing its status from `disabled` to `okay`.
This file can be found in `~/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts`
2. In function `button_led_irq_handler()` toggle LED each time button is pressed.
3. Build and Load
- Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.
- Check if the driver has been successfully loaded by running: `dmesg | grep button`.
4. Test

---

### Exercise 3: Snake Game Controlled by GPIO Button Driver

**Hardware environment**: Set up `SNAKE` circut from LKSS LAB3 Circuits (see page 3) from [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)

**Objective**: Execute a Python-based snake game that uses a custom GPIO button driver for input.

**Steps**:
1. Enable the `snake` node in the device tree file `imx93-11x11-frdm.dts` by changing its status from `disabled` to `okay`.
This file can be found in `~/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts`
2. TODO
3. Build and Load
- Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.
- Check if the driver has been successfully loaded by running: `dmesg | grep snake`.
4. Test
- Run the snake game: `python3 snake.py`
- Use physical GPIO-connected buttons to control movement within the game.

## Resources

- [Kernel Docs - GPIO Descriptor Consumer Interface](https://docs.kernel.org/driver-api/gpio/consumer.html#gpio-descriptor-consumer-interface)
- [Linux Kernel - gpiolib.c](https://elixir.bootlin.com/linux/latest/source/drivers/gpio/gpiolib.c#L4725)
