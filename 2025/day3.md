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

## Lab Exercises

> **Note**
>
> Before starting this lab, make sure your **exercises repository** is up to date. All lab exercises are
> located in subdirectories under `drivers/lkss/lab3`.
>
>
> ### If You Already Have the Repository
>
> If you are using the virtual machine described in [Infrastructure page](./infrastructure.md) the repo is already
cloned. 
>
> ```bash
> cd ~/linux  # or navigate to the location where you cloned the repo
> git pull origin main
> ```
>
> Make sure you are on the correct branch (that is `main` branch).
>
> ### If You Do Not Have the Repository Yet
>
> If you haven't cloned the repository yet, you can do so with the following commands:
>
> ```bash
> git clone https://github.com/Linux-Kernel-Summer-School/linux.git
> cd linux
> ```
>
> Navigate to `drivers/lkss/labs/lab3/` lab directory to begin your work.
>
> ---

### Exercise 1: Writing a Simple GPIO Button Driver

In this exercise, you'll explore a basic GPIO button driver that registers an interrupt handler
when the button is pressed. This example demonstrates how to use GPIO descriptors and platform
drivers in the Linux kernel.

1. **Understand the Driver Structure**

   Review the code in `arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts` and identify:
   - `button` device tree node
   - `gpios` properties under the `button` node.

   Explore the code under `drivers/lkss/labs/lab3/button.c`. This driver requests a GPIO line,
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

3. **Compile the Driver**
   Build the driver module using:

   ```bash
   make M=drivers/lkss/lab3 modules
   ```

4. **Load and Test the Driver**
   Insert the module on the target:
   ```bash
   $ insmod button.ko
   ```
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
time the button is pressed. The source code is provided in `drivers/lkss/labs/lab3/button_led.c`.

The driver retrieves two GPIOs: one for the button (input) and one for the LED (output). It sets
up an interrupt on the button GPIO that triggers on a falling edge. When the interrupt occurs,
the LED should toggle its current state.

1. **Understand the Driver Structure**
    Review the code in `arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts` and identify:
    - `button-led` device tree node
    - `button-gpios` and `led-gpios` properties

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

3. **Compile the Driver**
   Build the driver module using:

   ```bash
   make M=drivers/lkss/lab3 modules
   ```

4. **Load and Test the Driver**
   Insert the module on the target:
   ```bash
   $ insmod button_led.ko
   ```
   Press the button and observe the kernel log:
   ```bash
   $ dmesg  | tail -n 20
   ```

5. **Unload the Module**
   ```bash
   $ rmmod button_led
   ```
---

### Exercise 3: Semaphore-Style LED Driver with Button Control

In this exercise, you will complete and extend a kernel driver that implements a traffic
light-style LED semaphore. Each button press will cycle through Red -> Yellow -> Green -> Red, and
so on. Explore the code at `drivers/lkss/labs/lab3/button_semaphore.c`.

1. **Complete GPIO Retrieval in the Probe Function**
    The provided driver retrieves the button GPIO and the red LED. Your task is to extend this by
	retrieving the remaining two LEDs (yellow and green).
	Edit the `gpio_button_semaphore_probe()` function and add the following:
	- Retrieve the yellow LED GPIO using:
	```c
	yellow_gpio = devm_gpiod_get(&pdev->dev, "yellow-led", GPIOD_OUT_LOW);
    ```  
	- Retrieve the green LED GPIO using:
    ```c
	green_gpio = devm_gpiod_get(&pdev->dev, "green-led", GPIOD_OUT_LOW);
    ```
    Check for errors using IS_ERR() as already done for red.

2. **Implement the IRQ Handler**

	Complete the button_semaphore_irq_handler() function to perform the following:
	- Keep track of the current active LED state.
    - On each button press:
      - Turn off all LEDs.
	  - Turn on the next LED in the sequence: Red->Yellow->Green->Red
	  - Use `gpiod_set_value(led_gpio, 1)` to turn on and 0 to turn off.
	Hint: Use a static variable (e.g., static int state) to track which LED is currently active.

3. **Update the Device Tree***
   Update the device tree to include all four GPIOs: one button and three LEDs. Look for
   `button-semaphore` device tree node

4. **Compile the Driver**
   Build the driver module using:

   ```bash
   make M=drivers/lkss/lab3 modules
   ```

5. **Load and Test the Driver**
   Insert the module on the target:
   ```bash
   $ insmod button_semaphore.ko
   ```
   Press the button and observe the kernel log:
   ```bash
   $ dmesg  | tail -n 20
   ```

6. **Unload the Module**
   ```bash
   $ rmmod button_semaphore
   ```
---

## Resources

- [Interrupts](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html)
- [General Purpose IO](https://www.kernel.org/doc/html/v4.17/driver-api/gpio/index.html)
