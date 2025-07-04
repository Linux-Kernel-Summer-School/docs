# Day 3 – Interacting with hardware: buttons and leds

Slides: [day3_slides](./day3_slides)

## Description

This lab introduces students to the fundamentals of Linux kernel GPIO drivers through a
progressive series of hands-on exercises. Students incrementally explore working with GPIOs,
handling interrupts, and interacting with hardware through device tree bindings.

---

## Topics Covered

- Accessing GPIOs via the `gpiod_*` consumer API
- Registering and handling hardware interrupts with `devm_request_irq()`
- Mapping GPIOs using named device tree properties
- Building simple hardware setups using leds, resistors and buttons

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

### Exercise 0: Blink a LED using a button

Build a basic electronic circuit that allows you to **blink an LED** using a **push-button**
connected to **VCC (3.3V)**. Follow [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)
and look on page 3 at the circuit named `Button LED`. Wire the circuit on your breadboard.

### Exercise 1: Writing a Simple GPIO Button Driver

In this exercise, you'll explore a basic GPIO button driver that registers an interrupt handler
when the button is pressed. This example demonstrates how to use GPIO descriptors and platform
drivers in the Linux kernel.

0. **Create the circuit for the GPIO button**

   Follow  [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)
   and look at page 3 at the circuit named `GPIO button`. Wire the circuit on your breadboard.

1. **Understand the Driver Structure**

   Review the code in `arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts` and identify:
   - `button` device tree node
   - `gpios` properties under the `button` node.
   - activate the node by setting `status = "okay"`. Make sure all other nodes are `disabled`.

   Explore the code under `drivers/lkss/labs/lab3/ex1/button.c`. This driver requests a GPIO line,
   converts it to an IRQ, and sets up an interrupt handler triggered on a falling edge.
   Read through `button.c` and identify the following:
    - How the GPIO is retrieved using `devm_gpiod_get()`
    - How the GPIO is mapped to an IRQ using `gpiod_to_irq()`
    - How the interrupt is requested with `devm_request_irq()`
    - Where and how the interrupt handler is defined.
   Pay special attention to the `button_irq_handler()` function. You'll implement functionality in it.

2. **Implement Interrupt Handling Logic**
   In the interrupt handler `button_irq_handler()`, implement logic to:
     - Count how many times the button was pressed
     - Print the current count using `pr_info()`
   Use a `static int` variable inside the function or a global variable to maintain the count.

3. **Build and load the driver**
   Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.

4. **Test the Driver**
   Press the button and observe the kernel log:
   ```bash
   $ dmesg  | tail -n 20
   ```

5. **Unload the Module**
   ```bash
   $ rmmod button
   ```
---

### Exercise 2: Button-Controlled LED Driver with Named GPIOs

In this exercise, you'll develop a GPIO-based button and LED driver. The LED should toggle each
time the button is pressed. The source code is provided in `drivers/lkss/labs/lab3/ex2/button_led.c`.

The driver retrieves two GPIOs: one for the button (input) and one for the LED (output). It sets
up an interrupt on the button GPIO that triggers on a falling edge. When the interrupt occurs,
the LED should toggle its current state.

0. **Create the circuit for the GPIO button and LED**
   
   Follow  [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)
   and look at page 3 at the circuit named `GPIO button and LED`. Wire the circuit on your breadboard.

1. **Understand the Driver Structure**
    Review the code in `arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts` and identify:
    - `button-led` device tree node
    - `button-gpios` and `led-gpios` properties
    - activate the node by setting `status = "okay"`. Make sure all other nodes are `disabled`.

    Review the code in `button_led.c` and identify the following operations:
    - Retrieve the button GPIO using `devm_gpiod_get(&pdev->dev, "button", GPIOD_IN)`
    - Retrieve the LED GPIO using `devm_gpiod_get(&pdev->dev, "led", GPIOD_OUT_LOW)`
    - Register an interrupt handler for the button using `devm_request_irq()`
    - Implement logic inside `button_led_irq_handler()` to toggle the LED

2. **Implement the IRQ Handler**

    Complete the `button_led_irq_handler()` function with the following logic:
    - toggle the LED state using `gpiod_set_value(led_gpio, !current_state)`
    - Optionally print the new state to the kernel log using `pr_info()` or `dev_info()`
    This ensures that each button press causes the LED to change its state.

3. **Build and load the driver**
   Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.

4. **Test the Driver**
   Press the button and observe the kernel log:
   ```bash
   $ dmesg  | tail -n 20
   ```

5. **Unload the Module**
   ```bash
   $ rmmod button_led
   ```
---

### **Exercise 3: Snake Game Controlled via GPIO Buttons**

In this exercise, you'll explore how GPIO inputs can be used to interact with a Linux kernel 
module. You'll wire up four push buttons to control a "snake" rendered in a terminal-based 
Python application. Explore `drivers/lkss/labs/lab3/ex3/snake.c` processes GPIO input and acts as the backend 
driver for this game.

0. **Create the circuit for the GPIO button and LED**

   Follow  [LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)
   and look at page 3 at the circuit named `SNAKE`. Wire the circuit on your breadboard!

1. **Understand the Driver Structure**
    Review the code in `arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts` and identify:
    - `snake` device tree node
    - `up-gpios`, `down-gpios`, `left-gpios`, `right-gpios` properties
    - activate the node by setting `status = "okay"`. Make sure all other nodes are `disabled`.

    Review the code in `drivers/lkss/lab3/ex3/snake.c` and identify the following operations:
    - Retrieve the button GPIOs eg: `gpiod_get(&pdev->dev, "up", GPIOD_IN);`
    - Registering the interrupt handler for each gpio in `snake_request_gpio_irq`.

    Review the userspace application in `drivers/lkss/lab3/ex3/snake.py` and identify the following operations:
    - Game logic `while` loop inside `main` function.

2. **Build and load the driver**
   Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.

3. **Test the Driver**
   Press the button and observe the kernel log:
   ```bash
   $ dmesg  | tail -n 20
   ```

4. **Run snake.y app on the board**
   ```bash
   $ cd lab-tests/lab3
   $ python snake.py
   ```
---

## Resources

- [Interrupts](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html)
- [General Purpose IO](https://www.kernel.org/doc/html/v4.17/driver-api/gpio/index.html)
