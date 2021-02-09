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
    - WebSockets
---

# Abusing Django Channels
## for Fun and Profit

</br>

### IndyPy, Feb 2021

#### Calvin Hendryx-Parker
#### Peter Hull

---

# Backstory

* The web was mostly synchronous up until a few years ago
* Async is becoming more a part of Django with each 3.x release

::: notes
Django started life as a synchronous web application server.

:::

---

# Requests and Responses

![Django Request Response Loop](images/middleware.svg)

See <https://www.youtube.com/watch?v=RLo9RJhbOrQ>

::: notes
You need to know a bit about how Django processes requests.

This is all done synchronously since we are living in WSGI land.
:::

---

# WSGI vs ASGI

## Let's make it work bi-directional!

Ok...  so just swap `wsgi.py` with `asgi.py`?

</br>

See <https://arunrocks.com/a-guide-to-asgi-in-django-30-and-its-performance/>

::: notes
ASGI is a superset of WSGI and can call WSGI callables

Arun Ravindran, the author of Django Design Patterns and Best Practices, has talked about this quite a bit in his blog.

Prior to Django 3, Channels provided ASGI async support to allow for long-running connections such as WebSockets, MQTT and more

Allows us to use a fully async event loop.
:::

---

## You need to know that a WebSocket is...

* Let's see it in action!

::: notes
But why are we wanting to do asgi and async?

Demo of WebSocket usage in the real world.

Channels adds inbound WebSockets 
* really, it now wraps async views added in 3.1
* 3.0 only added an async router, channels was doing the old thing and really only did websockets

Using a fully async event loop
:::

---

# Enter Channels

* **Consumers** 👈 What we think when we talk about Channels and WebSockets
* **Channel Layers** 👈 How we talk to and between our code
* **Background Workers** 👈 Allows for background tasks like Celery

</br>

### That last bit is where we will focus

::: notes
Background Workers are new to channels and less used, but will do the bit we really want
:::

---

# What if we want the reverse?

::: notes
A web browser establishing a bi-directional channel to our app is one thing.

What if we want our app to establish a channel to some other long running service?
:::

---

# A Worker in Channels?

* What can you do with it?

## Listen for messages on the Channel Layer and then do some work. 

* Fast
* Easy
* Lightweight

**Beware:** at-most-once operation

::: notes
Example: Generate Thumbnails
:::

---

# Real World Use Case

</br>

## Discord Chat Bot

</br>

### Built inside of Django

::: notes
I have an idea, Discord requires me to talk to it async via a websocket to receive and send events

We had been doing Slack integration via the webhook events and POSTing messages back

Why is the inside of Django part important?  We want to use cool batteries included like the ORM!
:::

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

::: notes
We don't want to just respond to a channel layer message to make something happen

We are turning the standard Channels concept a bit inside out.
:::

---

Run a worker, but have it start a long running coroutine on start.

## Two Examples

we receive messages from Discord and we send them to our clients

We generate a notification on a scheduled celery task and we want to send it to Discord

---

Use this for any old long running task...

--- 

# Wish list

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

