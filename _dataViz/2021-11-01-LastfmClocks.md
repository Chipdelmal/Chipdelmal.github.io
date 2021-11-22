---
title: 'Last.fm Clocks'
tags: lastfm music datasci python polar matplotlib
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/clocks/clocks.png
cover: /media/clocks/clocks.jpg
---


<!--more-->


# Intro


# Development

## Loading data

```python
(WIDTH, HEIGHT, RESOLUTION) = (3840, 2160, 500)
HOURS_OFFSET = 6
##############################################################################
# Read artists file
##############################################################################
data = pd.read_csv(
    stp.DATA_PATH + stp.USR + '_cln.csv',
    parse_dates=[3]
)
data = data.drop_duplicates()
```


## Filtering dates

```python
data = data.drop_duplicates()
msk = [
    (
        (i.date() >= datetime.date(yLo[0], yLo[1], 1)) and 
        (i.date() < datetime.date(yHi[0], yHi[1], 1))
    ) 
    if (type(i) is not float) else (False) for i in data['Date']
]
dates = data.loc[msk]["Date"]
```

## Binning Frequencies

```python
hoursPlays = sorted([i.hour for i in dates if (type(i) is not float)], reverse=True)
hoursFreq = [len(list(group)) for key, group in groupby(hoursPlays)]
hoursFreq = [hoursPlays.count(hD) for hD in list(range(23, -1, -1))]
```

## Polar Plot

```python
N = 24
(minFreq, maxFreq) = (min(hoursFreq), max(hoursFreq))
fig = figure(figsize=(8, 8), dpi=RESOLUTION)
ax = fig.add_axes([0.2, 0.1, 0.8, 0.8], polar=True)
step=  2*np.pi/N
(theta, radii, width) = (
    np.arange(0.0+step, 2*np.pi+step, step),
    hoursFreq,
    2*np.pi/24 -.001
)
bars = ax.bar(
    theta, radii, width=width, 
    bottom=0.0, zorder=25, edgecolor='#ffffff77', lw=.75
)
rvb = aux.colorPaletteFromHexList(
    ['#bbdefb', '#64b5f6', '#2196f3', '#1976d2', '#0d47a1', '#001d5d']
)
for r, bar in zip(radii, bars):
    bar.set_facecolor(rvb(r/(np.max(hoursFreq)*1)))
    # bar.set_facecolor(cm.BuPu(r/(np.max(hoursFreq)*1.1)))
    bar.set_alpha(0.75)
```

<center><img style="width: 50%" src="/media/clocks/output.png"/></center>


## Axes Styling

```python
shades = 12
step =  np.pi/shades
ax.bar(
    np.arange(0.0+step, 2*np.pi+step, step), 
    1.15*maxFreq,
    width=np.pi/shades,
    alpha=.2, edgecolor="black", ls='-', lw=.5,
    zorder=-1
)
ax.set_theta_zero_location("N")
```

<center><img style="width: 50%" src="/media/clocks/PLC_HRL-2021_01-2022_01.png"/></center>




# Gallery

<!--
<style>
  .swiper-demo {height: 450px;}
  .swiper-demo .swiper__slide {
    display: flex; align-items: center; justify-content: center;
    font-size: 3rem; color: #fff;
  }
</style>


<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2012_01-2013_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2013_01-2014_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2014_01-2015_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2015_01-2016_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2016_01-2017_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2017_01-2018_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2018_01-2019_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2019_01-2020_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2020_01-2021_01.png" style="width:50%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2021_01-2022_01.png" style="width:50%;"></div>
  </div>
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
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
-->


<img style="width: 100%" src="/media/clocks/clocks.png"/>

# Code repo

* **Repository:** [Github repo](https://github.com/Chipdelmal/LastfmViz/blob/master/PlayByHour_polar.py)
