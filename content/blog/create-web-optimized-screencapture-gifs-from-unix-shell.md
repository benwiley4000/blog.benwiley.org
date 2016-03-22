+++
date = "2016-03-21T06:48:09-04:00"
draft = false
title = "Create Web-Optimized Screencapture GIFs from the Unix Shell"
slug = "create-web-optimized-screencapture-gifs-from-unix-shell"
contenttype = "article"
image = "img/terminalscreen.jpg"

+++

Last week I set out to finally build myself a web portfolio on [my site](http://benwiley.org/) --- a task that has been delayed numerous times since I went more-or-less "full-time" last Fall. I knew I had to have screenshots, since those are one of the most important ways you can grab people's attention before they resume totally ignoring you. But it can be hard to capture the essence of a dynamic web app in a single frame.

So I decided to embed GIFs of my web applications into my site! Not just any GIFs. You know what's the worst? GIFs that load slowly and look bad. I knew I would need GIFs that:

* show my apps off nicely (nice image quality/minimal compression artifacts)
* download quickly
* are concise, and have well-thought-out starting and ending points.

There are plenty of paid applications out there to help pull this off, but why pay money when I can just use free tools in the Unix shell?

Commence the nerd quest.


### Screencapturing with recordMyDesktop

Before I make a GIF from a screen recording I have to make the recording! It's easy to do this on Linux using a command line tool called [recordMyDesktop](http://recordmydesktop.sourceforge.net/about.php). There's also a graphical frontend available called gtk-recordMyDesktop. To install them both with `apt-get`, use:

{{<highlight bash>}}
$ apt-get install recordmydesktop gtk-recordmydesktop
{{</highlight>}}

In order to initiate screencapture, run `recordmydesktop`. Use `Alt`+`Tab` to return to the terminal window when finished, and press `Ctrl`+`C` to stop capturing and begin encoding. The longer you record, the longer encoding lasts (and it can last awhile), so plan shots before recording!

![recordMyDesktop running in the shell](img/recordmydesktop3.jpg)

I capture recordings that last between one and four seconds, and I use the graphical version of recordMyDesktop. Since I don't need sound, I disable it.

![GTK-recordMyDesktop](img/gtk-recordmydesktop1.jpg)

**Don't stop recording with `Ctrl`+`C` if you're using the graphical version! You will lose your progress. Use the red stop button to finish recording.**

![Quit GTK-recordMyDesktop by right-clicking the red stop button](img/gtk-recordmydesktop2.jpg)

**On a Mac?** QuickTime Player has a [built-in function](https://support.apple.com/kb/PH5882?locale=en_US) for recording your screen.

### Separate video into frames

Next I need to separate my video into individual jpeg frames, which will become the GIF. On Linux a number of tools can accomplish this, but one is [MPlayer](http://www.mplayerhq.hu/design7/news.html) - a movie player that has a command line interface. To install MPlayer I run:

{{<highlight bash>}}
$ apt-get install mplayer
{{</highlight>}}

Then I output your frames into a new directory:

{{<highlight bash>}}
$ mplayer -ao null myscreencapture.ogv -vo jpeg:outdir=frames
{{</highlight>}}

The new directory will now contain each individual frame from the screencapture I created a moment ago. If anything needs to be trimmed off the beginning of end of your capture (or even the middle!), **this is the time** to delete those frames. I set the frames up so the player will start and end in a similar position (to help with the looping effect).

**On a Mac?** You can use the Unix utility MPlayer relies on, [FFmpeg](https://www.ffmpeg.org/), to split your video into frames. [This answer from Super User](http://superuser.com/a/391146) should help you out.

### Crop frames

For the next few steps we'll be using a tool called [ImageMagick](http://www.imagemagick.org/script/binary-releases.php). It can be installed with:

{{<highlight bash>}}
$ apt-get install imagemagick
{{</highlight>}}

Mac users can download a simplified installer for ImageMagick [here](http://cactuslab.com/imagemagick/).

ImageMagick comes with a tool called `mogrify` that is able to powerfully batch-process images in ways that you would normally have to do one-at-a-time in visual image editors. Since I only want to keep a 860x860px area of my screencapture, I can use `mogrify` to crop each of my frames in the same place. Doing this job without the command line could take hours, if not days!

I need to open one of my frames up in an image editor like [GIMP](https://www.gimp.org/downloads/) so I can find the X and Y coordinates of the top-left corner of my capture area.

![Finding crop coordinates in GIMP](img/clusterjunkcrop.jpg)

Once I find those coordinates, I copy all my frames to a fresh directory (to be safe) and then crop them all in place:

{{<highlight bash>}}
$ mkdir cropped_frames
$ cp frames/* cropped_frames/
$ mogrify -crop 860x860+524+164 cropped_frames/*.*
{{</highlight>}}

All cropped!

### Scale frames

We're going to use `mogrify` again, but this time to scale our frames down a bit. I only want my final GIF to be about 300x300 pixels, so it can download in the browser more quickly. It's unnecessary to be zoomed all the way in, and we'll thank ourselves later when the smaller resolution means we don't have to sacrifice as much image quality. GIFs can be expensive!

Let's copy our frames (just to be safe again), then scale them in place:

{{<highlight bash>}}
$ mkdir scaled_frames
$ cp cropped_frames/* scaled_frames/
$ mogrify -scale 300x300 scaled_frames/*.*
{{</highlight>}}

### Make a GIF!

We're finally ready to make a GIF! This time we're using ImageMagick's `convert` command, which can do *a lot* of things, one of which is turn a bunch of frames into one GIF.

{{<highlight bash>}}
$ convert -delay 6 frames_crop_scaled/* animation.gif
{{</highlight>}}

I set the `-delay` flag's value to 6, or six tenths of a second per frame, to match the screencapture's refresh rate (60 frames per second). You can make that value smaller for a faster GIF, or larger for a slower one.

### Optimize our GIF

Our GIF's file size is now pretty small, thanks to cropping and scaling it earlier, but it could still be smaller. ImageMagick comes with tools for this too, but it's difficult to find settings that won't degrade the image quality a lot. For some types of footage (like people), that might be OK, but for a UI with solid swatches of color, that's not ideal.

Instead, we're going to use *another* command line tool called [Gifsicle](https://www.lcdf.org/gifsicle/), which comes with some smart optimization settings that are simple to apply.

{{<highlight bash>}}
$ apt-get install gifsicle
{{</highlight>}}

Check the Gifsicle website (above) for Mac installation options, including Homebrew.

We're going to limit our GIF to 256 colors, and use Gifsicle's most advanced image optimization algorithm (you can try out reducing the color count even more to further reduce the file size).

{{<highlight bash>}}
$ gifsicle --colors 256 -O3 < animation.gif > animation_optimized.gif
{{</highlight>}}

![Optimized, finished GIF](img/clusterjunk_optimized.gif)

Boom --- a web-optimized GIF!

Follow me on [Twitter](https://twitter.com/benwiley4000) and [GitHub](https://github.com/benwiley4000) for more.

### Links that helped me

[1] [Blog tvlooy --- Convert OGV to GIF](https://ctors.net/2015/07/25/ogv_to_gif)

[2] [grep Linux --- Mogrify: scale and crop multiple images in one command](http://www.greplinux.net/2012/08/mogrify-scale-and-crop-multiple-images.html)

[3] [parker higgins dot net --- howto: create an animated gif from a video with command line tools](http://parkerhiggins.net/2012/10/howto-create-an-animated-gif-from-a-video-with-command-line-tools/) (I actually found this after I'd finished writing this article, while looking for a Mac port of Gifsicle. He has a remarkably similar process to mine, so it's worth checking out for the differences.)
