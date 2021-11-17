---
title: "Movies' Dominant Colors"
tags: clustering artsci color movie video-processing image-processing
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/dc/1992_Aladdin.png
cover: /media/dc/1992_Aladdin.jpg
---


After coming across some posts online on the clustering of dominant colors in movies presented in a very appealing way, I decided to improve upon my original [movies fingerprint algorithm](https://chipdelmal.github.io/blog/posts/colorfingerprint).


<!--more-->

# Description

The first thing I realized whilst looking at other persons' implementations was that they tended to focus on one color on each of the sampled movie frames, which simplified the presentation of the results. The second thing, that they presented the results in a more appealing way than I originally did.

To remedy the second point, I settled on doing square representations of the colors with a title overlay. In these representation, each vertical bar stands for a sampled frame in the movie:

<img src="/media/dc/dc_NausicaaOR.png" style="width:100%;">


Now, on the trickier part, I looked at my code and realized that it worked fine with the case of one cluster for the whole frame. This, however, has one important drawback: as it assigns every pixel to the same cluster and then returns the centroid, so it pretty much calculates the average color of the image (as in the example shown above). This is not a problem for cases in which movies are very skewed in color-space, and is, in fact, the way many of the algorithms found online work; but, I wanted this algorithm to be more general and to be able to detect several clusters even if it only returned a subset. For this extension, a couple of changes were needed: we needed to calculate and detect the required clusters, and then return a subset.

As we can clearly see in the image below, the resulting colors were far more bright, as compared to the original that looked "washed away" due to the centroid being "pulled" from the detected color towards the other colors present in the frame.

<img src="/media/dc/dc_Nausicaa.png" style="width:100%;">

For bonus points on the scripts, I decided to parallelize the processing of the frames (as I was a bit tired of waiting for scripts to run). Doing this was a bit trickier than expected, as it required the use of a [*memmap*](https://numpy.org/doc/stable/reference/generated/numpy.memmap.html) to be able to modify a common numpy array from different threads.


<img src="/media/dc/dc_CastleInTheSky.png" style="width:100%;">

Finally, I coded a [bash script](https://github.com/Chipdelmal/moviesColorFingerprint/blob/master/fingerprint.sh) to automate the whole process of re-scaling, extracting frames, and exporting the fingerprint.

<img src="/media/dc/dc_KikisDeliveryService.png" style="width: 100%;">



# Documentation and Code

* **Repository:** [Github repo](https://github.com/Chipdelmal/moviesColorFingerprint)
* **Dependencies:** [opencv-python](https://pypi.org/project/opencv-python/), [ffmpeg-python](https://pypi.org/project/ffmpeg-python/), [Pillow](https://pillow.readthedocs.io/en/stable/), [numpy](https://numpy.org/), [scikit-learn](https://scikit-learn.org/stable/), [matplotlib](https://matplotlib.org/), [ffmpeg](https://www.ffmpeg.org/)
