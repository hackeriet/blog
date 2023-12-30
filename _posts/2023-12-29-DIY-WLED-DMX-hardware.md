---
layout: post
title: "DIY Hardware for DMX LED Pot Control with WLED"
author: atluxity
category: hardware
---
# DIY Hardware for DMX LED Pot Control with WLED
Hackeriet at one point aquired a few Fun Generation LED pots, and one of the strong selling arguments for us was the knowledge that they supported DMX.

"DMX (Digital Multiplex) is a protocol used to control devices such as lights or fog machines. The signal is unidirectional, meaning it only travels in one direction; from the controller or first light, all the way to the last. In its most basic form, DMX is just a protocol for lights, like how MIDI is for keyboards or DAW controllers." [1] 

This is not a blogpost about DMX, in this blogpost we will focus on the hardware we put together to be able to control these lights via a web application called WLED.

## Bill of Materials

 - Junction box
 - Sacrificial XLR cable
 - 5v power supply
 - D1 mini (ESP-8266EX board) [2]
 - MAX485 module
 - A few header pins and wire of choice


### Junction box

My thinking is that this box will probably live many hours on a floor and be in risk of poor threatment for much of its life, so I wanted something a bit rugged. Visiting my closest electronics dealer I just found one who felt right for me. It is 80x80x38mm and closed with a single central screw. It has modular soft walls ment for poking cables through.

### Sacrificial XLR cable

I chose to have a bit of a tail out of the box with an XLR-cable rather than a plug into the box. It saves me some space in the box and feels like the more durable and practical solution. This ment I had to cut a cable in two and just make sure to use the correct one (look behind the LED pots, you want the one that goes into the INPUT one)

### 5v power supply

This is what the big old drawer with legacy cables and wall sockets is good for. I found one that used to be for some HP palm-like device I had once, it even came with a plug. That part is optional. I like the idea that if someone stumbles in that cable it will just disconnect the plug and not yank the whole thing apart.

### D1 mini

This is the brain of the operation. ESP-8266 is an amazing ecosphere of geek-enabling technology. It is a lower powered 32-bit RISC CPU running at 160 Mhz and has a built-in WIFI module. Compared to an Arduino with its 8-bit microcontroller, 16Mhz clock speed. The D1 mini supports Micropython, Adruino and nodemcu. We are going to flash this device with a software project called WLED, which also supports DMX although it is made primarily for LED-strips and such.

### MAX485

This is a requirement from the software we are going to load onto the D1 mini. DMX is a serial format, and such is also RS-485. I dont really know how this part works, I just see that its a requirement. I have been unable to find the site of an actuall manufactorer, but there seem to be tens of tens of similar off brands modules that does the same.

## Schematics

![Schmeatics of rthe WLED d1 mini connections](https://blog.hackeriet.no/images/wled-d1-mini-schematic.png)

## Software


I wont copy this part into here, it is better if I just reference the relevant pages.
https://kno.wled.ge/advanced/compiling-wled/
https://kno.wled.ge/interfaces/dmx-output/
Read them both roughly before starting.

I got friendly advice from people around me to let the lamps use the channels 10, 20, 30 and 40, and set them all to 4 channel setup (this should avoid some issues, altought I am blissfully unaware of any details.) and then it was a trick to set the LED setup in the WLED to the correct number of lamps, this is to ensure that animations know how many points of lights it has available to play with.

### References
1 - https://www.sweetwater.com/sweetcare/articles/understanding-dmx/#What%20Is%20DMX
2 - https://www.wemos.cc/en/latest/d1/d1_mini.html
