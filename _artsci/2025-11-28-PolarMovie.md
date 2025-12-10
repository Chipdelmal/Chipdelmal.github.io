---
title: "Movies Colors and Waveform"
tags: dataviz clustering artsci scikit-learn audio color video image ffmpeg
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/polar/back.jpg
cover: /media/polar/nimona_r.jpg
---

Improving the dominant color visualizations and adding audio information.

<br>

<!--more-->

# Intro

I was never completely happy with the way the [dominant color fingerprints](./artsci/2021-07-12-MoviesDominantColors.html) were displayed, as it made it too abstract and removed from the original. Initially I tried to add a couple of callouts on top of the original rectangular display but I did not quite like the way they looked, so I started thinking how to improve it. Additionally, I had the idea of incorporating the audio information somehow. With these components and requirements at hand I decided to launch myself into improving the [original implementation](./artsci/2021-07-12-MoviesDominantColors.html).

# Code Dev

Most of the code this time is a combination of scripts developed in the [eclipse waveforms](./artsci/2022-10-15-EclipseWaveforms.html) and [dominant colors](./artsci/2021-07-12-MoviesDominantColors.html) implementations, so we will be referring and extending them throughout this post.

## Screencaps

The first thing to implement was the export of the movie screencaps to disk so that we can later grab their dominant colors and plot some selection into our visualization. These tasks had already been coded in a [previous post's](./artsci/2021-07-12-MoviesDominantColors.html) [codebase](https://github.com/Chipdelmal/moviesColorFingerprint/blob/master/exportFrames.py) through [ffmpeg](https://www.ffmpeg.org/), so that was already in good place and just needed a couple of tweaks. The main thing in this version to process a rescaled version of the video (to make the processing faster I am using a `480p30` rescaled using [Handbrake](https://handbrake.fr/)), and to limit the number of frames to a smaller number (for these examples, we will use 350 frames):

```python
# Calculating fps to match required number of frames --------------------------
probe = ffmpeg.probe(path.join(IN_PATH, FILE))
vInfo = next(s for s in probe['streams'] if s['codec_type'] == 'video')
framesNumMovie = int(vInfo['nb_frames'])
framerate = eval(vInfo['avg_frame_rate'])
fps = FRAMES_NUM/framesNumMovie*framerate
# Export frames ---------------------------------------------------------------
if (not folderExists) or (OVW):
    os.system(
        "ffmpeg -loglevel info "
        + "-i " + path.join(IN_PATH, FILE) + " "
        + "-vf fps=" + str(fps) + " "
        + f"-s {SIZE[0]}x{SIZE[1]} "
        + path.join(OUT_PATH, FILE.split(".")[0] + "%04d.png ")
        + "-hide_banner"
    )
```

## Audio

For some similar applications in the past, I had used the [pydub library](https://github.com/jiaaro/pydub), so I took some bits and pieces from [those scripts](https://github.com/Chipdelmal/WaveArt) in this application as well (for more information, have a look at my [original waveforms post](./artsci/2022-10-15-EclipseWaveforms.html)). This processing is not so different from the original code but with this time around we want to aggregate the sound entries into larger groups, so that we can plot bars instead of the full soundwave, which makes it look a bit better. To do this, we have to grab a subset of “frames” from the soundwave and take a summary statistic around them ([codelines here](https://github.com/Chipdelmal/moviesColorFingerprint/blob/e7e8466045f77422df41ae62525534582c92070b/wavePrint.py#L62)). In this case, for example, we will grab 350 evenly-spaced snapshots of the soundwave and take the mean around them.

## Dominant Color

```python
def dominantImage(
        img, domColNum, clustersNum, maxIter=100
    ):
    (frame, shp) = img
    flatFrame = frame.reshape([1, shp[0] * shp[1], 3])[0]
    kMeansCall = KMeans(n_clusters=clustersNum, max_iter=maxIter)
    kmeans = kMeansCall.fit(flatFrame)
    frequencies = {
        key: len(list(group)) for 
        (key, group) in groupby(sorted(kmeans.labels_))
    }
    dominant = dict(sorted(
      frequencies.items(), 
      key=itemgetter(1), reverse=True
    )[:domColNum])
    dominantKeys = list(dominant.keys())
    palette = [kmeans.cluster_centers_[j] for j in dominantKeys]
    myiter = cycle(palette)
    palettePad = [next(myiter) for _ in range(domColNum)]
    colors = [rescaleColor(color) for color in palettePad]
    return colors
```

## Strip Plot

The first version of the visualization was relatively straightforward as it was just a set of line plots at set intervals on the x-axis with the height corresponding to the averages of the soundwaves with the color corresponding to the obtained dominant color at the same time point. The only trick in this one was to add round line ends to make them look nicer.

The one last thing to add to this plot was the inclusion of the movie screencaps. This was achieved by adding an `AnnotationBbox` that holds the imported image corresponding to the frame in the soundwave. We wouldn't want to add all the corresponding frames, so we ad some conditional to plot every Nth one in two rows.

```python
(fig, ax) = plt.subplots(figsize=(20, 4))
for (ix, sndHeight) in enumerate(sndFrames):
    # Plot waveform -----------------------------------------------------------
    (y, x) = ([-YOFFSET, sndHeight], [ix*BAR_SPACING, ix*BAR_SPACING])
    ax.plot(
        x, y, 
        lw=LW, color=hexList[ix][0], 
        solid_capstyle='round', zorder=1
    )
    # Plot image --------------------------------------------------------------
    if ((SFRAME+ix)%DFRAMES==0):
        img = np.rot90(image.imread(filepaths[ix]), k=ROTATION, axes=(1, 0))
        imagebox = OffsetImage(img, zoom=ZOOM/1.5)
        off = OFFSETS[::][offCounter%len(OFFSETS)]       
        ab = AnnotationBbox(
            imagebox, (ix*BAR_SPACING, off), 
            frameon=False, box_alignment=(0.5, 0.5), 
        )
        ax.add_artist(ab)
        offCounter = offCounter + 1
        # Add callout line ----------------------------------------------------
        ax.plot(
            x, [0, off], 
            lw=CW, color=hexList[ix][0], 
            solid_capstyle='round', ls=':', zorder=1
        )
ax.set_xlim(XRANGE[0], sndFrames.shape[0]*BAR_SPACING+XRANGE[1])
ax.set_ylim(YRANGE[0], np.max(sndFrames)+YRANGE[1])
ax.set_axis_off()
```

Some examples of this idea in action would be:

<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/polar/nimona.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/polar/aladdin.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/polar/supermario.jpg" style="width:100%;"></div>
  </div>
  <!-- <div class="swiper__pagination"></div> -->
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
  <!-- <div class="swiper-scrollbar"></div> -->
</div>

## Polar Plot

I was still not entirely happy with the visualization as it was, as it made sharing complicated due to its extreme aspect ratio. I had the lingering idea of changing the representation into a circular version but had put it off for a while as I figured the screencaps would be a bit of a pain but I finally decided try to get it done. 

The color-sound bars were not difficult to change, as it only took changing into polar coordinates and converting the x coordinates into degrees and the y coordinates into radial heights.

The part that was slightly trickier was to have the screencaps assemble around this new representation. Getting them in the right positions was not different from the original bars at set angle intervals and a fixed radius but this time an additional rotation was needed to make the images turn along with the plot. For this, the [cv2 library ](https://pypi.org/project/opencv-python/) comes to the rescue again with the warpAffine function which rotates the numpy array. It took a bit of playing around to figure out the right angles but it worked fine, although after some inspection I realized it was creating a white square background around the original image, so it needed a slight modification. On import, I needed to add an additional step to convert the loaded image from `RGB` to `RGBA` and then use “fill” the square border with a transparent color.

```python
THETA = np.linspace(0, 2*np.pi, sndFrames.shape[0])
fig = plt.figure(figsize=(12, 12))
ax = fig.add_subplot(111, projection='polar')
ax.set_theta_direction(-1)
ax.set_theta_zero_location('S')
offCounter = 0
for (ix, _) in enumerate(sndFrames):
    ax.plot(
        [THETA[ix], THETA[ix]], 
        [RADIUS, RADIUS+sndFrames[ix]*15], 
        color=hexList[ix][0], linewidth=LW*.9,
        solid_capstyle='round', zorder=1
    )
    # Plot image --------------------------------------------------------------
    if ((SFRAME+ix)%DFRAMES==0) and (ix>=0):
        img = np.rot90(
            image.imread(filepaths[ix], cv2.IMREAD_UNCHANGED), 
            k=ROTATION, axes=(1, 0),
        )
        img = cv2.cvtColor(img, cv2.COLOR_RGB2RGBA)
        if ix<(sndFrames.shape[0]/2):
            img = aux.rotate(img, 270+math.degrees(THETA[ix]))
        else:
            img = aux.rotate(img, 90+math.degrees(THETA[ix]))
        imagebox = OffsetImage(img, zoom=ZOOM)
        off = OFFSETS[::][offCounter%len(OFFSETS)]   
        ab = AnnotationBbox(
            imagebox, (THETA[ix], off), frameon=False,
            box_alignment=(0.5, 0.5), 
        )
        ax.add_artist(ab)
        offCounter = offCounter + 1
        # Add callout line ----------------------------------------------------
        ax.plot(
            [THETA[ix], THETA[ix]], 
            [RADIUS, off], 
            lw=CW, color=hexList[ix][0], 
            solid_capstyle='round', ls=':', zorder=1
        )
    ax.text(
        0, 0, TITLE,
        ha='center', va='center',
        # color=MCOLOR,
        color='#22222255',
        fontsize=40,
        fontfamily='Phosphate'
    )
ax.set_axis_off()   
```

<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/polar/Nimona_R.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/polar/Aladdin_R.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/polar/SuperMario_R.png" style="width:100%;"></div>
  </div>
  <!-- <div class="swiper__pagination"></div> -->
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
  <!-- <div class="swiper-scrollbar"></div> -->
</div>