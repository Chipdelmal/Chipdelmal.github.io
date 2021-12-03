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