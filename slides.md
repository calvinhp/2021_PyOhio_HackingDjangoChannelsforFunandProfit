---
title-prefix: Six Feet Up
pagetitle: Hacking Django Channels for Fun and Profit
author: Calvin Hendryx-Parker, CTO, Six Feet Up
author-meta:
    - Calvin Hendryx-Parker
date: DjangoCon Europe 2021
date-meta: 2021
keywords:
    - Python
    - Django
    - Programming
    - Async
    - WebSockets
---

# Hacking Django Channels {data-background-image="images/bermix-studio-aX1hN4uNd-I-unsplash.jpg"}
## for Fun and Profit

<br>

### DjangoCon Europe, June 2021

#### Calvin Hendryx-Parker

::: notes
<span>Photo by <a href="https://unsplash.com/@bermixstudio?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Bermix Studio</a> on <a href="https://unsplash.com/s/photos/dollar?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Backstory {data-background-image="images/nadine-shaabana-LlenwFpj41c-unsplash.jpg"}

* The web was mostly synchronous up until a few years ago
* Async is becoming more a part of Django with each 3.x release

::: notes
Django started life as a synchronous web application server.
<span>Photo by <a href="https://unsplash.com/@nadineshaabana?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Nadine Shaabana</a> on <a href="https://unsplash.com/s/photos/story?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

:::

# Requests and Responses {data-background-image="images/ben-white-4Bs9kSDJsdc-unsplash.jpg"}

![Django Request Response Loop](images/middleware.svg)

See <https://www.youtube.com/watch?v=RLo9RJhbOrQ>

::: notes
You need to know a bit about how Django processes requests.

This is all done synchronously since we are living in WSGI land.

<span>Photo by <a href="https://unsplash.com/@benwhitephotography?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ben White</a> on <a href="https://unsplash.com/s/photos/whisper?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# WSGI vs ASGI {data-background-image="images/thanos-pal-bakq5bepwbQ-unsplash.jpg"}

## Let's make it work bi-directional!

Ok...  so just swap `wsgi.py` with `asgi.py`?

<br>

See <https://arunrocks.com/a-guide-to-asgi-in-django-30-and-its-performance/>

See more on ASGI here: <https://youtu.be/uRcnaI8Hnzg>

::: notes
This can be its own talk

ASGI is a superset of WSGI and can call WSGI callables

Arun Ravindran, the author of Django Design Patterns and Best Practices, has talked about this quite a bit in his blog.

Prior to Django 3, Channels provided ASGI async support to allow for long-running connections such as WebSockets, MQTT and more

Allows us to use a fully async event loop.

<span>Photo by <a href="https://unsplash.com/@thanospal?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Thanos Pal</a> on <a href="https://unsplash.com/s/photos/direction-sign?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# You need to know that a WebSocket is...  {data-background-image="images/neven-krcmarek-67FG6zD8WPQ-unsplash.jpg"}

* Let's see it in action!

::: notes
But why are we wanting to do asgi and async?

Demo of WebSocket usage in the real world.

Channels adds inbound WebSockets 
* really, it now wraps async views added in 3.1
* 3.0 only added an async router, channels was doing the old thing and really only did websockets

Using a fully async event loop

<span>Photo by <a href="https://unsplash.com/@nevenkrcmarek?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Neven Krcmarek</a> on <a href="https://unsplash.com/s/photos/socket?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Enter Channels {data-background-image="images/lara-puscas-bPU9Nsmb4Ms-unsplash.jpg"}

* **Consumers** 👈 What we think when we talk about Channels and WebSockets
* **Channel Layers** 👈 How we talk to and between our code
* **Background Workers** 👈 Allows for background tasks like Celery

::: notes
<span>Photo by <a href="https://unsplash.com/@dancingclouds?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Lara Puscas</a> on <a href="https://unsplash.com/s/photos/channel?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Consumer Example

~~~{.stretch .python}
from channels.generic.websocket import JsonWebsocketConsumer

class MyConsumer(JsonWebsocketConsumer):
    def connect(self):
        self.accept()
        self.send_json({"connected": "true"})
        
    def disconnect(self, close_code):
        pass
        
    def receive_json(self, content, **kwargs)):
        type_code = content["type"]
        data = content["message"]
        
        if type_code == "greeting":
            if (name := data.get("name")):
                self.send_json({
                    "type": "reply_to_greeting",
                    "message": f"hello there, {name}!"
                })
~~~

# ASGI Routing

~~~{.stretch .python}
application = SentryAsgiMiddleware(
    ProtocolTypeRouter(
        {
            # (http->django views is added by default)
            "websocket": AuthMiddlewareStack(
                URLRouter(loudswarm.chat.routing.websocket_urlpatterns)
            ),
            "channel": ChannelNameRouter(
                {
                    "discord_send": DiscordSender,
                }
            ),
        },
    )
)
~~~

::: notes
This is the older Channels 2 router since in 3 the http protocol is require to explicitly be there
:::

# Let's talk about Background Workers {data-background-image="images/omid-armin-kAjXxQ-wQu4-unsplash.jpg"}

### That last bit is where we will focus

::: notes
Background Workers are new to channels and less used, but will do the bit we really want

<span>Photo by <a href="https://unsplash.com/@omidarmin?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Omid Armin</a> on <a href="https://unsplash.com/s/photos/workers?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# What if we want the reverse? {data-background-image="images/nick-fewings-S7cyjr_3prc-unsplash.jpg"}

::: notes
A web browser establishing a bi-directional channel to our app is one thing.

What if we want our app to establish a channel to some other long running service?

