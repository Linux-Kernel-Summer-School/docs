# Day 2 – Linux Kernel Modules: Character Device Drivers, GPIO, and User Space Communication

Slides: [day2_slides](./day2_slides)

## Description

This lab introduces key concepts in Linux kernel module development, with a focus on character device drivers and the interaction between user space and kernel space. Through a sequence of practical exercises, the lab explores the use of file operations, or using custom commands via ioctl, and GPIO-based hardware control. The final goal is to build a solid understanding of how user applications can safely communicate with kernel modules to perform data exchange and hardware manipulation.

---

## Topics Covered

- Writing Linux kernel modules
- Implementing character device drivers (`open`, `read`, `write`, `release`)
- Data exchange between user space and kernel space
- Device registration: static and dynamic methods
- Using `ioctl` for custom commands
- GPIO handling in the Linux kernel
- Controlling LEDs (on/off, blinking) through device drivers

---

## Linux and Board Setup

### Initial Linux Kernel Compilation Setup for i.MX93 FRDM (ARM64)

1. **Set up the environment for ARM64 architecture and cross-compilation:**
```bash
$ source setenv
```

2. **Configure the kernel for the i.MX93 FRDM board:**
```bash
$ make imx93frdm_defconfig
```
This only needs to be run once, during the initial setup of your Linux environment for i.MX93 development.

3. **(Optional) Customize the kernel configuration (e.g., for new labs or exercises):**
```bash
$ make menuconfig
```
This needs to be run each time you want to enable or disable a new module (or new lab, or exercise).

4. **Build the kernel and modules:**
```bash
$ make -j8
```
Update the value `8` to match the number of threads supported by your machine.

### Update Root Filesystem with New Kernel Modules

From the directory `/home/student/work/repos/lkss-utils/2025`, run:
```bash
$ ./rootfs_util modules_install /home/student/work/repos/lkss-utils/2025/rootfs.ext2 /home/student/work/repos/linux/
```

### Boot the Board with the New Kernel

Still in `/home/student/work/repos/lkss-utils/2025`, boot the board using:
```bash
$ ./boot_imx93.sh \
  /home/student/work/repos/linux/arch/arm64/boot/Image \
  /home/student/work/repos/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dtb \
  /home/student/work/repos/lkss-utils/2025/rootfs.ext2
```

## Lab Exercises

Before starting this lab, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB2**.

**HINT**: Use `make menuconfig`

### Exercise 1: Simple Character Driver – File Operations

