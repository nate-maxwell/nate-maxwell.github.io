---
layout: post
title: "UE Mesh Merger"
date: 2025-03-21
permalink: /ue-mesh-merger/
author: Nate Maxwell
categories:
    - "Unreal"
---

Many friends of mine are working on small indie games with 2 to 8-man teams.
Frequently these teams engage in the discussion of optimization efficiency.
Specifically they weigh the resources they have, time, skill sets, amount of
work to convert, etc., and how much optimization they want to conduct. A common
problem is that they have a small art "department", often an artist or two, and
therefore rely on third party asset packs from various marketplaces.
Unfortunately these environment packs have assets in a more granular setup than
the teams want, but the artists do not have the time to refactor or convert the
entire asset pack.

Because of this, a common form of optimization amongst these teams is to merge
actors that encompass a percentage of the player's camera view in order to
reduce draw calls.

Following this repeated conversation I made this simple mesh merger that allows
artists to select assets in the viewport, and it will merge -> save -> replace
the models in the level for them. According to them, this has been quite
helpful on fixed camera angle games that use a grid system.

This tool will take a level name and a few other inputs to organize the merged
actors into a folder `content/<level_name>/<asset_name>`.

Here is a quick video of it working on some primitives.

<iframe width="560" height="315"
  src="https://www.youtube.com/embed/bcpPRMJeAXU"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen>
</iframe>
