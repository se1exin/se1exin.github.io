---
layout: post
title: "Building an Automatic Telescope"
date: 2020-03-18 12:11:00 +1000

youtubeId: X1VTMy0LoMA
---

I was recently watching TV and had the thought: what would be required to build
a telescope that could automatically find any planet or star in the sky for you?


Here's my initial sketches from that evening while watching TV (excuse my lack of artistic-ness).

![Sketches of Telescope Ideas][telescope_sketches]


I've wanted to build a robot arm for quite some time (I even attempted it several years ago without success),
and I thought an automatic telescope would incorporate many of the same technical requirements.
For example, both would need mechanisms to rotate along both an X and Y axis, and require fine grain control.
I had some spare servos laying around from that failed robot arm attempt, so I went to putting them to good use.

I didn't have have much to work with, just a spare Arduino and the servos, a recently emptied beer can, some
office supplies, and the trusty blutak. Here's the first prototype that I put together:

{% include youtubeEmbed.html id=page.youtubeId %}

The idea is that the Arduino gets sent some coordinates (that will eventually be the X and Y of the planet in the sky)
and then it moves the X and Y servos to point the pencil (which will eventually be a telescope) in the right direction.

Of course none of this will be accurate or strong enough to support and point a real telescope to exactly
the perfect point in the sky... but you have to start somewhere!

[telescope_sketches]: /assets/img/2020-03-18-telescope-sketches.jpg "Sketches of Telescope Ideas"