**Objective**: Implement a basic character device driver (`simple_char.c`) that supports `open`, `read`, `write`, and `release` operations.
The module is inserted using `modprobe`, and the device node is manually created using [`mknod`](https://man7.org/linux/man-pages/man1/mknod.1.html).
Test communication between a user space application and the device, verifying basic file operations and kernel logs using `dmesg`.

**Steps**:
1. Create the Kernel Module
- Use the provided `simple_char.c` template. Look into `drivers/lkss/lab2/ex1`
- Implement the four file operations: [`open()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2144), [`read()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2133), [`write()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2134), [`release()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2146). In `read()` function use [`copy_to_user()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/uaccess.h#L217), while in `write()` use [`copy_from_user()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/uaccess.h#L205).
2. Register/Unregister the Device
- In `char_init()` function, use [`register_chrdev()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2927) to register a major number and associate it with the file operations.
- In `char_exit()` function, use [`unregister_chrdev()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2933) to unregister and destroy a cdev.
3. Build the Kernel Image and Modules
4. Insert the Module
- Load it using `modprobe simple_char`.
5. Create the Device Node
- Use `mknod /dev/simple_char c <major> 0`, where `<major>` is the major number from `dmesg`.
6. Test the Driver
- Use `echo` and `cat` to write to and read from the device.
```bash
$ echo abc > /dev/simple_char
$ cat /dev/simple_char
```
- Alternatively, use the provided user space test application - check in `$lab-tests/lab2/test_simple_char`.
```bash
$ ./test_simple_char
```
7. Cleanup
- Use `rmmod simple_char` and `rm /dev/simple_char` when done.
---

### Exercise 2: Character Driver with String Reversal Logic

**Objective**: Create a character device driver (`reverse_char.c`) that automatically registers the device and creates a node in `/dev` using `class_create` and `device_create`. The driver should accept a string from the user (via `write`), reverse it in the kernel, and return the reversed string upon `read`.
This exercise highlights string processing and dynamic device registration.

**Steps**:
1. Use the provided template
- Start from `reverse_char.c` template. Look into `drivers/lkss/lab2/ex2`
2. Implement String Handling
- In `read()`, copy the string from kernel buffer to user buffer (e.g., using [`copy_to_user()`](https://manpages.debian.org/stretch-backports/linux-manual-4.11/__copy_to_user.9) or [`simple_read_from_buffer()`](https://elixir.bootlin.com/linux/v6.15.5/source/fs/libfs.c#L1096))
- Implement the `reverse_string(char *str)` helper function to reverse a string in place.
- In `write()`, copy data from user space to kernel buffer (e.g., using [`copy_from_user()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/uaccess.h#L205) or [`simple_write_to_buffer()`](https://elixir.bootlin.com/linux/v6.15.5/source/fs/libfs.c#L1131)), reverse the stored string and return number of bytes written.
3. Register the Device Dynamically
- Register character device and get dynamically assigned major number using [`register_chrdev()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2927).
- Create a device node using [`class_create()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/base/class.c#L255) and [`device_create()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/base/core.c#L4386).
4. Unregister the device
- Remove the device node and class with [`device_destroy()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/base/core.c#L4462) and [`class_destroy()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/base/class.c#L293).
5. Build and Load the Module
- Compile and load using `modprobe`.
6. Check `/dev` entry
- Confirm the device appears automatically in `/dev/reverse_char`.
7. Test Functionality
- Use `echo` and `cat` to write to and read from the device.
```bash
$ echo abc > /dev/reverse_char
$ cat /dev/reverse_char
```
- Alternatively, use the provided user space test application - check in `$lab-tests/lab2/test_reverse_char`.
```bash
$ ./test_reverse_char
```
8. Cleanup
- Unload the module with `rmmod`.
---

### Exercise 3: GPIO LED Control – On/Off Using Write

Before diving into the software-related tasks, we need to set up our hardware environment.
In the next three exercises, we’ll be working with the i.MX93 FRDM board to control an LED using a GPIO pin. The setup involves a simple circuit that allows us to safely toggle an LED from software.

The schematics for this setup can be found here:
[LKSS_SCHEMATICS](https://github.com/Linux-Kernel-Summer-School/docs/blob/main/2025/LKSS_SCHEMATICS.pdf)

Please check the bottom-right corner of each page for the name of the circuit. For today's lab, we'll be using LKSS LAB2 Circuits, GPIO LED (see page 2).

This diagram will help you verify your connections and understand the circuit layout.

Once your hardware is ready, we’ll move on to writing the software to control the LED via GPIO.

Before starting this exercise, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB2_EX3**.

Also, enable the `gpio-led` node in the device tree file `imx93-11x11-frdm.dts` by changing its status from `disabled` to `okay`.

**Objective**: Develop a character driver (`gpio_led.c`) that controls an LED connected to a GPIO pin. The LED should be turned `on` or `off` based on the values written to the device (0 for off, 1 for on), and on `read()` should report LED state.
This demonstrates how to use kernel GPIO APIs to interact with hardware and expose control through standard file operations.

**Steps**:
1. Use the provided template
- Start from `gpio_led.c` template. Look into `drivers/lkss/lab2/ex3`
2. Initialize a GPIO pin on `.probe` and relese it on `.remove` (e.g., using [`gpiod_get()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/gpio/gpiolib.c#L4715) and [`gpiod_put()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/gpio/gpiolib.c#L5110)).
3. Implement write()
- Map values written to the device (0 = off, 1 = on) to LED control using [`gpiod_set_value()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/gpio/gpiolib.c#L3826).
4. Implement read()
- Return the current LED state using [`gpiod_get_value()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/gpio/gpiolib.c#L3455).
5. Build and Load
- Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.
- Check if the driver has been successfully loaded by running: `dmesg | grep gpio`.
6. Test
- Write 1 or 0 to `/dev/gpio_led` using `echo`.
- Read from the same file using `cat` to confirm LED state.
- Alternatively, use the provided user space test application - check in `$lab-tests/lab2/test_gpio_led`.
```bash
$ ./test_gpio_led
```
8. Cleanup
- Unload the module with `rmmod`.
---

### Exercise 4: GPIO LED Control via ioctl

In this exercise, we’ll continue using the hardware setup from the previous task.
We’ll still be working with the i.MX93 FRDM board to control an LED connected to GPIO pin 5, but this time we’ll interact with the GPIO using `ioctl` system calls instead of the simpler `read`/`write` interface.

Before starting this exercise, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB2_EX4**.

**Objective**: Extend the previous driver to support LED control using ioctl (`gpio_led_ioctl.c`). Implement custom `ioctl` commands for turning the LED on and off.
A user space application will use these commands to control the LED, demonstrating more advanced and structured communication between user space and the kernel.

**Steps**:
1. Use the provided template
- Start from `gpio_led_ioctl.c` template. Look into `drivers/lkss/lab2/ex4`
2. Define `#define` constants for custom `ioctl` commands (LED_ON, LED_OFF).
3. Implement [`unlocked_ioctl()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/fs.h#L2141)
- Use a switch statement to handle `LED_ON` and `LED_OFF`, calling [`gpiod_set_value()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/gpio/gpiolib.c#L3826) accordingly.
4.	Build and Load
- Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.
- Check if the driver has been successfully loaded by running: `dmesg | grep gpio`.
5.	Test
- Use the provided user space test application - check in `$lab-tests/lab2/test_gpio_led_ioctl`.
```bash
$ ./test_gpio_led_ioctl
```
6. Cleanup
- Unload the module with `rmmod`.
---

### Exercise 5: GPIO LED Blinking Using Timer

In this exercise, we’ll continue using the hardware setup from the previous two tasks.

Before starting this exercise, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB2_EX5**.

**Objective**: Extend the driver from Exercise 3 to implement LED blinking functionality using a kernel timer (`gpio_led_blink.c`). The device should respond to standard write commands with a special input to trigger the blinking behavior. The LED toggling is managed inside the kernel using a timer callback function.
This exercise introduces periodic task execution in kernel space and demonstrates how timers can be integrated with character drivers for more advanced GPIO control.

**Steps**:
1. Use the provided template
- Start from `gpio_led_blink.c` template. Look into `drivers/lkss/lab2/ex5`
2. Setup timer on `.probe` and stop it on `.remove` (e.g., using [`timer_setup()`](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/timer.h#L111) and [`timer_delete_sync()`](https://elixir.bootlin.com/linux/v6.15.5/source/kernel/time/timer.c#L1674)).
3. In the timer callback, toggle the LED using [`gpiod_set_value()`](https://elixir.bootlin.com/linux/v6.15.5/source/drivers/gpio/gpiolib.c#L3826).
4. Control Blinking via Write
- Accept special values in `write()`:
- 1: turn LED on
- 0: turn LED off
- b or a keyword (e.g., "blink"): start blinking (e.g., using [`mod_timer()`](https://elixir.bootlin.com/linux/v6.15.5/source/kernel/time/timer.c#L1209)).
5. Stop Blinking
- Implement logic to stop the timer when receiving 0 or another stop signal (e.g., using [`timer_delete_sync()`](https://elixir.bootlin.com/linux/v6.15.5/source/kernel/time/timer.c#L1674)).
6. Build and Load
- Compile the module and load it using `modprobe`, or verify if it's already loaded with `lsmod`.
- Check if the driver has been successfully loaded by running: `dmesg | grep gpio`.
7. Test
- Echo `b` (or `blink`) to the device to start blinking.
- Echo `0` or `1` to stop it.
- Alternatively, use the provided user space test application - check in `$lab-tests/lab2/test_gpio_led_blink`.
```bash
$ ./test_gpio_led_blink
```
8. Cleanup
- Unload the module with `rmmod`.

## Resources

- [Linux Kernel Labs - Character device drivers](https://linux-kernel-labs.github.io/refs/heads/master/labs/device_drivers.html)
- [Creating Device Files](https://fastbitlab.com/creating-device-files/)
- [ioctl based interfaces](https://docs.kernel.org/driver-api/ioctl.html#ioctl-based-interfaces)
- [Kernel Docs - GPIO Descriptor Consumer Interface](https://docs.kernel.org/driver-api/gpio/consumer.html#gpio-descriptor-consumer-interface)
- [Linux Kernel - gpiolib.c](https://elixir.bootlin.com/linux/latest/source/drivers/gpio/gpiolib.c#L4725)
- [Linux Kernel - timer](https://elixir.bootlin.com/linux/latest/source/include/linux/timer.h#L104)
