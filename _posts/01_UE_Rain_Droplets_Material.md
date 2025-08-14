---
layout: post
title: "UE Rain Droplets Material"
date: 2025-08-14
permalink: /ue-rain-drop-material/
categories:
    - "Unreal"
---

Lately I have been playing around with the GenerateBand material function and have been animating the coordinates using some noise maps to create rain drop ripple effects.

<img src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExZGVsdWo4dTY3eWRtbHhqcXZmd3E2bnVyYjN3YnZocDdueXV4bnhsOCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/CSrquC3M44APXkKIPK/giphy.gif">

Using a blobular noise texture and running it through a panner and then using that result as the coordinates for a GenerateBand node, you get a circular growth pattern like the ripples of a rain drop. By repeating this blob over and over we can “simulate” a bunch of rain drops.

<img src="https://i.imgur.com/YJrDyay.png">

By offsetting the panner time with some noise, we get independent droplet movement. Here is the final “core” logic for the shader:

<img src="https://i.imgur.com/PFCQN4u.png">

Finally, we can use the HeightToNormalSmooth node, if we didn't want to make and animate a normal map ourselves, and we get some pixel height mimicking an actual splash and surface variation.

The surface:

<img src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExdXVpdmtoazNodHF5Y2M0eG02MnhpYmpsODdqNG1pY2xpdGp0dzE4MiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/cHmJrFmceplPHBY42t/giphy.gif">

Normal Calculation:

<img src="https://i.imgur.com/pCcUdIE.png">


Here is a picture of the final logic with some various sliders for fine-tuning.

<img src="https://i.imgur.com/HArLywM.png">

And the channel packed texture of the rain pattern and some stock photoshop noise for time offset.
<img src="https://i.imgur.com/8Py9oT2.png">
