---
title: Proxy Voice Calls Anonymously with Express
description: In this tutorial, we'll learn how to create an anonymous voice
  proxy using Nexmo Voice APIs, Nodejs, JavaScript and the Express framework.
  Protect your users' privacy by concealing their telephone numbers.
thumbnail: /content/blog/voice-proxy-node-javascript-express-dr/anonymous-voice-proxy-featured-image.png
author: aaron
published: true
published_at: 2018-05-22T14:53:51.000Z
updated_at: 2021-05-20T10:14:52.037Z
category: tutorial
tags:
  - node
  - voice-api
comments: true
redirect: ""
canonical: ""
---
We've all been there: you've gotten out of your cab and a few minutes later you realise you've left your umbrella behind. Alternatively, you're starving, staring out the window whenever you hear a car drive by, wondering where the food delivery you ordered an hour ago is.

<img src="https://www.nexmo.com/wp-content/uploads/2018/05/waiting-on-delivery.gif" alt="Girl waiting on a delivery" width="480" height="270" class="alignnone size-full wp-image-20893" />

Perhaps it's the other way around. Maybe you're making a delivery and you need some directions to the address.

In all these situations you're going to need to call the other person, but you don't want to be giving your phone number out to strangers. So instead you can use [a virtual number](https://www.nexmo.com/products/phone-numbers) which connects the two parties but is only usable for the duration of the current transaction.

## Renting a Virtual Number

In this tutorial, we're going to use [the Nexmo CLI](https://github.com/Nexmo/nexmo-cli) to rent our virtual number, but you can also manage your numbers and voice applications via [the Nexmo dashboard](https://dashboard.nexmo.com/buy-numbers) if you'd prefer to use a GUI. If you haven't done so already, you can [install the Nexmo CLI via npm/yarn](https://github.com/Nexmo/nexmo-cli#installation). Full instructions are in the [Github repository](https://github.com/Nexmo/nexmo-cli).

At the time of writing, you can rent virtual numbers in 85 different countries. I'm going to use GB (Great Britain) in my example; you can see a [complete list of countries and prices on our pricing page](https://www.nexmo.com/pricing).

```sh
    nexmo number:search GB
    nexmo number:buy <NUMBER>
```

## Creating Our Voice Proxy Server

At the moment if you attempt to call the virtual number we just rented, nothing happens.

<img src="https://www.nexmo.com/wp-content/uploads/2018/05/on-the-phone.gif" alt="Man on the phone" width="500" height="322" class="alignnone size-full wp-image-20896" />

We have to associate the virtual number with an application. You can link multiple numbers to the same voice application, allowing you to have multiple numbers—even in different countries—all powered by a single backend.

In this example, we're going to create our backend with Node and Express and deploy it on [Glitch](https://glitch.com/). You can [view the code on Github](https://github.com/nexmo-community/anonymous-voice-proxy-glitch-server) or [remix it directly on Glitch](https://glitch.com/edit/#!/remix/jungle-pigeon).

[![Remix on Glitch](https://cdn.glitch.com/2703baf2-b643-4da7-ab91-7ee2a2d00b5b%2Fremix-button.svg)](https://glitch.com/edit/#!/remix/jungle-pigeon)

Our Express server has a single endpoint, which looks like this:

```javascript
    app.get("/", (request, response) => {
        response.json([
            {
                "action": "connect",
                "eventUrl": [`${process.env.EVENTS_URL}`],
                "from": `${process.env.FROM_NUMBER}`,
                "endpoint": [
                    {
                        "type": "phone",
                        "number": `${process.env.TO_NUMBER}`
                    }
                ]
            }
        ])
    })
```

The route defined above returns an [NCCO (Nexmo Call Control Object)](https://developer.nexmo.com/api/voice/ncco), a JSON file which is used to provide instructions to the Nexmo API when someone answers an inbound or outbound call. An NCCO can contain many different types of actions. You can [view the available actions in our developer docs](https://developer.nexmo.com/api/voice/ncco).

Our proxy server requires a single action [`connect`](https://developer.nexmo.com/api/voice/ncco#connect). With this we can proxy our incoming call to a range of different endpoints: another phone number, a WebSocket, or even a SIP URI. In the example above we connect to another phone number.

One of the requirements of the `connect` action is that the `process.env.FROM_NUMBER` *must* be a Nexmo Virtual Number. This virtual number is what your call recipient sees. You can use the same virtual number you rented above, that way your caller and callee see the same virtual number, keeping their numbers private.

When you [remix the app](https://glitch.com/edit/#!/remix/jungle-pigeon), you need to configure the `FROM_NUMBER` and `TO_NUMBER` in your Glitch `.env`  file. These numbers need to be in the [E.164 format](https://en.wikipedia.org/wiki/E.164). We're not using the `EVENTS_URL` in this example, but if you're interested in how you can track analytics about your voice calls, then you should watch our webinar ["Inbound Voice Call Campaign Tracking with Nexmo Virtual Numbers and Mixpanel"](https://www.youtube.com/watch?v=gm-XUvUwgyc) or [read the accompanying blog post](https://learn.vonage.com/blog/2017/08/03/inbound-voice-call-campaign-tracking-dr/).

## Linking Our Virtual Number to Our Proxy Server

To link our virtual number to our proxy server on Glitch we first need to create a [Nexmo Voice Application](https://developer.nexmo.com/concepts/guides/applications). You can [create a voice application and link it to your number using the Nexmo dashboard](https://dashboard.nexmo.com/voice/create-application), or via [the Nexmo CLI](https://github.com/Nexmo/nexmo-cli).

```sh
    nexmo app:create "Application name" <GLITCH_URL> <EVENTS_URL>
    nexmo link:app <NUMBER> <APPLICATION_ID>
```

The [Application Overview](https://developer.nexmo.com/concepts/guides/applications) and the [Nexmo CLI README](https://github.com/Nexmo/nexmo-cli#applications) contain more information on `app:create` and the expected arguments.

## Where to Next?

Read the "[private voice communication](https://developer.nexmo.com/tutorials/private-voice-communication)" tutorial for a more in-depth example. For an example of [the proxy server in Kotlin, watch my webinar](https://www.youtube.com/watch?v=pHf9Df3Ns2U). Alternatively, for more information on what else you can do with [Nexmo Voice APIs](https://www.nexmo.com/products/voice) view our [example use cases](https://www.nexmo.com/use-cases) or read [the developer documentation](https://developer.nexmo.com/voice/voice-api/overview).
