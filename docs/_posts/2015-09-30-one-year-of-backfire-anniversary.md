---
layout: post
title: One year of Backfire anniversary
tags:
    - graphics
    - gamedev
excerpt: "Last week I celebrated a one-year anniversary since I started developing Backfire. This post contains some reflections on what happened during the last year."
---
Last week I celebrated a one-year anniversary since I started developing Backfire. This post contains some reflections on what happened during the last year.

### Some history

As [Piggy](http://pigrecords.weebly.com/) puts it: 

> Stardust is the working title of a 2d shmup that will probably be called Backfire. It's currently in 
> development for PC, Mac and Linux. The earliest records of Backfire date back to 2007, when Ben first 
> came up with the idea of making a game. 

Long story short, *Backfire* is a 2d shmup game that is in development since September 21, 2014. The aim for *Backfire* is to be a simple, elegant, enjoyable, story-driven, cross-platform 2d game currently developed by two persons:

* [Piggy](http://pigrecords.weebly.com/) who handles the game design, graphics and all artistic decisions and 
* myself who is responsible for the code. 

Should you want to find out more about the project motivation and history, you can do so by reading [this](http://pigrecords.weebly.com/records/first1) post. 

### Some stats

Topic | Value
--- | ---
Programming languages used | C++, Lua, Js, HTML, CSS
Commits | 177
Lines of code | 151285
Tech used | OpenGL, SDL2, glew, glm, lua, mongoose, jQuery, asciidoctor
Builds and runs on | Linux, OSX, Windows (mingw cross-build)
Tools used | SublimeText, CLion, git, CMake


### One year later thoughts

#### Hard things

##### Sticking to a schedule

We (piggy and me) decided from the very beginning that Backfire/Stardust would not be something that we must feel obliged to work on but rather something that we would love to work on whenever we get the time and the proper mood. We're not trying to ship a product on a certain date, nor make money just have fun so there wasn't a need for a schedule.

This resulted in quite a chaotic planning and implementation schedule which would not normally happen in a real life product.

##### Gathering requirements

A consequence of the above was that we didn't have the game design doc finished before we started the actual work which translated in missing requirements. We did however have a couple of design sessions over Skype trying to settle on some game core dynamics but they didn't all get documented in a formal way. The downside was that due to our erratic schedules we found it hard to synchronize on the missing details, some of which are still pending.

##### Maintaining cross-compatibility between platforms

While the code that I write is pretty portable, the platform differences do make an apparition from time to time, like for example when it comes to handling files. Since I didn't want the Windows version to be dependent on mingw but also be able to cross-compile, it meant I had to use platform specific file handling functions for each platform. The code in that area is filled with ugly `#ifdef`s.

#### Good decisions

##### Developing the UI as a webapp

This allowed for rapid prototyping, without the need of having to compile things when only working on the game editor. A simple edit on the css/js files, save, refresh in browser and bam, the results would be visible.

##### Going for a scripting language like Lua

Much like the rapid prototyping reasoning above, having most of the entity behaviour exposed to scripting meant that we could easily and painlessly try out new gameplay ideas by simply editing some brain scripts in the game editor, saving and re-runing the game.


All in all it's been a thrilling year, filled with achievements. Here's one for Backfire! 

Cheers!
