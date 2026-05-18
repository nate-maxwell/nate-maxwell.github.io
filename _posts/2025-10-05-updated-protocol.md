---
layout: post
title: "[GO] Rhizome Protocol"
date: 2025-10-05
permalink: /updated-protocol/
author: Nate Maxwell
categories:
    - "Go"
tags:
    - "protocol" 
    - "go"
    - "golang"
    - "networking"
    - "messaging"
    - "distribution"
---

# Rhizome - Signalweave Application Protocol

I've recently moved all of my go applications to a GitHub org that my friend and
I can share for collaborative projects. This currently includes a(n)
* Mycelia Event Broker
* Key-Value Database
* Container App
* Logging System
* Custom Protocol

The idea is that we are building all the underlying tools you might need to build a
service-based back-end system. Because we control so much of what the business
systems are built on, we gain the ability to standardize various things, like
having a singular protocol across each application.

Previously I posted about the custom protocol for Mycelia, taking inspiration
from Kafka to build my own instead of using a common one. The protocol has since
been moved into the org.

The Rhizome protocol is not meant to have concrete fields like `http`. Applications
are meant to build their own API on top of it using its command fields.

<img src="https://i.imgur.com/GP7CLrr.png">

Rhizome has 4 `command argument` fields to allow for the message to invoke
operations in the receiving application (should it be set up to act on them).
With 4 fields of u8 ints the protocol can support `256⁴ = 4,294,967,296` total
command combinations.

The `obj_type` and `cmd_type` fields still act as they did before:
* The `obj_type` signals which application system we are interacting with
* The `cmd_type` signals which action we wish to invoke in that system

And now the 4 command argument fields are values we can pass into that action,
like parameters to a function.

Mycelia can receive a message that has
`cmd_type=transformer, cmd_type=add, arg1=route, arg2=channel, arg3=address, arg4=nil`
which tells the broker to add a transformer to the given route + channel and to
forward messages to the given address.

The number of command fields are largely arbitrary and could possibly be reduced
or extended. For now this has served us very well.
