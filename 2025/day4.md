# Day 4 – Pulse Width Modulation (PWM)

Slides: [day4_slides](./day4_slides)

## Topics Covered

- _List bullet points summarizing key technical concepts introduced during this session._
- _These can include kernel mechanisms, APIs, interfaces, or subsystems discussed._
- _Use this as a quick reference or learning summary._

---

## Introduction

_Provide a short description of what this lab session is about. Mention the scope and practical goals
for the day, what students are expected to understand or achieve by the end of this lab._

---

## Lab Exercises

Before starting this lab, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB4**.

The PWM chip (controller) we'll be working with today has three channels (think of
a channel as a source for a PWM signal), which are connected to our pins as follows:

| Channel index   |   Pin (Net) name  |
|:---------------:|:-----------------:|
|       0         | EXP\_GPIO\_IO04   |
|       1         | EXP\_GPIO\_IO20   |
|       2         | EXP\_GPIO\_IO12   |

### Exercise 1: LED dimming using sysfs

Ok, time to get our hands dirty! Before going into the software-related stuff, we
need to prepare our hardware setup. The schematics used for this year's edition can
be found here (TODO). Please navigate to the **LKSS LAB4 CIRCUITS** page (you can see
the page title in the lower right corner :D). The circuit we're going to be interested
in here is called **LED DIMMER**.

Now, since we've no requirement stating that we need to create a driver
for this application, we can just use the PWM interface exposed to us,
the users, through sysfs.

1. Navigate to **/sys/class/pwm** and find the PWM chip we're going to be using. When
you find it, cd into it!

**HINT**: not much of a choice am I right?

**TIP**: if you do a **ls -la** inside the PWM root directory (i.e. **/sys/class/pwm**)
you can see that the directories for each of the PWM chips are actually soft links! If
you look at the paths pointed to by the soft links you'll see that the name of the name
of the PWM chip's devicetree node is also included. Therefore, if you had multiple PWM
chips you'd be able to distinguish between them by looking at the names of the devicetree
nodes.

2. Export the PWM channel tied to our pin.

Before being able to use a PWM channel, we need to export it by performing a write
to **/sys/class/pwm/pwmchipX/export** (assume the X is the index of our PWM chip).
For example, if we wanted to use channel 5 of PWM chip 4, we could export it using
the following command:

```bash
echo 5 > /sys/class/pwm/pwmchip4/export
```

**HINT**: check the table at the beginning of the [Lab Exercises](#lab-exercises) chapter
to find the correspondence between the name of the pin and the PWM channel used.

After exporting the PWM channel, you'll notice a new entry has been added to the
directory of our PWM chip. Notice it? If so, cd into it!

3. Set the period of the PWM signal. Don't forget we're using nanoseconds!

**HINT**: the name of the file you need to write to should be pretty suggestive

4. Set the duty cycle of the PWM signal. Don't forget we're using nanoseconds!

**HINT**: the name of the file you need to write to should be pretty suggestive
**HINT**: careful, you're not allowed to exceed the value of the period!

5. OK, we good! Time to fire up the PWM! Enable the PWM by writing to the
appropriate file.

**HINT**: the name of the file you need to write to should be pretty suggestive

6. Try to change the value of the duty cycle. Notice any difference in the LED
intensities?

**HINT**: no need to disable the LED before you change the duty cycle!

---

### Exercise 2: LED dimming using PWM API

_Goal: Explain the purpose of the second exercise if applicable._

---

### Exercise 3: RGB LED dimming

_Goal: Additional or challenge-level task to deepen understanding._

---

### Exercise 4: Buzzer piano

_Goal: I wonder what the goal of this might be._

---

## Resources

- _Provide links to slides, Linux kernel documentation, relevant source files, headers, or external
  guides._
- _Add any notes about kernel versions, dependencies, or hardware setup._

