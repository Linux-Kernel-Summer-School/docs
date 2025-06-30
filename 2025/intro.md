# Linux Kernel Summer School – Introduction

Welcome to the **Linux Kernel Summer School**, a 4-day hands-on learning program that aims to
teach you the basics of the **Linux kernel development** with a focus on **embedded systems**
using NXP i.MX93 FRDM platform.

## Why Learn Linux Kernel Development?

Linux is the dominant operating system in embedded, mobile, and server domains. By understanding the
Linux kernel, you gain the ability to:

- Write and debug device drivers
- Understand system-level programming on real hardware
- Get a grasp of software int he embedded world
- Contribute to open-source projects or develop your own

---

## Hardware Platform: FRDM-i.MX93

Throughout this course, we will use the **FRDM-i.MX93** board from NXP, a powerful and flexible
platform built for embedded Linux and AI/ML edge applications.

### 🔧 Key Features:

- **SoC**: NXP i.MX93 with dual Cortex-A55 (Linux-capable) and Cortex-M33 (RTOS or bare-metal)
- **RAM**: Up to 2GB LPDDR4
- **I/O**: USB, UART, GPIO, I2C, SPI, CAN, Ethernet, MIPI-CSI
- **Security**: EdgeLock® Secure Enclave
- **Expansion**: Arduino-compatible headers, mikroBUS™ socket

---

## Overview

Each lab is designed to progressively build your understanding of kernel development — starting from
writing simple modules to building functional drivers that control hardware using real sensors and
GPIOs.

| Day    | Title                                | Description                                                    | Files                             |
|--------|--------------------------------------|----------------------------------------------------------------|-----------------------------------|
| Day 1  | **Introduction to Linux Kernel**     | Overview of kernel structure, module development, and logging  | [Lab](./day1.md), [Slides](./day1_slides) |
| Day 2  | **Simple Character Driver**          | Character driver, file ops, control a buzzer from userspace    | [Lab](./day2.md), [Slides](./day2_slides) |
| Day 3  | **Interacting with Hardware**        | Read sensor values via I2C, use GPIO and kernel timers         | [Lab](./day3.md), [Slides](./day3_slides) |
| Day 4  | **Putting It All Together**          | Full integration: sensors, LEDs, buzzer, GPIO-controlled logic | [Lab](./day4.md), [Slides](./day4_slides) |

---

## Infrastructure

- You can either use your Linux native installation or a virtual machine. Follow the [Infrastructure Guide](./infrastructure.md).

---

##  Goals

By the end of the course, you will:

- Understand kernel space vs userspace
- Write and load basic kernel modules
- Build simple character drivers
- Interact with real hardware using Linux APIs
- Modify and read from the Device Tree

---

Let's dive into the kernel and explore the system from the inside out!
