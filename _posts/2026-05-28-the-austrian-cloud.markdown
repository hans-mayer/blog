---
layout: post
title:  The Austrian Cloud
date:   2026-05-28 13:32:00 CET
categories: fake
---


Tap on the image to see details.

<div class="image-switcher">
  <input type="checkbox" id="cloud-switch" class="switch-input">
  <label for="cloud-switch" class="switch-label">
    <img src="{{ '/images/austriacloud2026.png' | relative_url }}" class="img-gif" alt="Austria Cloud 2026">
    <img src="{{ '/images/viennacloud2026.jpg' | relative_url }}" class="img-static" alt="Vienna Cloud 2026">
  </label>
</div>

<style>
/* Versteckt die eigentliche Checkbox */
.image-switcher .switch-input {
  display: none;
}

/* Container-Styling */
.image-switcher .switch-label {
  position: relative;
  display: inline-block;
  cursor: pointer;
  max-width: 100%;
}

/* Grundeinstellungen für beide Bilder */
.image-switcher img {
  display: block;
  max-width: 100%;
  height: auto;
}

/* Das Zielbild (Vienna) wird exakt über das Startbild gelegt und unsichtbar gemacht */
.image-switcher .img-static {
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0;
  transition: opacity 2s ease-in-out; /* Hier sind die gewünschten 2 Sekunden eingestellt */
}

/* Das Startbild (Austria) bekommt ebenfalls die 2-Sekunden-Transition */
.image-switcher .img-gif {
  transition: opacity 2s ease-in-out;
}

/* Wenn geklickt wurde: Austria blendet aus, Vienna blendet ein */
.image-switcher .switch-input:checked ~ .switch-label .img-gif {
  opacity: 0;
}

.image-switcher .switch-input:checked ~ .switch-label .img-static {
  opacity: 1;
}
</style>

Aufgenommen in der Friedensstraße,  Wien Mauer <br>
Im Zentrum unten die Sternbauten von Atzgersdorf, rechts die Tilgnergasse
<br> 
<br> 

![kreuz und quer](/images/kreuzundquer2026.jpg)

"Kreuz und Quer" Nicht alle Bilder sind mit KI manipuliert. <br>
Aufgenommen in der Taglieberstraße 



