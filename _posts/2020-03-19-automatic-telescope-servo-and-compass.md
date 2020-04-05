---
layout: post
title: "Automatic Telescope Pt 2: Compass and Stepper Motor"
date: 2020-03-19 12:00:00 +1000

youtubeId: r3_Ks2S8_R8
---

I've recently starting [building an automatic telescope](/2020/03/18/building-an-automatic-telescope.html),
and am very much still in the prototyping phase.

Since [my last post](/2020/03/18/building-an-automatic-telescope.html), I thought I would investigate
adding a compass sensor to the mix so that I can start implementing the X axis control of the telescope.
The theory is that if I can find the position of the telescope on the X axis relative to north,
I can use that position as a reference point to find the &theta; position of a planet or star.
I'll figure out the Y axis later.

![Using a compass sensor to find a position on the X axis][compass_illustration]

I ran out to Jaycar (Australia's version of Radioshack) and picked up a cheap compass sensor. While
I was there I came across a cheap stepper motor which I decided I would try out instead of the servo to see
if there was any gain in control and accuracy.

Here are the parts that I bought:
- [($17.95) 3 Axis Compass Magnetometer Module](https://www.jaycar.com.au/arduino-compatible-3-axis-compass-magnetometer-module/p/XC4496)
- [($9.95) 5V Stepper Motor with Controller](https://www.jaycar.com.au/arduino-compatible-5v-stepper-motor-with-controller/p/XC4458)
- [($9.95) Aluminium Hub with Set Screws](https://www.jaycar.com.au/aluminium-hub-with-set-screws/p/YG2784) (This one was way over priced...)

After some modification to the beer-can-and-pencil prototype I had retrofitted the stepper motor in place of the old servo:

![Stepper Motor Beer Can][stepper_beer_can]

Now that the stepper motor had been attached to the beer can, it was time to add the compass module to the Arduino,
which will in turn tell the motor to rotate either clockwise or anti-clockwise until the compass points
at the desired point.

Here's a video of it in action, with the stepper trying to find magnetic north.
In case you were wondering and can't see it in the video, the compass is blutak-ed to the top of the can,
with the wires running through the middle of the can.

{% include youtubeEmbed.html id=page.youtubeId %}

> The video is a bit long (sorry). Here's the breakdown in case you want to skip ahead:
- The first 38 seconds is compass calibration.
- The full next minute (00:38 - 1:40) is rotating clockwise to find north.
- After that (around 1:40) you can see the stepper motor re-adjusting to find north as I disorient and move it around.

### A few take-aways from this experiment:

As you might notice, the stepper motor is super slow, which could be excused if it had a high resolution,
but it does not. I'll probably get a better stepper motor for the X axis soon - I haven't used stepper
motors in the past, so I need to do some more research before spending any more money in this department.

Additionally, the compass module is quite inaccurate and I can find very little documentation for it - so again I'll probably try to find a better compass sensor. Another thing to consider is that the Y Axis control will very likely need some sensors such as an accelerometer and/or gyro. I actually
have a spare MPU-6050 which I could use, but it's a pain-in-the-butt to write code for.
I saw in passing on the internet that you can buy 9-axis sensors (compass, acceleration, gyro)
which can do a lot of computation on-board - so I might just get one of those.


Looks like I need to do some more research and get some better parts.


[stepper_beer_can]: /assets/img/2020-03-19-stepper-beer-can.jpg "Stepper Motor Beer Can"
[compass_illustration]: /assets/img/2020-03-19-compass-illustration.jpg "Using a compass sensor to find a position on the X axis"
