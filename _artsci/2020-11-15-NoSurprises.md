---
title: 'Pixelator: "No Suprises"'
tags: artsci color video ffmpeg open-cv music
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/ns/ns_thumb.png
cover: /media/ns/ns_thumb.png
---


<!--more-->

# Intro

Using the [video pixelator](./artsci/2020-03-30-VideoPixelator.html) I coded some time ago, I decided to take an artsy go at making a video for Radiohead's "No Surprises".

# Description

The idea behind the video was to use the "lullaby"-like pacing of the song and use as little as possible to drive the music. To do this, I used a clock sound at 60 bpm as a metronome and let it in the song (so that each beat is equal to one second). This also helped in the next idea, that was to add a pixelated layer of the video but updated only at every clock-tick. For this to match up, I recorded the video at a framerate of 30, so that it was a nice division between the bpm and fps. Once that was set, I just had to extract the frames from the video (after clipping the start to match one of the "clicks"):

<img src="/media/ns/ns_original.png" style="width:100%;">

And then throw them all into the pixelator:

<img src="/media/ns/ns_pixel.png" style="width:100%;">

Now, the only remaining trick to pull off was to match the speed of the video to the song speed, which was easy to do by generating the video with [ffmpeg](http://ffmpeg.org/) at a framerate of 1/2.

Finally, I overlaid the pixelated video onto the original one to make the video "smoother":

<div>{%- include extensions/youtube.html id='tfTf2wWyotU' -%}</div>


As a last note, recording the voice on this song was extremely difficult to me because of the reduced speed as compared to the original (had to hold the notes steady for way longer). It took a lot of attempts to get it right.

# Code repo

* **Repository:** [Github repo](https://github.com/Chipdelmal/videoPixelator)
* **Dependencies:**  [ffmpeg](https://www.ffmpeg.org/), [opencv-python](https://pypi.org/project/opencv-python/)
