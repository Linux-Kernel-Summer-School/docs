# Day 1 – Introduction to Linux Kernel

Slides: [day1_slides](./slides/slides_day1.pdf)

## Description

In this lab session we will explore the basics elements of Linux Kernel loadable modules
by starting with the usual `Hello World` module. Then will expand into topics including
printing information, kernel API (timers), simple device driver model and device tree nodes.
By the end of the day, everyone should be able to write, build and interact with kernel modules.

---

## Topics Covered

- Anatomy of a simple Linux kernel module
- Kernel module **compilation and loading/unloading** (`insmod`, `rmmod`)
- Using `printk` and examining kernel logs via serial console and `dmesg`
- Exploring Linux kernel timers API for delayed execution
- Observing **errors** at kernel level (`oops` vs `panic`).
- Touching on platform drivers and their connection with device tree files.

---

## Lab Exercises

> **Note**
>
> Before starting this lab, make sure your **exercises repository** is up to date. All lab exercises are
> located in subdirectories under `drivers/lkss/lab1`.
>
>
> ### If You Already Have the Repository
>
> If you are using the virtual machine described in [Infrastructure page](./infrastructure.md) the repo is already
cloned. 
>
> ```bash
> cd ~/linux  # or navigate to the location where you cloned the repo
> git pull origin dev
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
> Navigate to `drivers/lkss/labs/lab1/` lab directory to begin your work.
>
> ---

### Linux kernel Image compilation

1. **Source the environment**  
   Create `setenv.sh` file with the content below.
   ```bash
   $ cat ~/setenv.sh
   $ export ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
   ```
   and source in the terminal to be used for kernel Image and modules compilation
   ```bash
   $ source ~/setenv.sh
   ```
2. **Compile the kernel Image, dtb** (this needs to be done only once)
   ```bash
   $ cd ~/linux
   # create the default config file
   $ make imx93frdm_defconfig
   # start compilation 
   $ make -j8
   ```
   This will create the kernel `Image` at `~/linux/arch/arm64/boot/Image` and the `dtb`
   at `~/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dtb`

---

### Linux kernel modules compilation

1. **Source the environment**  
   Create `setenv.sh` file with the content below.
   ```bash
   $ cat ~/setenv.sh
   $ export ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
   ```
   and source in the terminal to be used for kernel Image and modules compilation
   ```bash
   $ source ~/setenv.sh
   ```

2. **Select the kernel modules for compilation**  
   ```bash
   $ cd ~/linux/
   $ make menuconfig
   ```
   Select `M` to compile a module. Go to `Device drivers` -> `Linux kernel Summer School Drivers`.

3. **Compile the kernel modules selected**  
   After selecting the  modules, compile them:
   ```bash
   $ cd ~/linux/
   $ make M=drivers/lkss/lab1 modules
   ```
---

#### Prepare board boot

At this point we already have `Image` and `dtb` ready. We now need to prepare the rootfs with the update kernel modules
so that the next boot will see our last compiled modules.

1. **Clone the lkss-utils repo**  
   ```bash
   $ git clone https://github.com/Linux-Kernel-Summer-School/lkss-utils.git
   ```
   This repo will contain some prebuilt `Image` and `dtb`. They work fine to boot the board, but in order to test your
   modules you need to use you own `Image` and `dtb`. They are located under `~/linux/arch/arm64/boot/Image` and
   `~/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dtb`.

2. **Download the rootfs**  
   Download the `rootfs` from [here](https://github.com/Linux-Kernel-Summer-School/buildroot/actions/runs/16078651144/artifacts/3467623530)
   and store it` ~/lkss-utils/2025/`

3. **Copy newly compiled modules to the rootfs**  
   ```bash
   $ cd ~/lkss-utils/2025
   $ ./rootfs_util modules_install ./rootfs.ext2 ~/linux/
   ```
   This will install the linux kernel modules to the rootfs file.

---

#### Boot the board

1. **Boot the board**  
   Use the `boot_imx93.sh` script in order to boot the board.
   ```bash
   $ cd ~/lkss-utils/2025/
   $ ./boot_imx93.sh <path/to/Image> <path/to/device.dtb> <path/to/rootfs.ext2>
   ```
---

### Exercise 1: A simple Hello World kernel module

Explore the contents of `drivers/lkss/labs/lab1/hello.c` to learn the basic anatomy of a Linux Kernel Module. 

1. **Build the kernel module**  
   The following commands builds all the kernel modules for `lab1` and will be used throught the lab.

   ```bash
   $ make M=drivers/lkss/lab1 modules
   ```

2. **Load the kernel module**
   Once built, the compiled module will be `drivers/lkss/labs/lab1/hello.ko`. Update the `rootfs` to contain this file and boot the board.

   ```bash
   $ insmod hello.ko
   ```
   Notice what kernel module function is called and the output on the serial console. What function uses the kernel
   module to display the message?

3. **Unload the module**
   To unload the module use the `rmmod` command. Notice which function is called at unload! 
   ```bash
   $ rmmod hello
   ```
4. **Check kernel log output**
   Use `dmesg` to inspect the kernel log. 

   ```bash
   $ dmesg | tail
   ```
---

### Exercise 2: Kernel Oops and Fault Handling

In this exercise, you’ll explore what happens when something goes wrong inside kernel space—such as
accessing invalid memory or dereferencing a NULL pointer. You'll trigger a kernel oops and observe
how the Linux kernel reports errors, including stack traces and register dumps.

1. **Explore the Oops Source Code**  
   Open the source file: `drivers/lkss/lab/lab1/oops.c`. Read through the code to understand what
   it does to deliberately trigger an error in kernel space.

2. **Load the Module and Trigger the Oops**  
   Build and insert the module on your development board. Observe the kernel output once the module
   executes and triggers the fault.

3. **Analyze the Oops Output**  
   Look at the output from `dmesg` or the system console. Identify the following:  
   - The **stack trace** that shows where the error occurred  
   - The **register dump** that helps identify the system state at the time of the error  
   - The name of the function that caused the crash

---

### Exercise 3: Working with Kernel Timers

In this exercise, you’ll gain hands-on experience with the Linux kernel's timer API by exploring and
modifying a simple timer implementation.

1. **Explore Kernel Timer Usage** Open and study the source file located at: `drivers/lkss/lab1/ex3/timer.c`
   Identify how a timer is initialized and associated with a callback function in the Linux kernel. Identify how the
   timer `timeout` is configured.

2. **Modify the Timer Callback**  
   Locate the `my_timer_callback` function. Extend its behavior by modifying it to **reschedule the
   timer**, ensuring it continues to trigger at a regular interval.

3. **Trigger a Kernel Error**  
   Intentionally cause a kernel error by **dereferencing a NULL pointer** inside the
   `my_timer_callback` function. Observe and note what happens (e.g., kernel panic, stack trace,
   system behavior). This step is designed to demonstrate how the kernel handles critical errors and
   introduce you to kernel debugging.

---

### Exercise 4: Platform drivers and device tree matching

In this exercise, you'll learn how Linux platform drivers bind to hardware devices defined in the
**Device Tree**, and how to read custom properties from those nodes. You'll write a simple driver and
match it to a device using the `compatible` string. Then, you'll read device-specific data using the
kernel's OF (Open Firmware) APIs.

1. **Explore the Driver Source Code**  
   Open the source file: `drivers/lkss/lab1/ex4/lkss_platform_driver.c`. Read through the code to
   understand how a platform driver is structured. Pay close attention to the `sample_probe()`  
   function, where properties are read from the device tree and to the `lkss_sample_of_match` structure
   where compatible strings are defined.

2. **Match the Device Tree Node**  
   Open the device tree source: `arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dts`. Find the node named  
   `lkss-device` under the `simple_bus` node. Confirm that the `compatible` string matches the one  
   used in the driver's `of_match_table`.
   - If the strings don’t match, update either the DTS or the driver to make them consistent.
   - Both should use `lkss,my-lkss-device`.

3. **Build and Load the Module**  
   Compile the driver module and load it on your development board. Use `insmod`
   to insert the module and `dmesg` to observe the output from the probe function.

4. **Verify Property Reading**  
   Confirm that the driver successfully reads the properties defined in the device tree:

   - The `mystring` property should print as a string (e.g., `"hello"`)  
   - The `myint` property should print as an integer (e.g., `42`)

## Resources

- [Linux Kernel Labs - Introduction](https://linux-kernel-labs.github.io/refs/heads/master/labs/introduction.html)
- [Linux Kernel Labs - Basics of writing a Linux kernel module](https://linux-kernel-labs.github.io/refs/heads/master/labs/kernel_modules.html)
- [Linux Kernel Labs - Kernel API](https://linux-kernel-labs.github.io/refs/heads/master/labs/kernel_api.html)
