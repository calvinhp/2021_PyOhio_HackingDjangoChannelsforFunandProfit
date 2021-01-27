---
title-prefix: Six Feet Up
pagetitle: Abusing Django Channels for Fun and Profit
author: Calvin Hendryx-Parker, CTO, Six Feet Up
author-meta:
    - Calvin Hendryx-Parker
date: IndyPy 2021
date-meta: 2021
keywords:
    - Python
    - Django
    - Programming
    - Async
    - Websockets
---

# Abusing Django Channels for Fun and Profit

---

Backstory

Channels is awesome and async is becoming more a part of Django with each 3.x release

---

You need to know that a Websocket is...

* Demo LoudSwarm Slack Message on the Front-end

---

You need to know a bit about how Django processes request

HTTP Request -> Response Loop

Channels adds inbound websockets 
    really, it now wraps async views added in 3.1
    3.0 only added an async router, channels was doing the old thing and really only did websockets

Using a fully async event loop

---

Websocket Consumers
Channel Layers  <-- lets the other two communicate
Background Workers <-- Generally not used for incoming websockets, new-ish and allows for background tasks like Celery

---

# A Worker in Channels?

What can you do with it?

They listened for messages on the Channel Layer and then do some work. 

Fast
Easy
Lightweight

Beware: at-most-once operation

Example: Generate Thumbnails

---

I have an idea, Discord requires me to talk to it async via a websocket to receive and send events

We had been doing Slack integration via the webhook events and POSTing messages back

---

# Let's hack our own Worker

We want something that Channels can do, but doesn't out of the box

---

We will do like the Channels `runworker` and make our own from the asgiref.server.StatelessServer

---

Wait, what is ASGI...

Big talk, but here is the TL;DR

See more here:
<https://youtu.be/uRcnaI8Hnzg>

::: notes
This can be its own talk
:::

---

# Why?

We don't want to just respond to a channel layer message to make something happen

We are turning the standard Channels concept a bit inside out.

---

Run a worker, but have it start a long running coroutine on start.

## two examples
we receive messages from Discord and we send them to our clients

We generate a notification on a scheduled celery task and we want to send it to Discord

---

Use this for any old long running task...

--- 

Wish list
* get this added to Channels codebase
* Show examples of long running and single shot coroutines
* Add options for two classifications of coroutines
    * Ones that start right away
    * Optionally ones the run after those stop

---

# Tips and Tricks (aka not in the docs)

* Channel Layer Capacity defaults to 100
    If you push more to a channel group, they drop silently
  

---

# So long and Thanks for all the Fish...

