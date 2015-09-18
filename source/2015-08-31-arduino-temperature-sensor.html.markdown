---
title: Arduino Temperature Sensor
date: 2015-08-31 14:34 UTC
layout: post
tags: arduino
---

Because we live on a boat, my wife and I have recently had trouble keeping the temperature consistent in our son’s bedroom. The ideal temperature for infants to sleep in is apparently 68°F, but in the late summer when we’ve put him down, the nights tend to hover around 73°, then drop precipitously to around 64° around 5am. Consequently, my wife tends to wake up around 4–5 times a night to either turn a fan or a heater on or off. We have a thermometer in the room, but it’s a cheap one without a light, so she has to go into the room, try to read the thermometer, inevitably take it out of the room, turn on a light, hurt her sleepy eyes, then read the temperature, and get the thermometer back next to his bed without waking him up. I wanted to fix this problem, and I’ve long been looking for an excuse to hack on an Arduino project. [READMORE]

Luckily, [my work](http://wintr.us) happened to have a [Sparkfun Electronics Inventor’s Kit](https://www.sparkfun.com/products/12060) for Arduino, which came with an Arduino Uno, a solderless breadboard, and several LEDs, sensors, and jumper wires. I had some time this last Friday evening, so I figured I’d get set up, run into a bunch of issues, and maybe have time write some code to blink an LED before it was time to head for bed. Turns out the Arduino IDE is incredibly easy to set up and install, thanks to some great documentation. As a result, I had a blinking LED done within 10 minutes or so. So I figured, let’s see if we can make something useful.

I had already speced the MVP of the project. Rather than making my wife deal with the thermometer at all, ideally, she’d be able to just pull aside the curtain to my son’s bedroom and see at a glance not what the temperature was, but if it was outside of the range we cared about (66-73). This turned out to be relatively straightforward by combining two of the tutorials that came with the kit: turning on an LED and using the temperature sensor.

I initially thought it would be great to use the tri-color LED to turn on blue if it was too cold, or red if it was too hot. I played around with it for about 15 minutes before hypothesizing it had been burnt out by the person using the kit before me (most likely by running too much voltage through it without a resistor). So I did the next best thing: read the temperature every 1.5s. If the temperature is below 66, turn on a yellow LED. If it’s above 73, turn on a red LED.

Here was the code I used to achieve that:

<script src="https://gist.github.com/thenickcox/17aa08fa95478bb680cf.js">
</script>

And here is the video of it in action.

<iframe width="420" height="315" src="https://www.youtube.com/embed/StsFU_14DNE" frameborder="0" allowfullscreen='allowfullscreen'> </iframe>

If you want to make it, here is the Fritzing diagram.

<img src="/images/temperature_bb.svg"></img>

## Iteration

My goal wil be to connect the Arduino to a Raspberry Pi with a Rails app running on it that will text us if the temperature goes out of range. I’ll be planning to work on that in the next few weeks. (You don’t have a ton of free time when you have a baby.) I’ll post the results here.

