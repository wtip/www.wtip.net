---
date: 2021-08-16T14:00:00-04:00
title: "Raspberry Pi Camera RTSP Streaming with multiple resolution feeds"
subtitle: "Using GPU accelerated H.264 encoding"
tags:
  - scripts
  - projects
comments: false
draft: false
---

I was looking for a way to stream two H.264 video feeds from an old Raspberry Pi 2 and [Raspberry Pi Camera Module](https://www.raspberrypi.org/products/camera-module-v2/) with the first feed being the highest resolution that I could achieve from the camera and the second being a lower resolution that would be more suitable for analysis.
To monitor the Pi Camera and perform motion and AI object detection with a [Frigate](https://blakeblackshear.github.io/frigate/) NVR system, I wanted the feeds served from an RTSP server.
<!--more-->
{{< figure link="images/picamera-stand.jpg" caption="Raspberry Pi 2 and Raspberry Pi Camera Module in 3D printed stand" caption-position="bottom" >}}

Below are my setup notes for the solution I put together using [rtsp-simple-server](https://github.com/aler9/rtsp-simple-server/), [gstreamer](https://gstreamer.freedesktop.org/documentation/tools/gst-launch.html) and the [picamera](https://picamera.readthedocs.io/) python library.

**1.)** Start with a clean install of [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) (Raspbian 10)

**2.)** Install deb packages
``` bash
apt install -y gstreamer1.0-tools gstreamer1.0-rtsp gstreamer1.0-plugins-bad python3-picamera
```
**3.)** Edit `/boot/config.txt` and add the following two lines at the bottom to enable the camera and increase GPU memory
```bash
start_x=1
gpu_mem=256
```

**4.)** Create and edit `/root/picam_stream.py`
{{< gist wtip 3057c777d2887919af2f1d3872c21405 "picam_stream.py">}}

Depending on your application you might want to adjust some of the picamera library parameters.
It's a good idea to first read up on the [camera sensor modes](https://picamera.readthedocs.io/en/release-1.13/fov.html#sensor-modes) to understand the relationships between Resolution, Frame rates and Field of View.  

Keep in mind the following GPU limitation:  
>The maximum horizontal resolution for default H264 recording is 1920 (this is a limit of the H264 block in the GPU). Any attempt to record H264 video at higher horizontal resolutions will fail.

It's also good to read up on the [MMAL pipeline documentation](https://picamera.readthedocs.io/en/release-1.13/fov.html#pipelines) to understand how the capture ports work.  
In the example above I'm using 3 out of the possible 4 capture ports.  
Port 0 is used for the still image capture that is updated every few seconds.  
Port 1 is used for the highest resoltuion h264 stream.  
Port 2 is used for the resized lower resolution stream.  

The commented out `camera.zoom = (0.0, 0.1, 1.0, 0.75)` option is helpful if you are trying to get a `1920x1080` resolution video using the full width of the sensor.
You'll need to use the `3280x2464` camera resolution at 15 FPS, the zoom option and then resize one of the ports to 1920x1080.

**5.)** Download and extract **[rtsp-simple-server](https://github.com/aler9/rtsp-simple-server/)**.
``` bash
wget https://github.com/aler9/rtsp-simple-server/releases/download/v0.16.4/rtsp-simple-server_v0.16.4_linux_armv7.tar.gz
tar -xvf rtsp-simple-server_v0.16.4_linux_armv7.tar.gz
mv rtsp-simple-server /usr/local/bin/
mv rtsp-simple-server.yml /usr/local/etc/
# review rtsp-simple-server github readme for systemd config
nano /etc/systemd/system/rtsp-simple-server.service
```
**6.)** Edit the `/usr/local/etc/rtsp-simple-server.yml` configuration.  
The main thing that needs to be changed is the section under `paths:`
``` yaml
paths:
  all:
    publishUser: myuser
    publishPass: mypass
  hqstream:
    runOnInit: python3 /root/picam_stream.py
    runOnInitRestart: yes
```
Additionally I've disabled RTMP and HLS support since I don't need it.  
My complete config is available in the [GitHub Gist](https://gist.github.com/wtip/3057c777d2887919af2f1d3872c21405#file-rtsp-simple-server-yml).

Then enable and start the systemd service:
```bash
systemctl enable rtsp-simple-server
systemctl start rtsp-simple-server
```

**7.)** Access the RTSP stream.  
If everything goes well you should be able to access two streams at the following URLs:
`rtsp://IP_Address_OF_PI:8554/hqstream` and 
`rtsp://IP_Address_OF_PI:8554/lqstream`



### Some additional optional steps
- Disable swap  
`apt remove dphys-swapfile`

- Save logs to RAM disk to reduce SD card writes.  
[Install log2ram](https://github.com/azlux/log2ram)

- Install nginx to serve jpg snapshot  
``` bash
apt install nginx
ln -s /dev/shm/camera.jpg /var/www/html/camera.jpg
# Access the snapshot at http://IP_Address_OF_PI/camera.jpg
```
