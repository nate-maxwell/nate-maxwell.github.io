---
layout: post
title: "UE Distributed Automation"
date: 2025-08-14
permalink: /ue-distributed-automation/
categories:
    - "Unreal"
---

# Distributing Automation In Unreal

My roommate recently set up a build machine with a jenkins bot to make nightly builds of his indie game [Asteros](https://store.steampowered.com/app/2991430/Asteros/) on Steam.

However, on occasion, he would want the machine to make a build immediately after adjusting some critical system.
This would require him to remote into the machine and run the bat file jenkins would load to create another test build.

<img src="https://i.imgur.com/0XrBjj8.jpg">

Coincidentally, at the same time, I had been experimenting with event brokers for distributed pipeline events at work.
This boiled down to artist machines only writing out the file and then sending a description of the data to distributed service machines to log, track,
validate, format, and notify invested parties about the published asset. This greatly sped up the artist workstation, since a lot of these instructions
were moved off of their box and onto the network.

Achieving this was fairly straightforward using a pretty standard pub/sub model machine that directed information between all the other service boxes.
This way artist workstations only needed to connect to one other machine, and they could gain access to full pipeline functionality.
This all sounds way more interesting than it really is.

This was achieved using my, currently simple, event broker Mycelia.
Mycelia is an event broker written in Go to leverage its concurrency model for speed.
The primary architecture uses channels in routes and subscribers can specify which channel in a route they would like to receive event notifications from.
The event then travels through all channels in a route, and each channel applies its transformers to the event data, then passes the transformed event to
all subscribers, and then finally to the next channel in the route.

This allows for data transformation to easily be hosted within the broker, rather than using a separate service that re-sends the enriched data back to
the broker for another trip, making observability and event tracking more painful. Subscribers specify where in the channel stream they would like to receive data.

<img src="https://i.imgur.com/q3cwIBJ.png">

Mycelia is in the early days of development and uses an incredibly primitive custom protocol.
Some friends and I who are working on it have written APIs for clients to talk to the broker machine in Python, Go, and C#.

The point of this entry isn’t really about the inner workings of Mycelia, but rather getting a developer’s Unreal client to talk directly to another machine.

Since I have the python API for Mycelia, if we add it to the python sub folder of the content directory or a plugin directory, or add it to the sys path
in the running UE client, we can communicate directly from there. All we need is a procedural way to build messages and we’re good.

Here's our super advanced gui that sends the string “hello world” in its apex form with no capitals or punctuation to the broker.

<img src="https://i.imgur.com/wylWdOc.png">

Put simply, we package up the gui values and invoke the api python function directly from the widget, as shown in this blueprint function.

<img src="https://i.imgur.com/5dVkxii.png">

Here I have set up a simplistic test running the broker and a mock subscriber on the same machine as the unreal client.
On the left you can see the Mycelia broker terminal and on the right is the consumer, which just prints its incoming data to std out.

<img src="https://i.imgur.com/lWVwuZl.png">

It's easy now to imagine setting up a bunch of machines, all connected to the broker, listening for when a developer signals an event.
These could be anything from triggering a build to pushing up database updates to the online service handler, or submitting updates to in-game shop json tables to the server backend.

Currently, I have been doing it to batch update shaders from the engine at the end of every week after artists have republished them.
