---
title: 'Last.fm Clocks'
tags: lastfm music datasci python
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

<img style="width: 100%" src="/media/clocks/clocks.png"/>

# Gallery

<style>
  .swiper-demo {height: 900px;}
  .swiper-demo .swiper__slide {
    display: flex; align-items: center; justify-content: center;
    font-size: 3rem; color: #fff;
  }
</style>


<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2012_01-2013_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2013_01-2014_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2014_01-2015_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2015_01-2016_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2016_01-2017_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2017_01-2018_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2018_01-2019_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2019_01-2020_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2020_01-2021_01.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/clocks/PLC_HRL-2021_01-2022_01.png" style="width:100%;"></div>
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


# Code repo

# Documentation and Code

* **Repository:** [Github repo](https://github.com/Chipdelmal/LastfmViz)
