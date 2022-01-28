---
date: 2022-01-27T15:00:00-05:00
title: "Building an ESPHome environmental multi-sensor"
subtitle: "Envirosense ESP8266 take 2"
tags:
  - esp8266 
  - projects
  - ESPHome
  - home assistant
comments: false
draft: false
---

Sometime early last year I replaced my SmartThings home automation hub with a [Home Assistant](https://www.home-assistant.io/) VM running on an Intel NUC.

Not long after that I converted my Envirosense boards ([see previous post](/blog/2020/03/envirosense-esp8266-prometheus-exporter/)) to run the [ESPHome](https://esphome.io/) firmware and started scraping Prometheus metrics from Home Assistant.

There was one thing I wasn't happy about with my original design.
The PIR sensor was frequently showing false positive motion events.  
This seems to have gotten worse with the switch to the ESPHome firmware. It appears to be a known issue that these HC-SR501 modules are easily triggered by Wifi interference. 

So I decided to redesign the board and enclosure to use a better PIR sensor and also added an ambient light sensor.
{{< gallery >}}
{{< figure link="images/envirosense2-with-lid.jpg" caption="Assembled proto-board installed in 3D printed enclosure" caption-position="bottom" >}}
{{< figure link="images/enivirosense2-pcb.jpg" caption="Assembled proto-board installed in case with lid removed" caption-position="bottom" >}}
{{< figure link="images/perfy-board-layout.jpg" caption="Perf+ 2 board layout done with Perfy application" caption-position="bottom" >}}
{{< figure link="images/Fusion360-screen-capture1.jpg" caption="Enclosure design in Fusion 360" caption-position="bottom" >}}
{{< figure link="images/Fusion360-screen-capture2.jpg" caption="Fusion 360 enclosure render" caption-position="bottom" >}}
{{< figure link="images/home-assistant-esphome-screen.jpg" caption="Home Assistant ESPHome screenshot" caption-position="bottom" >}}
{{< /gallery >}}
I used some [Perf+ 2](https://www.crowdsupply.com/ben-wang/perf-2) wire free electronic prototyping board that I had picked up a while back. Unfortunately it looks like this is no longer available anywhere. There's a simple windows .NET based circuit design program called Perfy, that I used for the board layout.

Here's the main parts list:
- NodeMCU ESP8266
- Bosch BME280 breakout board
- Panasonic [EKMC1609111](https://www.digikey.com/en/products/detail/panasonic-electric-works/EKMC1609111/13618562)
- Adafruit [BH1750 Light Sensor](https://www.adafruit.com/product/4681)
- 1 x 47KΩ resistor (*used as a pull-down on the PIR breakout*)

Example ESPHome configuration below:

``` yaml
# EnviroSense ESPHome device configuration
esphome:
  name: sense2

esp8266:
  board: nodemcuv2

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: HIGH

logger:

api:
  password: !secret ota_password

ota:
  password: !secret api_password

switch:
  - platform: restart
    name: Workshop restart

i2c:
  sda: D2
  scl: D1

sensor:
  - platform: bme280
    temperature:
      name: Workshop Temperature
      oversampling: 1x
      accuracy_decimals: 1
    pressure:
      name: Workshop Pressure
      oversampling: 1x
      accuracy_decimals: 0
    humidity:
      name: Workshop Humidity
      oversampling: 1x
      accuracy_decimals: 0
    address: 0x77
    update_interval: 60s

  - platform: bh1750
    name: "Workshop Illuminance"
    address: 0x23
    measurement_duration: 69
    resolution: 4.0
    accuracy_decimals: 0
    update_interval: 60s

  - platform: wifi_signal
    name: Workshop WiFi Signal
    update_interval: 60s

binary_sensor:
  - platform: gpio
    name: Workshop PIR Motion
    pin:
      number: D6
      mode:
        input: true
        pullup: false
    device_class: motion
    filters:
      - delayed_on: 10ms
      - delayed_off: 60000ms
    on_press:
      then:
        - light.turn_on: led
    on_release:
      then:
        - light.turn_off: led

  - platform: status
    name: Workshop Status

output:
  - platform: esp8266_pwm
    id: onboard_led
    pin:
      number: D0
      inverted: true

light:
 - platform: monochromatic
   output: onboard_led
   id: led
```
Note that you'll also need an an additional `secrets.yaml` file that has matching entries for the various `!secret` variables referenced in the above example.
