---
layout: post
title: "[UE] Pixel Crunch Post Process"
date: 2019-07-24
permalink: /ue-pixel-crunch-post-process/
author: Nate Maxwell
categories:
    - "Unreal"
tags:
    - "unreal"
    - "shader"
    - "material"
    - "uv"
---

# Anemoia

## Noun
    1. Nostalgia for a time or a place one has never known.

I grew up in the 90s but was too young to understand gaming culture or the
games industry at large. In high school I loved reading about the solutions to
hardware limitations plaguing software development since the personal computer
revolution at the tail of the 70s. Among many books on the subject was the now
very famous David Kushner book [Masters of Doom](https://www.amazon.com/Masters-Doom-Created-Transformed-Culture/dp/0812972155/ref=sr_1_1?crid=3QE03KQGDQML6&dib=eyJ2IjoiMSJ9.w6g7hd0-kCe6oZR6ldX0wYv9yIObRa4ZuBrG4Cvy3k_rJCPGGjeOB9FQAZPXJUmIAl5vyh-s2sJN9KNCHfXYm0YhKg4SYiY_fWFUkgaNa-lqCYaQizyYV3bq53Xx-QGJ-vNL8eqDbW5GZhAUUW-z0jH3ajnl5VO19wZvXQ1Rn4IJwvJ7Q7isVVtnpYwEnU0-NowLdiKz86TcLZcQbazyB9eaj2RIca6LEjKOD-F8JGg.Q23NKO2ILnBbvtQkU8w9VUlB13z0t4Nk8c8nmOyNp7o&dib_tag=se&keywords=masters+of+doom&qid=1755262587&sprefix=masters+of+do%2Caps%2C227&sr=8-1).
The early days of Id Software and the various innovators in games from the 90s
were a constant fascination, but this also led to my love of the retro game art
style.

To me, there are 4 key elements that make the retro art style:
* Low resolution
* Cheaper filtering methods (like the trilinear of the n64 or the bilinear of the PS1)
* Vertex wobbling
* Short distance - hard edged - pre-baked lighting

And then occasionally the distance occluding fog that was a common fix for long
render distances.

The first of which, low resolution, is actually trivial to do in unreal. The
most common method is the MFD, or multiply + floor + divide method. If you
multiply your UV space by a number, then floor (or ceil, anything that removes
the floating points), then divide by the number previously multiplied by, you
end up with a pixel crunched UV.

Here you can see in the first quadrant normal 0 to 1 UV values. If you were to
mask x or y in this space it would be a linear gradient from 0 to 1 representing
all floating point values between. In quadrant two you can see that the UV
space has been multiplied by 10, stretching it from 0 to 10, and stretching all
floating point values between. If we remove the floating point values, in the
third quadrant's case using floor, we're left with whole numbers for that axis.
If we divide by the previous number we fit all of these values back into the 0 
to 1 space, eliminating any floating point UV values.

<img src="https://i.imgur.com/mM88rGZ.png">

Here you can see the effect applied to some noise texture. I've multiplied and
divided by a value of 10, which gives us 10 pixels. The number chosen will
equal the resolution, so to speak.

<img src="https://i.imgur.com/RlOEbkX.png">

From here we can apply this effect to a post process material by applying the
calculation to some SceneTexture UVs and piping out the color like so:

<img src="https://i.imgur.com/O3NwOIx.png">

When applied to an unbound volume, we have effectively crunched the resolution
of the entire scene.

<img src="https://i.imgur.com/zES9kt5.png">

Ideally, in a full retro graphics game, the other 3 points are also applied. It
seems today that too many developers stop here and assume they've achieved the
retro look. If you're trying to capture the N64 nostalgia, trilinear filtering
is crucial as textures on that platform were often desaturated and blurrier
than its PS1 counterpart. Meanwhile, the PS1 was a lower bit system and therefore
performed some agressive rounding for vertex position resulting in that "wobbly"
look on the models.
