# Day 2 – Linux Kernel Modules: Character Drivers, GPIO, and User-Space Communication

Slides: [day2_slides](./day2_slides)

## Description

This lab introduces key concepts in Linux kernel module development, with a focus on character drivers and the interaction between user space and kernel space. Through a sequence of practical exercises, the lab explores the use of file operations, ioctl, and GPIO-based hardware control. The final goal is to build a solid understanding of how user applications can safely communicate with kernel modules to perform data exchange and hardware manipulation.

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

## Lab Exercises

Before starting this lab, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB2**.

**HINT**: Use `make menuconfig`

### Exercise 1: Simple Character Driver – File Operations

**Objective**: Implement a basic character device driver (simple_char.c) that supports `open`, `read`, `write`, and `release` operations.
The module is inserted using `modprobe`, and the device node is manually created using `mknod`.
Test communication between a user-space application and the device, verifying basic file operations and kernel logs using `dmesg`.

**Steps**:
1.	Create the Kernel Module
- Use the provided `simple_char.c` template. Look into `drivers/lkss/lab2/ex1`
- Implement the four file operations: `open()`, `read()`, `write()`, `release()`.
2.	Register the Device
- In `char_init()` function, use `register_chrdev()` to register a major number and associate it with the file operations, 
3.	Build the Kernel Image and Modules
4.	Insert the Module
- Load it using `modprobe simple_char`.
5.	Create the Device Node
- Use `mknod /dev/simple_char c <major> 0`, where `<major>` is the major number from `dmesg`.
6.	Test the Driver
- Use `echo` and `cat` to write to and read from the device.
```bash
$ echo abc > /dev/simple_char
$ cat /dev/simple_char
```
- Alternatively, use the provided user-space test application - check in `$lab-tests/lab2/test_simple_char`.
```bash
$ ./test_simple_char
```
7.	Cleanup
- Use `rmmod simple_char` and `rm /dev/simple_char` when done.
---

### Exercise 2: Character Driver with String Reversal Logic

**Objective**: Create a character driver (`reverse_char.c`) that automatically registers the device and creates a node in `/dev` using `class_create` and `device_create`. The driver should accept a string from the user (via `write`), reverse it in the kernel, and return the reversed string upon `read`.

This exercise highlights string processing and dynamic device registration.

**Steps**:
1.	Use the Provided Template
- Start from `reverse_char.c` template. Look into `drivers/lkss/lab2/ex2`
2.	Implement String Handling
- In `read()`, copy the string from kernel buffer to user buffer.
- Implement the `reverse_string(char *str)` helper function to reverse a string in place.
- In `write()`, copy data from user space to kernel buffer, reverse the stored string and return number of bytes written.
3.	Register the Device Dynamically
- Register character device and get dynamically assigned major number using `register_chrdev`.
- Create a device node using `class_create()` and `device_create()`.
4.	Unregister the device
- Remove the device node and class with `device_destroy()` and `class_destroy()`.
5.	Build and Load the Module
- Compile and load using `modprobe`.
6.	Check `/dev` Entry
- Confirm the device appears automatically in `/dev/reverse_char`.
7.	Test Functionality
- Use `echo` and `cat` to write to and read from the device.
```bash
$ echo abc > /dev/reverse_char
$ cat /dev/reverse_char
```
- Alternatively, use the provided user-space test application - check in `$lab-tests/lab2/test_reverse_char`.
```bash
$ ./test_reverse_char
```
8.	Cleanup
- Unload the module with `rmmod`.
---

### Exercise 3: [Optional Exercise Title]

## Resources

- _Provide links to slides, Linux kernel documentation, relevant source files, headers, or external
  guides._
- _Add any notes about kernel versions, dependencies, or hardware setup._

