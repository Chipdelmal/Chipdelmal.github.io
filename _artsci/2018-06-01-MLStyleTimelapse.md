---
title: Artstyle Timelapse
tags: artsci timelapse machine-learning mathematica video gif artstyle-transfer ffmpeg
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/tl/tl_video.gif
cover: /media/tl/tl_vanGogh.jpg
---


<!--more-->

# Intro

Some time ago I watched the movie ["Loving Vincent"](https://www.youtube.com/watch?v=CGzKnyhYDQI) and was dazzled by the amount of work,m and the quality of the results. Around 65,000 oil-painted frames for an animated movie painted in Van Gogh's style to create a visually stunning movie. After some thinking and playing around with [Mathematica v12](https://www.wolfram.com/language/12/machine-learning-for-images/style-transfer-for-creative-art.html?product=mathematica)'s art-style transfer I wanted to try to automate a copycat of the idea.



<div>{%- include extensions/youtube.html id='CGzKnyhYDQI' -%}</div>

# CodeDev

## Scrape frames

The idea is similar to the one described in the [Machine-Learning Pop Art](./2019-01-15-MLStylePop.html) post. The only difference here, is that we apply the same style-transfer to the individual frames of a video. To do this, we begin with a video clip, and we save the frames to images:

{% highlight bash %}
ffmpeg -i videoClip.mp4 frame%04d.png -hide_banner
{% endhighlight %}

## Apply styling

Once the frames are exported, we define a function to do the processing of the frames:

{% highlight mathematica %}
ApplyStyle[frame_, size_, net_] := Module[{img, resizedNet, netEnc},
  img = ImageResize[frame, size];
  netEnc = NetEncoder[{"Image", ImageDimensions[img]}];
  resizedNet = NetReplacePart[net, {"Input" -> netEnc, "Output" -> NetDecoder[{"Image"}]}];
  resizedNet[img]
]
{% endhighlight %}

With this function defined, we can run the styling process on our images:

{% highlight mathematica %}
fileNames = FileNames["*.jpg", Directory[] <> "/TimeLapse/"][[1 ;; All]];
images = Import[#] & /@ fileNames;
frames = ParallelMap[ApplyStyle[#, 500, net] &, images];
{% endhighlight %}

And then export the results to disk:

## Export to disk

{% highlight mathematica %}
Export["./Output/PreTrained/Style" <> ToString[STYLE] <> "/" <> StringPadLeft[#[[1]], 4, "0"] <> ".png", #[[2]]] & /@ Transpose[{ToString /@ Range[1, Length[frames]], frames}];
{% endhighlight %}

This will output a series of image files to our disk:


<img src="/media/tl/tl_frames.jpg" style="width:100%;">

## Re-assemble video

So, to put them back together in a *GIF*, we run the following [FFmpeg](https://www.ffmpeg.org/) commands (a great explanation of why to use these commands can be found [here](https://engineering.giphy.com/how-to-make-gifs-with-ffmpeg/)):

{% highlight bash %}
ffmpeg -i ArtStyleML.mp4 -i palette.png -r 15 -lavfi paletteuse image.gif
ffmpeg -ss 2.6 -i ArtStyleML.mp4 -i palette.png -filter_complex "fps=5,scale=500:-1flags=lanczos[x];[x][1:v]paletteuse" sixthtry.gif
{% endhighlight %}


<img src="/media/tl/tl_video.gif" style="width:100%;">

And we have a simple approximation of the process that we can run in the computer.

# Further Thoughts

Several things to point out here. Even though the process works, it's a bit contrived and extremely inefficient in terms of computation and time. Jumping from  [FFmpeg](https://www.ffmpeg.org/) to [Mathematica v12](https://www.wolfram.com/language/12/machine-learning-for-images/style-transfer-for-creative-art.html?product=mathematica) is hardly the best approach to do things. Even though, part of the process could be done by calling _bash_ scripts from [Mathematica v12](https://www.wolfram.com/language/12/machine-learning-for-images/style-transfer-for-creative-art.html?product=mathematica), a better alternative would be to use python with the [artistic-videos
](https://youtu.be/vQk_Sfl7kSc) implementation (their [arXiv paper is also definitely worth reading](https://arxiv.org/pdf/1604.08610.pdf)) which fixes one of the main problems with the approach I took: temporal dependency. In the naïve implementation I provided neglects the temporal dependency between frames, so artifacts appear in some regions of the frames (specially in the sky), as can be seen in the Munch style-transfer:

<img src="/media/tl/tl_munch.gif" style="width:100%;">

# Code Repo

* Repository: [Github repo](https://github.com/Chipdelmal/ArtStyleTimeLapseTransfer)
* Dependencies: [Mathematica v12](https://www.wolfram.com/language/12/machine-learning-for-images/style-transfer-for-creative-art.html?product=mathematica), [FFmpeg](https://www.ffmpeg.org/)
