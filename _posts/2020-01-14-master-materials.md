---
layout: post
title: "UE Master Material"
date: 2020-01-14
permalink: /ue-master-material/
author: Nate Maxwell
categories:
    - "Unreal"
---

As of the past few weeks I've been going through my preferred material library
and have been going through my master materials for water, glass, eyes,
foliage, car paint, etc. Recently at work we have been doing a lot of trailer
work for clients and I love when they send over unreal projects for us to work
with. Its always interesting seeing production grade master materials from
other studios. One that I saw recently made extensive use of detail masks and I
thought I would give a try at something similar.

Herein is a tour of a pretty packed mask-based master material for general use.


<img src="https://i.imgur.com/dnFg1Qn.png">

So as previously stated, the primary workflow with this material is using masks
for detail textures over a primary set of Base Color, ORM, and Normal maps.

Here is the standard map controller - It handles the base trifecta maps, UVs,
and the base detail mask texture.

<img src="https://i.imgur.com/K1W3IqU.png">

Here is a look at the UV controller - The top section is for primary map UVs
and the bottom section is basic X + Y tiling for all the detail maps along with
some world aligned normal controls.

<img src="https://i.imgur.com/FOrGzh4.png">

Here is where the UVs are hooked up to each detail texture. The detail maps are
all ORM (occlusion, roughness, metallic) channel packed maps. In this section
they are all controlled by their respective UV outputs and broken into each
individual map. Below is the same for the detail normal maps.

<img src="https://i.imgur.com/yUBSBwK.png">

To see how these are applied we will take a look at the AO section. The top box
is the primary AO map and below is a row for each detail AO map. If you recall
from the standard map controller, there is a red + blue + green texture with
non-overlapping sections. Each color is masked out and then multiple or lerped
by the corresponding mask channel. Additionally, there is some normal aligned
controls.

By using switches we control the number of instructions loaded in the instance
based on the channels used in the detail map. In a full production setting we
would ideally track which channels are used so that the ingest tool can switch
on the necessary channels.

<img src="https://i.imgur.com/3jinwnd.png">

This basic setup is the repeated for roughness, metallic, and normals.
Sometimes the detail maps are multiplied against each other while other times
they are simply added. The instruction depends on the type of greyscale data
that is being used.

<img src="https://i.imgur.com/lPzOFan.png">

That is the primary workflow section accounted for. In addition to all the
detail masking I've also added some various utilities. For example, here is the
emissive controller:

<img src="https://i.imgur.com/Za7YDj8.png">

Along with the normal aligned mask setup, non-normal aligned masking, and some
subsurface controls:

<img src="https://i.imgur.com/jy9grB0.png">

Here are some grass wind and world-space gradient helpers:

<img src="https://i.imgur.com/dnIABl6.png">

And then finally some basic variation and basecolor controls:

<img src="https://i.imgur.com/r1dJ69q.png">

And there you have it. This material has been serving me well as a starter to
get basic environments up and running before more bespoke functionality is
required. I've only used it on a handful of hero environments as most of the
projects I've gotten to use it on up to this point have been very fast turn
arounds (like a week or two).
