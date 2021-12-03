---
title: 'Last.fm Artists Playcounts'
tags: lastfm artsci music artsci python wordcloud matplotlib
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/artists/ART_WDC-2021_2022.jpg
cover: /media/artists/ART_WDC-2021_2022Lo.jpg
---

Coming soon!

<!--more-->

# Intro

In latest years, Spotify has released features for users to get a summary of the artists they have listened the most to. I don't use Spotify too much, but I do keep track of my music-listening habits through last.fm (as I've described in my previous posts: [Last.fm Visualization](/artsci/2019-12-10-LastfmViz.html) and  [Last.fm Clocks](/artsci/2021-11-01-LastfmClocks.html)). Not wanting to be left behind, I put something together to show my playcounts in the form of wordclouds.

# CodeDev

## Downloading and cleaning

To download a CSV file of my "scrobbles" I've been using [website](https://benjaminbenben.com/lastfm-to-csv/), which takes in a username and retreives the scrobbles summary in tableform. As I've described before, I've already coded a [script]() that cleans the CSV dataset, and another [script]() that parses the artists' data from [MusicBrainz](https://musicbrainz.org/); so, the first step is running these two pieces of code upon the CSV file.

## Counting artists' playcounts

Now, the first step is to do the counting.

```python
data = pd.read_csv(stp.DATA_PATH + stp.USR + '_cln.csv', parse_dates=[3])
data = data.drop_duplicates()
artists = sorted(data.get('Artist').unique())
artistCount = data.groupby('Artist').size().sort_values(ascending=False)
```

## Creating cmap

I wanted to have control over the color mapping, so I used a custom function that I've used for other projects. This function takes a list of hex colors and returns a cmap object that interpolates between them:

```python

```

With this function in place, the color palette used was:

```python

```

It is worth noting that the white color is repeated to make the transitions between colors "sharper".
## Generating wordcloud



```python
wordcloudDef = WordCloud(
        width=WIDTH, height=HEIGHT, max_words=2000,
        relative_scaling=.5, min_font_size=5, font_path=stp.FONT,
        background_color='rgba(0, 0, 0, 1)', mode='RGBA',
        colormap='BuPu' # stp.cMap
    )
wordcloud = wordcloudDef.generate_from_frequencies(artistCount)
```

## Plotting 

We now create our figure object:

```python

```

Doing a solid black background was not very appealing, so I loaded a custom texture for background:

```python

```

With this in place, we can plot our wordcloud in our canvas:

```python
plt.imshow(wordcloud, interpolation='bilinear')

```

Now, for the final touches and export:

```python
plt.tight_layout(pad=0)
plt.axis("off")
plt.savefig(
  stp.IMG_PATH + '/ART_WDC.png',
  dpi=RESOLUTION, facecolor='Black', edgecolor='w',
  orientation='portrait', papertype=None, format=None,
  transparent=True, bbox_inches='tight', pad_inches=.1,
  metadata=None
)
```


# Gallery

<style>
  .swiper-demo {height: 850px;}
  .swiper-demo .swiper__slide {
    display: flex; align-items: center; justify-content: center;
    font-size: 3rem; color: #fff;
  }
</style>


<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2021_2022.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2020_2021.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2019_2020.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2018_2019.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2017_2018.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2016_2017.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2015_2016.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2014_2015.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2013_2014.jpg" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/artists/ART_WDC-2012_2013.jpg" style="width:100%;"></div>
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

# Code repo

* **Repository:** [Github repo](https://github.com/Chipdelmal/LastfmViz)
* **Dependencies:** [pandas](https://pandas.pydata.org/),  [musicbrainzngs](https://github.com/alastair/python-musicbrainzngs), [geopy](https://geopy.readthedocs.io/), [basemap](https://matplotlib.org/basemap/), [matplotlib](https://matplotlib.org/), [wordcloud](https://github.com/amueller/word_cloud), [opencv-python](https://pypi.org/project/opencv-python/)
