---
title: Style Transfer "Pop Art"
tags: machine-learning artsci mathematica pop-art mathematica artstyle-transfer
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/pp/pp_fender.jpg
cover: /media/pp/pp_fender.jpg
---


<!--more-->

# Intro

Some time ago I ran across a [paper](https://arxiv.org/pdf/1508.06576.pdf) describing the machine-learning application of transferring the art-style from a painting into another image. [Mathematica v12](https://www.wolfram.com/language/12/machine-learning-for-images/style-transfer-for-creative-art.html?product=mathematica) includes a built-in command to do this in a single line of code, so I started using it to explore ideas until I had something I liked, which ended up being a "Pop-Art" Machine Learning application for photos.

<img src="/media/pp/pp_semi.jpg" style="width:100%;">

# Development

The idea was to take a photo, and then apply different artstyles or textures borrowed from other images onto it, so that we could then take slices from it and arrange them side by side. As an example, we start with this image:

<img src="/media/pp/pp_fenderIn.jpg" style="width:100%;">


## Step 1: Load and Rescale

Which is loaded by the following code:

{% highlight mathematica %}
(* File: main.nb *)
{imagesSize, styleWeight, goal, overlayWeight} = {1500, .8, "Quality", .25};
(*Load images*)
SetDirectory[NotebookDirectory[]];
input = Import["./in.jpg"];
imgOriginal = ImageResize[input // ImageAdjust, imagesSize] // ColorConvert[#, "Grayscale"] &;

{% endhighlight %}

Similarly, we load all the artsyles we want transferred:

{% highlight mathematica %}
(* File: main.nb *)
styleFilenames = FileNames[{"*.jpg", "*.png"}, "./styles"];
stylesRaw = Import[#] & /@ styleFilenames;
styles = ImageResize[#, imagesSize] & /@ stylesRaw;

{% endhighlight %}

<img src="/media/pp/pp_styles.jpg" style="width:100%;">

## Step 2: Artstyle Transfer

Then, we apply the art-style transfer process (takes a **really** long time):

{% highlight mathematica %}
(* File: main.nb *)
Table[
    (*Transfer*)
    img = ImageRestyle[imgOriginal
        ,{styleWeight -> styles[[i]]}
        ,PerformanceGoal -> goal
    ];
    (*Export*)
    Export[("./batch/pop" <> StringPadLeft[ToString[i], 3, "0"] <>".jpg")
        ,img
        ,ImageSize -> 2*imagesSize
    ];
    (*Memory clear*)
    ClearSystemCache[]; Clear[img];
,{i, 1, Length[styles]}];
{% endhighlight %}

<img src="/media/pp/pp_grid.jpg" style="width:100%;">

## Step 3: Assemble the pieces

Finally, we put it back together by taking random slices from each of the style-transferred results:

{% highlight mathematica %}
(* File: main.nb *)
filenames = FileNames[{"pop*.jpg", "pop*.png"}, "./finals/"];
imgs = RandomSample[ImageResize[Import[#], imagesSize] & /@ filenames];
{width, height} = ImageDimensions[imgOriginal];
{deltaV, deltaH} = {Round[width/Length[imgs]], Round[height/Length[imgs]]}

slices = Table[
      probe = imgs[[i + 1]];
      ImageTake[probe, {0, height}, {deltaV*i, deltaV*i + deltaV}]
, {i, 0, (imgs // Length) - 1}];
merged = ImageCompose[ImageAssemble[slices], {imgOriginal, overlayWeight}] // Sharpen[#, sharpen] &;
Export["./art/vertical.jpg", merged // ImageAdjust // Sharpen[#, sharpen] &, ImageSize -> imagesSize]
{% endhighlight %}

And we have our assembled image:

<img src="/media/pp/pp_fender.jpg" style="width:100%;">


# Gallery

<style>
  .swiper-demo {height: 450px;}
  .swiper-demo .swiper__slide {
    display: flex; align-items: center; justify-content: center;
    font-size: 3rem; color: #fff;
  }
</style>


<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/pp/f_verticalacoustic.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/pp/f_verticalcustom.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/pp/f_verticalsemi.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/pp/pp_fender.jpg" style="width:100%;"></div>
  </div>
  <!-- <div class="swiper__pagination"></div> -->
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
  <!-- <div class="swiper-scrollbar"></div> -->
</div>

<script>
  {%- include scripts/lib/swiper.js -%}
  var SOURCES = window.TEXT_VARIABLES.sources;
  window.Lazyload.js(SOURCES.jquery, function() {
    $('.swiper-demo--0').swiper(); $('.swiper-demo--1').swiper();
    $('.swiper-demo--2').swiper(); $('.swiper-demo--3').swiper();
    $('.swiper-demo--4').swiper({ animation: false });
  });
</script>


# Code Repo

As it stands, the process is extremely inefficient in terms of computation. We don't really need to process the whole image if we are gonna take only one slice at the end. It would also work faster if it was coded in Python with [available packages](https://pypi.org/project/neural-style/).

* Repository: [Github repo](https://github.com/Chipdelmal/styleTransferPopArt)
* Dependencies: [Mathematica v12](https://www.wolfram.com/language/12/machine-learning-for-images/style-transfer-for-creative-art.html?product=mathematica)

