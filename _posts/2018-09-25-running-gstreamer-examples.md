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

This is because If you use **/NODEFAULTLIB** option, for example, to build your program without the C run-time library, you may have to also use **/ENTRY** to specify the entry point (function) in your program ([read the this](https://docs.microsoft.com/en-us/cpp/build/reference/nodefaultlib-ignore-libraries?view=vs-2017)).

…And, of course, the **gstreamer** examples use the **/NODEFAULTLIB** option.
![Linker is using /nodefaultlib option](https://imgur.com/dqMIlJM.gif)

If you want to check it go to **project properties** > **linker** > **input**

As said in the MSDN documentation the solution is to specify the entry point of our program. To do it you need to go to **project properties** > **linker** > **advanced** > **entry point** and then specify the name of the entry point function (in the examples it is named main).
![Specifying the entry point function name](https://imgur.com/myzCbMK.gif)

And that’s all, I hope you can find useful.