<span>Photo by <a href="https://unsplash.com/@jannerboy62?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Nick Fewings</a> on <a href="https://unsplash.com/s/photos/arrow?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# A Worker in Channels? {data-background-image="images/antonio-molinari-qYuhYRDNKbA-unsplash.jpg"}

<br>

What can you do with it?

::: notes
<span>Photo by <a href="https://unsplash.com/@amolinari?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">antonio molinari</a> on <a href="https://unsplash.com/s/photos/gondola?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Listen for messages on the Channel Layer and then do some work. {data-background-image="images/guillaume-jaillet-Nl-GCtizDHg-unsplash.jpg"}

* Fast
* Easy
* Lightweight

<br>

**Beware:** at-most-once operation

::: notes
Example: Generate Thumbnails

<span>Photo by <a href="https://unsplash.com/@i_am_g?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Guillaume Jaillet</a> on <a href="https://unsplash.com/s/photos/fast?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Real World Use Case {data-background-image="images/phillip-glickman-2umO15jsZKM-unsplash.jpg"}

<br>

## Discord Chat Bot

<br>

### Built inside of Django

::: notes
I have an idea, Discord requires me to talk to it async via a websocket to receive and send events

We had been doing Slack integration via the webhook events and POSTing messages back

Why is the inside of Django part important?  We want to use cool batteries included like the ORM!

<span>Photo by <a href="https://unsplash.com/@phillipglickman?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Phillip Glickman</a> on <a href="https://unsplash.com/s/photos/robot?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Let's hack our own Worker {data-background-image="images/markus-spiske-iar-afB0QQw-unsplash.jpg"}

We want something that Channels can do, but doesn't out of the box

::: notes
<span>Photo by <a href="https://unsplash.com/@markusspiske?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Markus Spiske</a> on <a href="https://unsplash.com/s/photos/hacker?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

---

We will do like the Channels `runworker` and make our own from the `asgiref.server.StatelessServer`

~~~{.stretch .python}
# Only the one method is different from the default worker

    async def handle(self):
        """
        Listens on all the provided channels and handles the messages.
        """
        # For each channel, launch its own listening coroutine
        lackeys = []
        for channel in self.channels:
            lackeys.append(asyncio.ensure_future(self.listener(channel)))

        # Add coroutine for outgoing websocket connection to Discord API
        lackeys.append(asyncio.ensure_future(self.outgoing_connection()))

        # Most of our time is spent here, waiting until all the lackeys exit
        await asyncio.wait(lackeys)
        # See if any of the listeners had an error (e.g. channel layer error)
        [lackey.result() for lackey in lackeys]
~~~

# Why? {data-background-image="images/mikel-parera-yB7ghV4dSa4-unsplash.jpg"}

::: notes
We don't want to just respond to a channel layer message to make something happen

We are turning the standard Channels concept a bit inside out.

<span>Photo by <a href="https://unsplash.com/@mikelparera?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Mikel Parera</a> on <a href="https://unsplash.com/s/photos/question?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

---

## Two Examples

We receive messages from Discord and we send them to our clients

~~~{.stretch .python}
class ChatClient(discord.Client):
    async def on_message(self, message):
        data = {
            "channel": str(message.channel.id),
            "event_ts": str(message.created_at.timestamp()),
            "text": message.content,
            "ts": str(message.created_at.timestamp()),
            "user": message.author.nick,
            "event_id": str(message.id),
            "event_time": int(message.created_at.timestamp()),
            "team_id": str(message.guild.id),
        }

        await make_webhook_transaction(data)
~~~

::: notes
Run a worker, but have it start a long running coroutine on start.
:::

---

We generate a notification on a scheduled celery task and we want to send it to Discord so we write a consumer that listens for our message

~~~{.stretch .python}
from discordworker import discord_client

class DiscordSender(AsyncConsumer):
    async def send_to_discord(self, message):

        text_msg = message["message"]
        chat_chan = int(message["discord_channel"])
        channel = discord_client.client.get_channel(chat_chan)

        await channel.send(text_msg)  # this is Discord's "channel", not ours
~~~

---

and we send it like this

~~~{.stretch .python}
await channel_layer.send(
    "discord_send",  # name from ChannelNameRouter config
    dict(
        type="send_to_discord",  # maps to method name on consumer
        discord_channel=self.chat_chan,
        message=message,
    ),
)
~~~

# {data-background-image="images/jenny-hill-mQVWb7kUoOE-unsplash.jpg"}

Use this for any old long running task...

::: notes
<span>Photo by <a href="https://unsplash.com/@jennyhill?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Jenny Hill</a> on <a href="https://unsplash.com/s/photos/runner?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Wish list {data-background-image="images/melvina-mak-WKwBxFi96FY-unsplash.jpg"}

* Get this added to Channels codebase
* Show examples of long running and single shot coroutines
* Add options for two classifications of coroutines
    * Ones that start right away
    * Optionally ones the run after those stop
    
::: notes
<span>Photo by <a href="https://unsplash.com/@melvina?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Melvina Mak</a> on <a href="https://unsplash.com/s/photos/wish?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# Tips and Tricks {data-background-image="images/jared-rice-xce530fBHrk-unsplash.jpg"}
## (aka not in the docs)

* Channel Layer Capacity defaults to 100
    If you push more to a channel group, they drop silently
  
::: notes
<span>Photo by <a href="https://unsplash.com/@jareddrice?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Jared Rice</a> on <a href="https://unsplash.com/s/photos/meditate?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::

# So long and Thanks for all the Fish... {data-background-image="images/pete-pedroza-VyC0YSFRDTU-unsplash.jpg"}

::: notes
<span>Photo by <a href="https://unsplash.com/@peet818?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Pete Pedroza</a> on <a href="https://unsplash.com/s/photos/thanks?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
:::
