---
layout: post
title: Using udev to disable the built in keyboard on your laptop
author: eilefsen
category: linux
---

The built in keyboard on my laptop is *annoying* to use, everything is wrong about it (Dell XPS Plus). To make matters worse, it is also a european layout that i am not used to.
I type at perhaps half my usual speed using the built in keyboard, so i wish to place my regular mechanical keyboard on top and disable the built in keyboard (so i dont accidentally hit the built in keys)

## How to

First you must determine the name of your bluetooth keyboard and the device path of your laptop keyboard.
`cat /proc/bus/input/devices | less`

The name of the bluetooth keyboard in my case is "BT Keyboard", this is probably going to be the same as appears in your DE's bluetooth settings

My laptops built in keyboard is named "AT Translated Set 2 keyboard", you can verify this using something like evtest.
You want the device path for this, located beside the "Sysfs" key (in the output of the above cat command), in my case it is `/devices/platform/i8042/serio0/input/input2/`.

We can inhibit the device by writing a 1 to the file located at `/sys/<your device path>/inhibited`, you can dis-inhibit the device by writing a 0.
This can be done with echo: `echo 1 > /sys/devices/platform/i8042/serio0/input/input2/inhibited`. Upon running that command, your laptop keyboard should be disabled, to reverse it, either reboot or write a 0 to the same file.

Now that we have the mechanism to disable and enable the device on demand, we can write a udev rule for it. (This is probably possible in a less hacky way, but it works)

Here are my particular udev rules:

```udev
SUBSYSTEM=="input", ATTRS{name}=="BT Keyboard", ACTION=="add", RUN+="/usr/bin/sh -c '/usr/bin/echo 1 > /sys/devices/platform/i8042/serio0/input/input2/inhibited'"

SUBSYSTEM=="input", ATTRS{name}=="BT Keyboard", ACTION=="remove", RUN+="/usr/bin/sh -c '/usr/bin/echo 0 > /sys/devices/platform/i8042/serio0/input/input2/inhibited'"
```

each rule (line) matches either the connect or disconnect of the bluetooth keyboard, and each will run the corresponding echo command from before.

You should be able to copy and paste those rules into your own file inside `/etc/udev/rules.d/`, as long as you swap the values for the name and the device path.
