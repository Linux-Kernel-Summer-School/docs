# Day 4 – Pulse Width Modulation (PWM)

Slides: [day4_slides](./slides/slides_day4.pdf)

## Table of contents

1. [Introduction](#introduction)
2. [PWM API summary](#pwm-api-summary)
3. [Before getting started](#before-getting-started)
4. [Hardware setup](#hardware-setup)
5. [Lab Exercises](#lab-exercises)
6. [Resources](#resources)

---

## PWM API summary

- [pwm\_get()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L489)

Obtain a reference to a `struct pwm_device`.

- [pwm\_put()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L490)

Release resources allocated during `pwm_get()`.

- [pwm\_init\_state()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L192)

Initialize a PWM state.

- [pwm\_set\_relative\_duty\_cycle()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L232)

Set the duty cycle using the following formula: `duty = (period * duty_cycle) / scale`, where `duty_cycle`
and `scale` are the parameters passed to this function and `duty` is the resulting duty cycle.

- [pwm\_apply\_might\_sleep()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L388)

Apply a state to a PWM channel.

- [pwm\_enable()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L421)

Enable a PWM channel.

- [pwm\_disable()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L442)

Disable a PWM channel.

- [struct pwm\_device](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L96)

Describes a PWM channel.

- [struct pwm\_state](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L88)

Describes the state of a PWM channel.

---

## Before getting started

Check that:

✅ you're using the [latest](https://github.com/Linux-Kernel-Summer-School/buildroot/releases/download/lkss-2025-v4/rootfs.ext2) `rootfs.ext2`  
✅ [you've updated the Linux kernel repository](cheatsheet.md#updating-the-Linux-kernel-repository)  
✅ [you've updated the utilities repository](cheatsheet.md#updating-the-utilities-repository)  

---

## Hardware setup

The schematics for this year's edition of LKSS can be found [here](./LKSS_SCHEMATICS.pdf).
For this lab's exercises we're going to use the circuits described in the **LKSS LAB4 CIRCUITS**
sheet (the title of the page is written in the lower right corner).

The PWM chip we'll be working with today has three channels (PWM sources), which are
connected to the board's pins as follows:

| Channel index   |   Pin (Net) name  |
|:---------------:|:-----------------:|
|       0         | EXP\_GPIO\_IO04   |
|       1         | EXP\_GPIO\_IO20   |
|       2         | EXP\_GPIO\_IO12   |

---

## Lab Exercises

Before starting this lab, please make sure you enable **CONFIG_LKSS_DRIVERS_LAB4**.

### Exercise 1: LED dimming using sysfs

Ok, time to get our hands dirty! Before going into the software-related stuff, we
need to prepare our hardware setup. For this exercise we're going to be using the
**LED DIMMER** circuit.

Now, since we've no requirement stating that we need to create a driver
for this application, we can just use the PWM interface exposed to us,
the users, through sysfs.

- Navigate to **/sys/class/pwm** and find the PWM chip we're going to be using. When
you find it, cd into it!

**HINT**: not much of a choice am I right?

**TIP**: if you do a **ls -la** inside the PWM root directory (i.e. **/sys/class/pwm**)
you can see that the directories for each of the PWM chips are actually soft links! If
you look at the paths pointed at by the soft links you'll see that the name of the PWM
chip's devicetree node is also included. Therefore, if you had multiple PWM chips you'd
be able to distinguish between them by looking at the names of the devicetree nodes.

- Export the PWM channel tied to our pin.

Before being able to use a PWM channel, we need to export it by performing a write
to **/sys/class/pwm/pwmchipX/export** (assume the X is the index of our PWM chip).
For example, if we wanted to use channel 5 of PWM chip 4, we would export it using
the following command:

```bash
echo 5 > /sys/class/pwm/pwmchip4/export
```

**HINT**: check the table in the [Hardware setup](#hardware-setup) section to find
the mapping between the name of the board pin and the index of the PWM channel used.

After exporting the PWM channel, you'll notice a new entry has been added to the
directory of our PWM chip. Notice it? If so, cd into it!

- Set the period of the PWM signal. Don't forget we're using nanoseconds!

**HINT**: the name of the file you need to write to should be pretty suggestive

- Set the duty cycle of the PWM signal. Don't forget we're using nanoseconds!

**HINT**: the name of the file you need to write to should be pretty suggestive

**HINT**: careful, you're not allowed to exceed the value of the period!

- OK, we good! Time to fire up the PWM! Enable the PWM by writing to the
appropriate file.

**HINT**: the name of the file you need to write to should be pretty suggestive

- Try to change the value of the duty cycle. Notice any difference in the LED
intensities?

**HINT**: no need to disable the LED before you change the duty cycle!

---

### Exercise 2: LED dimming using PWM API

OK, so our client, Gygel From Ploiesti, doesn't want to use sysfs in his
LED dimming application and, instead, would like us to create a Linux kernel
driver. To get this done as fast as possible, we've asked ChatGPT to generate
the driver for us. Unfortunately, ChatGPT has never heard of PWM before so
all he could do is offer us a skeleton of the driver, which we'll have to
complete ourselves.

- Place the **LED DIMMER** circuit on your breadboards.

- Open the DTS (i.e. `imx93-11x11-frdm.dts`) and enable the `led-dimmer` node.
**Make sure all of the other nodes are disabled.**

- Open the `drivers/lkss/lab3/led_dimmer.c` file.

- **(TODO 1)** Request a PWM channel during the driver's `probe()` function and release it during `remove()`.

**HINT**: you can request a PWM channel using [pwm\_get()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L489)

**HINT**: you can release a PWM channel using [pwm\_put()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L490)

- **(TODO 2)** Set the duty cycle to the value requested by the user. This can range
from 0% to 100%.

**HINT**: you can use [pwm\_set\_relative\_duty\_cycle()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L232)
to avoid doing the math.

- **(TODO 3)** Enable the PWM signal only if this hasn't been already done.

**HINT**: use the state of the PWM channel obtained using [pwm\_init\_state()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L192)

- **(TODO 4)** Disable the PWM channel.

**HINT**: you can disable a PWM channel using [pwm\_disable()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L442)

Once you're done, you can test your driver by following these steps:

- Create an entry in `/dev` for the character device by doing:

```bash
mknod /dev/led_dimmer c 51 0
```

- Run the `test_led_timmer` test script as follows:

```bash
# the value of the duty_cycle parameter will have to be in the [0, 100] range
./lab-tests/lab4/test_led_dimmer /dev/led_dimmer <duty_cycle>
```

---

### Exercise 3: RGB LED dimming

We've just received an e-mail from our imaginary marketing team working for
our imaginary company that there's a client who's oddly interested in RGB
keyboards. Therefore, the marketing team has asked us, the engineers, to
create a driver that is able to control an RGB LED. This way, the client will
be able to eventually create their own, colorful keyboard.

- Place the **RGB LED** circuit on your breadboards.

- Open the DTS (i.e. `imx93-11x11-frdm.dts`) and enable the `pwm-rgb-led` node.
**Make sure all of the other nodes are disabled.**

- Open the `drivers/lkss/lab4/rgb_led.c` file.

- (TODO 1) Request the PWM channels for each of the LEDs: red, blue, green during
the driver's `probe()` operation. Release the PWM channels during `remove()`.

**HINT**: you can request a PWM channel using [pwm\_get()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L489)

**HINT**: you can release a PWM channel using [pwm\_put()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L490)

- (TODO 2) Set the duty cycle to the value requested by the user. This can range
from 0% to 100%.

**HINT**: you can use [pwm\_set\_relative\_duty\_cycle()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L232)
to avoid doing the math.

- (TODO 3) Enable the PWM signal only if this hasn't been already done.

**HINT**: use the state of the PWM channel obtained using [pwm\_init\_state()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L192)

- (TODO 4) Disable the PWM channels

**HINT**: you can disable a PWM channel using [pwm\_disable()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L442)

Once you're done, you can test your driver by following these steps:

- Create an entry in `/dev` for the character device by doing:

```bash
mknod /dev/rgb_led c 52 0
```

- Run the `test_rgb_led` test script as follows:

```bash
./lab-tests/lab4/test_rgb_led /dev/rgb_led
```

---

### Exercise 4: Buzzer piano

Our next client, Florin Salami, likes music a lot. Lately, he's been listening to
a lot of classical piano music and has come across the transcribed version of a very
interesting piece: `Ode To Joy`. Therefore, he's asked us to create a driver, which
he can use to endlessly listen to this amazing piece.

- Place the **BUZZER** circuit on your breadboards (don't use the amplified
version at this point).

- Open the DTS (i.e. `imx93-11x11-frdm.dts`) and enable the `buzzer` node.
**Make sure all of the other nodes are disabled.**

- Open the `drivers/lkss/lab4/buzzer.c` file.

- (TODO 1) Request the PWM channel during the driver's `probe()` function and release it during `remove()`.

**HINT**: you can request a PWM channel using [pwm\_get()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L489)

**HINT**: you can release a PWM channel using [pwm\_put()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L490)

- (TODO 2) Initialize the timer. Make sure you set the callback argument to `timer_fn()` and the flags argument to `0`.

**HINT**: remember [timer\_setup()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/timer.h#L110)?

- (TODO 3) Parse the song provided by the user. For this, you'll have to fill in the `struct buzzer_song`.

- (TODO 4) Start the timer immediatelly.

**HINT**: remember [mod\_timer()](https://elixir.bootlin.com/linux/v6.15.5/source/kernel/time/timer.c#L1208)?

**HINT**: you can trigger the timer immediately by setting the `expires` argument to the
current jiffies.

- (TODO 5) Play the current note by setting the PWM's frequency to the value of the note's
frequency. Use a duty cycle of 50%.

**IMPORTANT**: a note frequency of 0 means pause.

- (TODO 6) Re-arm the timer using the value of `next_jiffies`.

- (TODO 7) Disable the PWM channel.

**HINT**: you can disable a PWM channel using [pwm\_disable()](https://elixir.bootlin.com/linux/v6.15.5/source/include/linux/pwm.h#L442)

Once you're done, you can test your driver by running the following command:

```bash
# you can use ode_to_joy_right or ode_to_joy_left here. We encourage playing the
# right hand over the left one since it's more melodic.
python3 lab-tests/lab4/test_buzzer.py -s lab-tests/lab4/ode_to_joy_right -p /dev/buzzer
```

If everything is working correctly, you can try swapping the current circuit with its
amplified version called **BUZZER (AMPLIFIED)**.

---

## Resources

- [Elixir Cross Referencer](https://elixir.bootlin.com/linux/v6.15.5/source)
