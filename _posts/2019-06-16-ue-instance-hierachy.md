---
layout: post
title: "[UE] Material Instance Hierarchy"
date: 2019-06-16
permalink: /ue-material-instance-hierarchy/
author: Nate Maxwell
categories:
    - "Unreal"
tags:
    - "unreal"
    - "shader"
    - "material"
---

Today's post will be very short. I've been increasingly frustrated with certain
artists competing for the 'better' master material. Mainly competing over the
better instruction to feature ratio. In production this has been painful as
they want to try out differing master materials between builds but have been
pasting node networks into the master material graph and then saving and using
perforce to revert if needed.

This has led to me adopting 'master instances' as a standard practice. Material
instances can be instances of another instance instead of the source material.
This is super useful if you create a master instance which all other instances
are derived. And depending on how extensive your master material is, you can
begin to make 'master group' instances.

For example, you could have a master material which is the parent of a master
instance. And from there you could have a master subsurface instance and a
master hard surface instance which have differing preset values.

Additionally, we now retain the ability to swap the source material of the
master instance itself if we want to try something radically different. To keep
the same values in our instances, however, we need to retain the same variable
names. Unreal saves these in the instance uassets as a map of string names to
values.

Interestingly, it keeps these values if you swap to a material that doesn't
share the previous one's values and then swap back.

Here is the basic instance hierarchy I've settled on at work.

<img src="https://i.imgur.com/ovar1YD.png">
