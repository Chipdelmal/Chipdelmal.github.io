---
title: 'Color Palette Extraction'
tags: clustering artsci color movie video-processing image-processing ffmpeg open-cv
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/cp/cp_yakul.jpg
cover: /media/cp/cp_yakul.jpg
---

<!--more-->

# Intro

Browsing around online, I came across a movie still which had the dominant colors lined up at the bottom of the frame and started to think of a way to automate it using clustering techniques. After some research and brainstorming, I figured it wouldn't be difficult to do using [scikit-learn](https://scikit-learn.org/stable/) with [Pillow](https://pillow.readthedocs.io/en/stable/), and came up with a couple of ideas to use it to generate fancy desktop backgrounds.


<img src="/media/cp/Princess.jpg" style="width:100%;">

# Development

## Step 1: Loading image

The first thing was to load images to Python. I decided to use [opencv-python](https://pypi.org/project/opencv-python/) due to its extensive documentation and support. One important thing was to notice that, by default, the image was not imported in RGB format, so it has to be converted (which can be done easily with opencv itself).

<img src="/media/cp/cp_spirited.jpg" style="width:100%;">


{% highlight python %}
# File: aux.py
bgr = cv2.imread(imgPath)
img = cv2.cvtColor(bgr, cv2.COLOR_BGR2RGB)
{% endhighlight %}

## Step 2: Reshape & clustering

After this, we do a slight reshaping to get the image in an RGB array of integers (8 bit). Now, if we think about the RGB points in 3D space, each point exists in the intersection of three (R,G,B) coordinates, so we can use unsupervised clustering techniques to get the dominant colors by obtaining the centroids of such agglomerations of colors. Although there are many clustering techniques that would get the job done in this situation, I chose to use K-Means because of its simplicity and the fact that we can provide it with the number of clusters we want as the output.

{% highlight python %}
# File: aux.py
def calcDominantColors(img, cltsNumb=10, maxIter=1000):
    frame = img.reshape((img.shape[0] * img.shape[1], 3))
    # Cluster the colors for dominance detection
    kmeans = KMeans(n_clusters=cltsNumb, max_iter=maxIter).fit(frame)
    (palette, labels) = (kmeans.cluster_centers_, kmeans.labels_)
    # Rescale the colors for matplotlib
    colors = [reshapeColor(color) for color in palette]
    return (colors, labels)
{% endhighlight %}

## Step 3: Dominant colors

With this function in place, we are now able to calculate the dominant colors of the image, and then transform the resulting [matplotlib](https://matplotlib.org/) colors into *HEX* and *8 bit RBG* palettes:

{% highlight python %}
# File: aux.py
(colors, labels) = calcDominantColors(
        img, cltsNumb=clstNum, maxIter=maxIters
    )
(hexColors, rgbColors) = calcHexAndRGBFromPalette(colors)
{% endhighlight %}

<img src="/media/cp/cp_mononoke.jpg" style="width:100%;">

## Step 4: Assemble image

Finally, we assemble together the frame with the color palette on the top and bottom for aesthetic purposes:

{% highlight python %}
# File: aux.py
# Create color swatch
colorsBars = genColorSwatch(img, colorBarHeight, colors)
# Put the image back together
whiteBar = genColorBar(
        width, round(height * bufferHeight), color=colorBuffer
    )
newImg = np.row_stack((
        whiteBar, colorsBars,
        img,
        colorsBars, whiteBar
    ))
palette = calcHexAndRGBFromPalette(colors)
swatch = Image.fromarray(colorsBars.astype('uint8'), 'RGB')
imgOut = Image.fromarray(newImg.astype('uint8'), 'RGB')
return (imgOut, swatch, palette)
{% endhighlight %}

The fully-documented code is available in the [Github repo](https://github.com/Chipdelmal/colorPaletteExtractor) with clearly-defined functions and notes, with ruotines for [single images](https://github.com/Chipdelmal/colorPaletteExtractor/blob/master/mainSingle.py), and [batch export](https://github.com/Chipdelmal/colorPaletteExtractor/blob/master/mainBatch.py) of frames.

<img src="/media/cp/cp_tonight.jpg" style="width:100%;">

# Further Thoughts

One final thing I did with this code, was to take the [batch routine](https://github.com/Chipdelmal/colorPaletteExtractor/blob/master/mainBatch.py) and use it to get the color palette of the frames of movies, and then to re-assemble them into grids for their use as fancy wallpapers.

<img src="/media/cp/Nausicaa.jpg" style="width:100%;">

# Documentation and Code

* Repository: [Github repo](https://github.com/Chipdelmal/colorPaletteExtractor)
* Dependencies: [opencv-python](https://pypi.org/project/opencv-python/), [ffmpeg-python](https://pypi.org/project/ffmpeg-python/), [Pillow](https://pillow.readthedocs.io/en/stable/), [numpy](https://numpy.org/), [scikit-learn](https://scikit-learn.org/stable/), [matplotlib](https://matplotlib.org/)
