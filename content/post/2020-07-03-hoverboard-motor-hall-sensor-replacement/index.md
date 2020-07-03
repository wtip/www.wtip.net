---
date: 2020-07-03T17:15:56-04:00
title: "Hoverboard motor hall sensor replacement"
# subtitle: 
# bigimg: [{src: "images/envirosense-complete1.jpg", desc: "Path"}]
tags:
  - robot
  - projects
comments: false
draft: false
---

I have a couple Hoverboard ([self-balancing scooter](https://en.wikipedia.org/wiki/Self-balancing_scooter)) motors that I'm intending to use for a project with an [oDrive](https://odriverobotics.com/) motor controller.
One of the motors was causing an `ERROR_ILLEGAL_HALL_STATE` on the oDrive and after swapping the motor channels and some troubleshooting with a multimeter I had determined that 1 of the 3 internal hall-effect sensors was not functioning as expected.  
Digi-Key has a nice article on how these [hall sensors function in BLDC (Brushless DC)](https://www.digikey.com/en/blog/using-bldc-hall-sensors-as-position-encoders-part-1) motors.
<!--more-->
I disconnected the motor from the controller and hooked up a 5V bench supply to the positive and ground of the hall sensor cables. Then I connected a multimeter to ground and one of the 3 hall signal wires at a time. On the working sensors I could rotate the motor on the shaft and cause the multimeter reading to cycle between high and low. I disassembled the motor and determined that the wiring was fine but noticed that the internal resistance of the defective sensor was not the same as the working sensors.

I desoldered the defective sensor with some solder wick and then used some pliers to pull it out of the metal motor housing. It appears to have been held in place with Super Glue (cyanoacrylate) but it wasn't too difficult to remove. Unfortunately I couldn't see any markings on the sensor so I had to make some guesses as to what to replace it with.  
I found a number of e-bike hub motor hall sensor replacement guides that mentioned using a [Honeywell SS41](https://sensing.honeywell.com/honeywell-sensing-bipolar-hall-effect-digital-position-sensor-IC-ss41-l-t2-t3-s-sp-datasheet-32312814-b-en.pdf) replacement.
After reviewing the part dimensions in the datasheet and making some measurements of the sensor I removed, I thought it might be a good fit.

I ordered a handful of the [Honeywell SS41 sensors from Digi-Key](https://www.digikey.com/product-detail/en/SS41/480-1999-ND/701354).

I added some heat shrink around each lead and bent them into shape. With a bit of effort I threaded the sensor into the motor housing and the leads into the PCB and soldered everything back together.

{{< gallery >}}
{{< figure link="images/hall-sensor-side-by-side.jpg" caption="side by side comparison of the old and new sensor" caption-position="bottom" >}}
{{< figure link="images/hoverboard-motor-inside1.jpg" caption="new SS41 installed" caption-position="bottom" >}}
{{< figure link="images/hoverboard-motor-inside2.jpg" caption="SS41 installed and soldered to PCB" caption-position="bottom" >}}
{{< /gallery >}}

It worked!

*If you're not in a hurry you might be able to get a free [sample directly from Honeywell](https://sensing.honeywell.com/SS41-bipolar).*