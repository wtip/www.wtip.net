---
date: 2023-07-28T11:00:00-04:00
title: "Picamera2 RTSP Streaming with multiple resolution feeds"
subtitle: "Using Raspberry Pi OS - 11 (bullseye) Python camera library"
tags:
  - scripts
  - projects
comments: false
draft: false
---
This is an update to my [previous blog post](/blog/2021/08/raspberry-pi-camera-rtsp-streaming-with-multiple-resolution-feeds/) where I wrote about how to stream a high and low resolution hardware encoded H.264 video feed from a Raspberry Pi and Camera Module over RTSP.

With the release of Raspberry Pi OS based on Debian Bullseye in November 2021, there were some significant changes to the camera driver that deprecated[^1] the Picamera V1 Python library that I had been using in my previous code. At the time of publishing this post, [Picamera2](https://github.com/raspberrypi/picamera2), which is the new library that is compatible with the Bullseye based OS is still a Beta release. With the release of version 0.3.10, the library finally has support for running "multiple encoders, either on the same or different streams"[^2]. This seemed like an appropriate time upgrade the OS on the Pi and redo the script to work with the new Python library.

Below are my setup notes for a new script using the Picamera2 library, FFmpeg (*instead of gstreamer*) and MediaMTX *(formerly rtsp-simple-server)*.

**1.)** Start with a clean install of [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) - Debian version: 11 (bullseye)

**2.)** Install FFmpeg
``` bash
apt update && apt install ffmpeg
```
The `python3-picamera2` library should already be pre-installed.

**3.)** Create and edit `/root/picam_stream.py`
{{< gist wtip 02835ffb2057d5c189da4ea2ee1516c6 "picam_stream.py">}}

Review the [Picamera2 Library manual](https://datasheets.raspberrypi.com/camera/picamera2-manual.pdf) and [Github project page](https://github.com/raspberrypi/picamera2) for additional documentation.

**4.)** Download and install [MediaMTX](https://github.com/bluenviron/mediamtx) (formerly rtsp-simple-server). Have a look at the [Systemd setup instructions](https://github.com/bluenviron/mediamtx#linux) in the project README.

**5.)** Edit the `/usr/local/etc/mediamtx.yml` configuration.  
The main thing that needs to be changed is the section under `paths:`
``` yaml
paths:
  all:
    source: publisher
    publishUser: myuser
    publishPass: mypass
    publishIPs: []
    readUser:
    readPass:
    readIPs: []
  hqstream:
    runOnInit: python3 /root/picam_stream.py
    runOnInitRestart: yes
```
Additionally I've disabled RTMP and HLS support since I don't need it.  
My complete config is available in the [GitHub Gist](https://gist.github.com/wtip/02835ffb2057d5c189da4ea2ee1516c6#file-mediamtx-yml).

Then enable and start the systemd service:
```bash
systemctl enable mediamtx
systemctl start mediamtx
```
‎  
**6.)** Access the RTSP stream.  
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

[^1]: https://www.raspberrypi.com/news/bullseye-camera-system/
[^2]: https://github.com/raspberrypi/picamera2/releases/tag/v0.3.10