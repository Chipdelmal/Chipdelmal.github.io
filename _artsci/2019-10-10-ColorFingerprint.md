---
title: 'Movie Color Fingerprint'
tags: clustering artsci color movie video-processing image-processing
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/cf/back.png
cover: /media/cf/back.png
---


This was the original application of the dominant color detection described in my ["Color Palette Extractor" blog post](https://chipdelmal.github.io/blog/posts/colorpalette). These scripts take a video (a movie, ideally), and extract the dominant colors every *n* frames of the movie, to create a "fingerprint" of the colors used throughout the movie.


<!--more-->

<img src="https://chipdelmal.github.io/blog/uploads/cf_nausicaa.jpg" style="width:100%;">

# Development

The first step was to pre-process the movie to lower the resolution to a more manageable size (mainly to speeup the processing time of the clustering). Even though resizing could potentially have effects on the colors due to compression, we aren't downsizing the video too much. Here's a [snippet of the code](https://github.com/Chipdelmal/moviesColorFingerprint/blob/master/rescaleMovie.py) that does this downsampling by using [ffmpeg](https://www.ffmpeg.org/):

{% highlight python %}
# File: rescaleMovie.py
import os
# Inputs
(FILE_NAME, DIMS) = ('Pilot.mp4', (640, 320))
(IN_PATH, OUT_PATH) = ('./original/', './rescaled/')
# Call to terminal to run ffmpeg to rescale the video
os.system(
        "ffmpeg -loglevel panic "
        + "-i " + IN_PATH + FILE_NAME + " "
        + "-vf scale=" + str(DIMS[0]) + ":" + str(DIMS[1]) + " "
        + OUT_PATH + FILE_NAME
    )
{% endhighlight %}

Now, to do the clustering to identify the dominant color of the images, there's two approaches we can take:
1. Calculate the dominant colors "on-the-fly" by loading the video and going through the frames in memory (more efficient)
2. Export the frames to independent images, and then calculate the clustering on the images in a different script (more flexible)

As this is an exploratory script, which requires various tweaks and incremental improvements, and that I wanted to build upon it for other applications, I chose to go for the second route (although I do provide a [deprecated version](https://github.com/Chipdelmal/moviesColorFingerprint/blob/master/deprecated/onTheFly.py) of the "on-the-fly" analysis for anyone interested).
The [exportFrames](https://github.com/Chipdelmal/moviesColorFingerprint/blob/master/exportFrames.py) file does this by taking a video file, and the number of frames we need exported:

{% highlight python %}
# File: exportFrames.py
import os
import ffmpeg
# Inputs
(FILE, FRAMES_NUM) = ("Pilot.mp4", 100)
(IN_PATH, OUT_PATH) = ("./rescaled/", "./frames/")
# Calculating fps to match required number of frames
probe = ffmpeg.probe(IN_PATH + FILE)
vInfo = next(s for s in probe['streams'] if s['codec_type'] == 'video')
framesNumMovie = int(vInfo['nb_frames'])
framerate = eval(vInfo['avg_frame_rate'])
fps = FRAMES_NUM / framesNumMovie * framerate
# Export frames into images
os.system(
        "ffmpeg -loglevel panic "
        + "-i " + IN_PATH + FILE + " "
        + "-vf fps=" + str(fps) + " "
        + OUT_PATH + FILE.split(".")[0] + "%04d.png "
        + "-hide_banner"
    )
{% endhighlight %}

Which gives us all our files in a folder for further processing in this and other applications (such as ["Color Palette Extractor"](https://chipdelmal.github.io/blog/posts/colorpalette)):

<img src="https://chipdelmal.github.io/blog/uploads/cf_frames.jpg" style="width:100%;">

With our images ready, we go on to calculate the clusters of dominant colors with [fingerprint.py](https://github.com/Chipdelmal/moviesColorFingerprint/blob/master/fingerprint.py):

{% highlight python %}
# File: main.py
import aux
# User Inputs
(FILE, DOMINANT, DPI) = ("00_Pilot", 6, 500)
(IN_PATH, OUT_PATH) = ("./frames/", './fingerprint/')
# Get frames paths and calculate the dominant clusters of the images
filepaths = aux.getFilepaths(IN_PATH, FILE)
clusters = aux.calculateDominantColors(filepaths, DOMINANT)
# Export the resulting fingerprints
aux.exportFingerprintPlot(
        OUT_PATH, FILE + '.png', clusters,
        dims=(10, 5), dpi=DPI
    )
{% endhighlight %}


The function that does all the work is `aux.calculateDominantColors`, so we're gonna break it down:

{% highlight python %}
def calculateDominantColors(filepaths, domColNum, maxIter=100):
  # Create an empty array with dimensions: (framesNumber, dominantColors, 3)
  clusters = np.empty((len(filepaths), domColNum, 3))
  # Initialize a KMean instance for clustering
  kMeansCall = MiniBatchKMeans(n_clusters=domColNum, max_iter=maxIter)
  # Iterate through the files
  for (i, path) in enumerate(filepaths):
      # Read image and reshape to an RGB vector of vectors
      (frame, shp) = readAndProcessImg(path)
      flatFrame = frame.reshape([1, shp[0] * shp[1], 3])[0]
      # Cluster the RGB entries for color dominance detection
      kmeans = kMeansCall.fit(flatFrame)
      # Take the color palette and add it to the clusters container
      palette = kmeans.cluster_centers_
      clusters[i] = [rescaleColor(color) for color in palette]
  return clusters
{% endhighlight %}

Here, we are iterating through the images in the folder by loading them, converting from BGR to RGB ([opencv](https://pypi.org/project/opencv-python/) loads them in BGR format), reshaping the array to remove one of the dimensions of the image (converting a 2D array of RGB entries, into a 1D vector of RGB rows). Finally, we do the clustering of the colors to obtain the dominant color palette (this process is described in more detail in my other blog post ["Color Palette Extractor"](https://chipdelmal.github.io/blog/posts/colorpalette)), and add the result to a vector that will contain all of the palettes of the frames.

<img src="https://chipdelmal.github.io/blog/uploads/cf_spirited.jpg" style="width:100%;">

# Further Thoughts

I'd like to automate this process even further in the near future by creating a wrapper that takes care of all the steps in the process, so stay tuned!

# Documentation and Code

* **Repository:** [Github repo](https://github.com/Chipdelmal/moviesColorFingerprint)
* **Dependencies:** [opencv-python](https://pypi.org/project/opencv-python/), [ffmpeg-python](https://pypi.org/project/ffmpeg-python/), [Pillow](https://pillow.readthedocs.io/en/stable/), [numpy](https://numpy.org/), [scikit-learn](https://scikit-learn.org/stable/), [matplotlib](https://matplotlib.org/), [ffmpeg](https://www.ffmpeg.org/)
