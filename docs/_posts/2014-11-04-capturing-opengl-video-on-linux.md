---
layout: post
title: Capturing OpenGL video on Linux
tags:
    - demoscene
    - graphics
    - gamedev
excerpt: "While on Windows the dedicated solution for realtime video capture software is [Fraps](http://www.fraps.com/), on Linux there does not seem to be such a dedicated solution. Or is there?"
---
### The problem

While on Windows the dedicated solution for realtime video capture software is [Fraps](http://www.fraps.com/), on Linux there does not seem to be such a dedicated solution. Or is there?

All I wanted was a decent *something* that would allow me to produce full HD, high quality OpenGL captures in Linux that I can afterwards upload to [Youtube](https://wwww.youtube.com) and embed in my blog posts.

Everywhere I looked on the net, [recordmydesktop](http://recordmydesktop.sourceforge.net/about.php) was promoted as a viable substitute but I personally found it cumbersome and difficult to use with respect to capturing the output of my OpenGL experiments.

### The solution

After more investigation and receiving advice from #opengl's *derhass* (Thanks *derhass*!), I settled on a mix of technologies:

#### APITRACE

[Apitrace](http://apitrace.github.io/) is a crossplatform set of tools that allow you to:

* trace graphics API calls to a file
* replay the API calls on any machine and dumping the output of each frame or a frame range to images

This means that we actually have a recording of the OpenGL calls that made up each and every frame ever drawn by our app, including the used data (textures, vbos, etc) which we can selectively choose to re-render at will.

In our mix, *apitrace* is used to capture every OpenGL call made by our application and transform each frame to an image later on.

#### FFMPEG

So *apitrace* enables us to obtain a bunch of images representing every frame ever rendered by our application. Now what? We want to turn them into a movie, obviously, and that's where *ffmpeg* comes in.

<sub>FFMPEG doesn't seem to be part of the official ubuntu repositories for 10.04+ so a good way to add it is by using this [PPA](https://launchpad.net/~jon-severinsson/+archive/ubuntu/ffmpeg).</sub>

### The process

It all starts with obtaining a trace of what the desired program sends to the OpenGL driver. This is obtained by simply prepending your program with `apitrace trace`, e.g:

```
apitrace trace ./stardust
```

This has the effect of capturing and dumping all OpenGL calls into a file called `stardust.trace`. If output file name is not specified, *apitrace* derives it from the captured program's name.

Now that we have our capture, we can review our capture by running:

```
apitrace replay stardust.trace
```

If everything is ok and we are satisfied with the contents of the capture, it's time to make a video out of it

```
apitrace dump-images -o - stardust.trace | \
ffmpeg -y -r 60 -f image2pipe -vcodec ppm -i pipe: -c:v libx264 -preset slow -b:v 8000k -pass 1 -f mp4 /dev/null && \
apitrace dump-images -o - stardust.trace | \
ffmpeg -r 60 -f image2pipe -vcodec ppm -i pipe: -c:v libx264 -preset slow -b:v 8000k -pass 2 -f mp4 stardust.mp4
```

Note that I provide some parameters in the example above: 

1. the video output framerate, specified with the **-r** argument
2. the input trace file, **stardust.trace** in our case
3. the video output filename, **stardust.mp4** in the example above
4. the video bitrate, specified with the **-b:v** argument

This is the result of stardust.mp4 being uploaded to [Youtube](https://www.youtube.com). You should check the HD setting.

<iframe width="640" height="390" src="//www.youtube.com/embed/ESZcf_YJqms" frameborder="0" allowfullscreen></iframe>

### Additional notes

- when attempting to replay your full HD @ 60fps encoded output, you might get some choppy or invalid output. That is rather the effect of a bad player than of the encoded output. Whenever in doubt, try `vlc`!
- try fiddling around with the video bitrate in order to affect the resulting filesize. Please note that this also affects quality.
- try lowering the output framerate for huge resolutions. 30fps should be enough
