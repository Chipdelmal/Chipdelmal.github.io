---
title: "Robotics and Computer Vision"
tags: robotics computer-vision labview mathematica dynamics
article_header:
  type: overlay
  theme: dark
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 0, 0 , .4), rgba(0, 0, 0, .4))'
    src: /media/robots/labView.png
cover: /media/robots/labView.png
---

<br>

Before shifting to working in Public Health, I did some work in computer vision and humanoid robots as part of a couple of research internships.

<!--more-->

# Intro

My first approach to scientific research was under [Alejandro Aceves-Lopez'](https://www.researchgate.net/profile/Alejandro-Aceves-Lopez) supervision in determining dynamic chains in humanoid robot's limbs, which was followed by an internship at the [Mexican Institute for Nuclear Research](http://www.inin.gob.mx/index.cfm) in computer vision applications in mobile robots control.

![](/media/robots/bogo.png)

# Projects

These two projects resulted in technical reports that can be accessed here: [Humanoid Robots](https://www.researchgate.net/publication/281294678_Simulacion_Dinamica_de_una_Cadena_Cinematica_Abierta_a_partir_del_Modelo_Euler-LaGrange_y_Considerando_el_Modelo_Dinamico_de_los_Servo-motores_para_su_Aplicacion_de_Robots_Humanoides), and [Computer Vision](https://www.researchgate.net/publication/262449532_Computer_Vision_Via_Weighting_Algorithm_and_an_Artificial_Target_for_Location_with_a_Vehicle). A brief description will be provided in this website.


## Humanoid Robot Dynamic Chains

This project's main goal was to determine the dynamic chains of a humanoid robot's actuator to enhance its precision in movement and manipulation of objects. The first step to do this was to calculate the Voltage-Position response curves for the servo-motors. 
With this in hand, we were able to determine the Torque-Voltage transfer functions for the servo-motors' responses.

<center><img src="/media/robots/servo.png" style="width:50%;"></center>

The Voltage-Torque equations were then combined with the inertia equations of the manipulators to get the relations for all the components of the system. Once we got all the bits together, we were able to put together the dynamics equations taking into account the manipulators' and servo-motors influences:

![](/media/robots/eq.png)

## Computer Vision for Target Locations

This project had as an objective the detection of a target using computer vision to control a mobile robot in conditions of low to medium visual noise. The general idea was to make a mobile robot turn to face the direction of the target whenever it was in sight.

The approach we took to solve the problem was to use four different identification techniques, namely: color segmentation, color segmentation with ellipse detection, edge ellipse detection, and pattern matching. All these techniques have their advantages and disadvantages in terms of their reliability in detecting objectives, so we decided to come up with a way to combine them in a meaningful way that allowed us to have more certainty about the target's position. To do this, we used the following variables:

* Detected something
* Detected multiple targets
* Position "jumped" from the previous frame to this one
* Detection is close in distance to the position other algorithms detected
* Position is close to the one predicted by a squared fit of the previous ten frames of the video
* Score provided by the detection algorithm

With these in hand, we created a behavior dataset that was tagged manually. These data allowed us to understand the correlations between the behavior variables and the response of all the algorithms.

<div class="swiper my-3 swiper-demo swiper-demo--0">
  <div class="swiper__wrapper">
    <div class="swiper__slide"><img src="/media/robots/colorSeg.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/robots/patMat.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/robots/ellipse.png" style="width:100%;"></div>
    <div class="swiper__slide"><img src="/media/robots/edge.png" style="width:100%;"></div>
  </div>
  <!-- <div class="swiper__pagination"></div> -->
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
  <!-- <div class="swiper-scrollbar"></div> -->
</div>

Once that was calibrated, we created a weighted-voting system across algorithms in which the ones that yield the most reliable prediction of the position (according to the behavioral variables), would contribute the most to the movement of the robot.


![](/media/robots/labView.png)

Additionally, we provided a neural network implementation of the voting system, along with the option to add a filter to reduce the noise in the prediction of the position of the target, and the possibility to automatically shut down detection algorithms that were failing constantly in their predictions.

An overview of how the system works can be found in the following [video](https://www.youtube.com/embed/-yCt_6OfRT4).

# Videos and Links

* [Humanoid Robots](https://www.researchgate.net/publication/281294678_Simulacion_Dinamica_de_una_Cadena_Cinematica_Abierta_a_partir_del_Modelo_Euler-LaGrange_y_Considerando_el_Modelo_Dinamico_de_los_Servo-motores_para_su_Aplicacion_de_Robots_Humanoides) report.
* [Computer Vision](https://www.researchgate.net/publication/262449532_Computer_Vision_Via_Weighting_Algorithm_and_an_Artificial_Target_for_Location_with_a_Vehicle) report.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/-yCt_6OfRT4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>





<script>
  {%- include scripts/lib/swiper.js -%}
  var SOURCES = window.TEXT_VARIABLES.sources;
  window.Lazyload.js(SOURCES.jquery, function() {
    $('.swiper-demo--0').swiper(); $('.swiper-demo--1').swiper();
    $('.swiper-demo--2').swiper(); $('.swiper-demo--3').swiper();
    $('.swiper-demo--4').swiper({ animation: false });
  });
</script>