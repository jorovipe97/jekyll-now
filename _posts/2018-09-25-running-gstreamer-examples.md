---
layout: post
title: How to run gstreamer examples in visual studio 2017
published: true
---

In this short post I am going to summarize how to run the gstreamer examples in visual studio 2017.

You can find the **gstreamer examples** in the [following repository](https://github.com/GStreamer/gst-docs).

First, follow the steps on the [original documentation]( https://gstreamer.freedesktop.org/documentation/installing/on-windows.html#InstallingonWindows-Build) for install gstreamer. Make sure to install **gstreamer** and **gstreamer-devel**.

After you install the **gstreamer** you may be tempted to run the examples. The problem is that when you try to run the examples you can get the following linker errors.

![Linker errors](https://imgur.com/shvsKyZ.gif)
