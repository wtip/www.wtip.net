---
date: 2022-02-08T18:00:00-05:00
title: "Home Assistant Dashboard on Polycom VVX"
subtitle: "Showing sensor states on a VOIP desk phone idle screen"
tags:
  - home assistant
  - projects
comments: false
draft: false
---

I've had a Polycom VVX 500 "business media phone" on my desk since 2016 and have only used it for making phone calls since then.
However this series of Polycom VVX phones also has a built in limited-capability web browser.
The phone can display a non interactive web page on it's idle screen and an interactive page that's accessible through the phone's "Applications" menu.

On a side note, I think I've been running a VOIP PBX for personal or business use, more or less continuously since 2006. Starting with open source systems based on Asterisk, then FreeSWITCH and then in 2019 I migrated to a proprietary 3CX Phone System. The best part for me about 3CX has been that it's painless to configure and keep up to date.

{{< figure link="images/poly-vvx_500-browser-idle-screen.jpg" caption="Polycom VVX 500 showing Home Assistant sensor states" caption-position="bottom" >}}

When I started using Home Assistant last year I thought it might be cool to build a simple web dashboard that could run on the VVX phone and display sensor states.

That's what brought me to making the [Poly VVX web service add-on](https://github.com/wtip/poly-vvx-web_HAOS-addon) for Home Assistant.
_Add-ons for Home Assistant allow the user to extend the functionality around Home Assistant._ [^1] 

It's a Python Flask web application that exposes a list of user defined Home Assistant sensor states in a simple HTML table for Polycom VVX phones to display using their microbrowser.

For more details check out the [Poly VVX web service add-on repository on GitHub](https://github.com/wtip/poly-vvx-web_HAOS-addon).



[^1]: https://developers.home-assistant.io/docs/add-ons