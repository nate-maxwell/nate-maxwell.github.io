---
layout: post
title: "[GO] Mycelia's Custom Protocol"
date: 2025-08-27
permalink: /custom-protocol/
author: Nate Maxwell
categories:
    - "Update"
tags:
    - "protocol" 
    - "go"
    - "golang"
    - "networking"
    - "messaging"
    - "distribution"
---

Today I am writing a follow-up post to the post about my golang message broker
I posted back in May. I still think it has quite a ways to go before a version
1 release.

Today I thought I would write about the custom protocol I've been developing
for it. It's not fancy or advanced, but it was a fantastic learning exercise.

Originally it was some overly simplistic python code that called:
```python
def _serialize_command(cmd: Command) -> str:
    delimiter = ';;'
    return delimiter.join(cmd.__dict__.keys())

def process_command(cmd: Command, address: str, port: int) -> None:
    """Sends the CommandType message."""
    payload = _serialize_command(message)
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.connect((address, port))
        sock.sendall(payload)
```
```
>>> "1;;SendMessage;;123e4567-e89b-12d3-a456-426614174000;;example_route;;channel1;;10.0.0.52:4401"
```
Where I checked for `\n` characters to denote message termination.

This was great in the very beginning while I focussed my attention on the
structure of the broker. I knew that it needed reworking so that clients could
send messages with `\n` or `;;` inside of them, as well as shortening the
message size to the bare minimum and decoupling the object type from the given
command (`MESSAGE.ADD` VS `send_message`).

### Breakdown
This protocol isn't meant to work with additional applications, only clients
and Mycelia itself.

The protocol comprises 3 parts - a fixed field sized header, a variable field
sized sub-header, and a body.

<img src="https://i.imgur.com/UE35YBK.png">

#### Header
The header contains 4 fields, although I expect one to be removed upon full
release.
* <u>Message Length</u>

    There are two common methods for denoting the end of a message: A body
length prefix, or a termination byte. I've chosen the former.
* <u>Protocol Version</u> (possibly removed in the future)

    This has helped in testing so that I can deploy distributed systems for
    testing while not having to update every single system's client side API.
* <u>Object Type</u>

    What kind of data we are when talking to the broker: { `MESSAGE`,
`SUBSCRIBER`, `TRANSFORMER` }.
* <u>Command Type</u>

    What we what the object to do with the object value that was sent to the
broker: { `SEND`, `ADD`, `REMOVE` }.

This effectively makes the header a 4byte length prefix followed by 3 bytes
that denote protocol version, argument 1, and argument 2.

#### Sub-Header
The sub-header comprises two variable length fields each of which contain a
`uint32` length prefix and a value.
* <u>UID</u>

    The unique message identifier.
* <u>Route</u>

    Which route the object shall take. Messages are ran through every channel
by the route while subscribers and transformers are added to specific channels
on the route, which is why the fields diverge from here.

#### Body
The body currently comes in two forms.
* <u>Subscriber + Transformer</u>

    Both of these objects require the same data, which channel to add
themselves to and where to forward the data. Both fields are like the
sub-header fields, they contain a `uint32` length prefix and a value.

* <u>Message</u>

    Similarly, message bodies have a `uint32` length prefix and value field,
but they only contain the one field. This field is the delivery payload to the
subscriber.

<img src="https://i.imgur.com/6hPqZgC.png">

These are read and fed to the broker as goroutines which then get parsed into
struct objects and ran through the system. Based on the protocol version, some
struct fields are plugged with default values.

I'm still expanding the message handling to the client APIs to account for
errors, confirmations, timeouts, etc., but the base of the protocol is here.
