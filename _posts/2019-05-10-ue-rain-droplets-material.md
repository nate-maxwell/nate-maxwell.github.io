---
layout: post
title: "[UE] Rain Droplets Material"
date: 2019-05-10
permalink: /ue-rain-drop-material/
author: Nate Maxwell
categories:
    - "Unreal"
tags:
    - "unreal"
    - "shader"
    - "material"
    - "uv"
---

Lately I have been playing around with the GenerateBand material function and have been animating the coordinates using some noise maps to create rain drop ripple effects.

<p align="center">
<img src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExZGVsdWo4dTY3eWRtbHhqcXZmd3E2bnVyYjN3YnZocDdueXV4bnhsOCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/CSrquC3M44APXkKIPK/giphy.gif">
</p>

Using a blobular noise texture and running it through a panner and then using that result as the coordinates for a GenerateBand node, you get a circular growth pattern like the ripples of a rain drop. By repeating this blob over and over we can “simulate” a bunch of rain drops.

<p align="center">
<img src="https://i.imgur.com/YJrDyay.png">
</p>

By offsetting the panner time with some noise, we get independent droplet movement. Here is the final “core” logic for the shader:

<p align="center">
<img src="https://i.imgur.com/PFCQN4u.png">
</p>

Finally, we can use the HeightToNormalSmooth node, if we didn't want to make and animate a normal map ourselves, and we get some pixel height mimicking an actual splash and surface variation.

The surface (with cranked up normals for easier view):

<p align="center">
<img src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExdXVpdmtoazNodHF5Y2M0eG02MnhpYmpsODdqNG1pY2xpdGp0dzE4MiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/cHmJrFmceplPHBY42t/giphy.gif">
</p>

Normal Calculation:

<p align="center">
<img src="https://i.imgur.com/pCcUdIE.png">
</p>

Here is a picture of the final logic with some various sliders for fine-tuning.

<p align="center">
<img src="https://i.imgur.com/HArLywM.png">
</p>

And the channel packed texture of the rain pattern and some stock photoshop noise for time offset.

<p align="center">
<img src="https://i.imgur.com/8Py9oT2.png">
</p>
