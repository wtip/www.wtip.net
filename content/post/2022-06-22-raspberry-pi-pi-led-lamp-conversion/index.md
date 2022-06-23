---
date: 2022-06-22T15:00:00-04:00
title: "Raspberry Pi Pico powered LED Lamp"
subtitle: "Converting a Dazor P-2134 task lamp to LED"
tags:
  - projects
  - scripts
comments: false
draft: false
---

I recently completed a LED conversion on a Dazor P-2134 fluorescent tube task lamp.
Dazor has been making variations of this lamp [since 1938](https://www.dazor.com/history.html).  
The newly converted lamp has **adjustable brightness and color temperature and uses a 20kHz PWM frequency** to avoid any image banding on digital camera image capture.

<!--more-->

{{< youtube zHLGDwlwsGM >}}

Here's a list of the main components that I used for the conversion:
- Raspberry Pi Pico [SC0915](https://www.digikey.com/en/products/detail/raspberry-pi/SC0915/13624793)
- 4-Channel MOSFET board [DMT10H010LPS (100Vds, 4.5Vgs) with gate driver option](https://www.tindie.com/products/drazzy/4-channel-mosfet-board-with-optional-driver/)
- LM2596 DC-DC Buck Converter
- Auxmer 2400K + 5500K dual color temperature LED strip, 24V, 28.8W/meter, 120LEDs/meter, 95CRI [aliexpress link](https://www.aliexpress.com/item/2251832766293016.html) *use link at your own risk*
- Muzata LED Aluminum channel with diffuser. *I'd probably look for something better next time.*
- Polycarbonate sheet to mount the LED channels and PCBs to
- Power jack 2.1X5.5MM [EJ501A](https://www.digikey.com/en/products/detail/mpd-memory-protection-devices/EJ501A/2439531)
- Rotary Encoders [PEC11R-4215F-S0024](https://www.digikey.com/en/products/detail/bourns-inc/PEC11R-4215F-S0024/4499665). *If I was doing this again I would look for ones with slightly longer threaded collars and shafts. I also ended up not using the push button switch*
- Mean Well 24V 60W power adapter [GST60A24-P1J](https://www.digikey.com/en/products/detail/mean-well-usa-inc/GST60A24-P1J/7703715). *Maximum power draw is around 24 watts so this is overkill.*

{{< gallery >}}
{{< figure link="images/pico-led-mosfet-bench-test.jpg" caption="final LED fixture assembly bench test" caption-position="bottom" >}}
{{< figure link="images/dazor-lamp-base.jpg" caption="base of the Dazor lamp showing how the power jack is connected" caption-position="bottom" >}}
{{< figure link="images/led-fixture-lamp-assembly.jpg" caption="final assembly step before mounting the LED fixture to the lamp" caption-position="bottom" >}}
{{< /gallery >}}

High frequency PWM dimming is needed to avoid image distortion on rolling shutter CMOS image sensors. A more detailed description of the problem is [outlined in this article](https://www.mikewoodconsulting.com/articles/Protocol%20Summer%202011%20-%20Rolling%20Shutters.pdf). The exact minimum frequency needed to eliminate this problem is dependent on the shutter speed and frame rate.
Additionally to avoid any audible hum from the PWM switching you want to use [frequencies above 20kHz](https://www.analog.com/en/technical-articles/avoid-the-audio-band-with-pwm-led-dimming.html).

The product guide for the [4-Channel MOSFET board by Spence Konde](https://spencekonde.github.io/ProductInfo/MOSFETs/Guide) was helpful in getting everything wired up. I did make a hand drawn wiring sketch for this project and can provide it if anyone's interested.


I'm running [CircuitPython on the Pico](https://circuitpython.org/board/raspberry_pi_pico/) and put together the code below for the lamp.

``` python
# SPDX-FileCopyrightText: 2022 William Cooley www.wtip.net
#
# SPDX-License-Identifier: MIT

"""
Use PWM to control the brightness of two LED channels, using readings from two rotary encoders.
One for total brightness and the other to control the color balance.

"""
import board
import time
import pwmio
import digitalio
import rotaryio
import alarm

led_pin_warm = board.GP19
led_pin_cold = board.GP20
enc1_pinA = board.GP0
enc1_pinB = board.GP1
enc2_pinA = board.GP8
enc2_pinB = board.GP9

brightness = 50
color = 25

led_warm = pwmio.PWMOut(led_pin_warm, frequency=20000)
led_cold = pwmio.PWMOut(led_pin_cold, frequency=20000)

encoder1 = rotaryio.IncrementalEncoder(enc1_pinA, enc1_pinB)
encoder2 = rotaryio.IncrementalEncoder(enc2_pinA, enc2_pinB)

enc1last_position = encoder1.position
enc2last_position = encoder2.position

def duty_cycle_mix(brightness, color):
    cold_intensity = int(brightness * 0.01 * (0 + color) * 655.36)
    warm_intensity = int(brightness * 0.01 * (100 - color) * 655.36)
    led_warm.duty_cycle = warm_intensity
    led_cold.duty_cycle = cold_intensity
    print("Warm led " + str(warm_intensity) + " Cold led " + str(cold_intensity))

# startup sequence
for cycle in range(0, brightness):
    duty_cycle_mix(cycle, 0)
    time.sleep(0.02)
for cycle in range(0, color):
    duty_cycle_mix(brightness, cycle)
    time.sleep(0.02)

while True:
    enc1current_position = encoder1.position
    enc1position_change = enc1current_position - enc1last_position
    if enc1position_change > 0:
        brightness = min(100, brightness + 5)
        print("Brightness:" + str(brightness))
        duty_cycle_mix(brightness, color)
        print("Increased position to: " + str(enc1current_position))
        print("--------------------")
    elif enc1position_change < 0:
        brightness = max(0, brightness - 5)
        print("Brightness:" + str(brightness))
        duty_cycle_mix(brightness, color)
        print("Decreased position to: " + str(enc1current_position))
        print("--------------------")
    enc1last_position = enc1current_position

    if brightness == 0 and enc1position_change < 0:
        print("Going to sleep")
        encoder1.deinit()
        pin_alarm1 = alarm.pin.PinAlarm(pin=enc1_pinA, value=False, edge=True, pull=True)
        pin_alarm2 = alarm.pin.PinAlarm(pin=enc1_pinB, value=False, edge=True, pull=True)
        # Do a light sleep until the alarm wakes us.
        alarm.light_sleep_until_alarms(pin_alarm1)
        print("Wakeup from alarm1")
        time.sleep(0.5)
        alarm.light_sleep_until_alarms(pin_alarm2)
        print("Wakeup from alarm2")
        encoder1 = rotaryio.IncrementalEncoder(enc1_pinA, enc1_pinB)
        enc1last_position = encoder1.position


    enc2current_position = encoder2.position
    enc2position_change = enc2current_position - enc2last_position
    if enc2position_change > 0:
        color = min(100, color + 5)
        print("Color:" + str(color))
        duty_cycle_mix(brightness, color)
        print("Increased position to: " + str(enc2current_position))
    elif enc2position_change < 0:
        color = max(0, color - 5)
        print("Color:" + str(color))
        duty_cycle_mix(brightness, color)
        print("Decreased position to: " + str(enc2current_position))
    enc2last_position = enc2current_position
```