---
title: Explore Vonage Video API Capabilities with Vonage Video API Playground
description: Learn how to use the Vonage Video API Playground, a demo of a web
  app where you can explore basic Vonage Video API capabilities
thumbnail: /content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/videoapi_playground_1200x600.png
author: yukari-dumburs
published: false
published_at: 2021-08-10T13:09:13.499Z
updated_at: 2021-08-10T13:09:13.537Z
category: tutorial
tags:
  - video-api
  - playground
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## What Is the ‘Playground’?

Vonage Video API Playground is a demo of a web app where you can explore basic Vonage Video API capabilities. You can publish a stream and subscribe to it using your own project API key. It is based on the OpenTok.js SDK and you can choose the SDK version up to the second to last. 

## Prerequisites

To [access the Playground](https://tokbox.com/developer/tools/playground/), you will need a [Vonage Video API account](https://tokbox.com/account/#/).

<sign-up></sign-up>

## How to Use Playground

You don’t need to write any code. Playground requires you to log in to your Video API account if you haven't logged in yet. 

What you can do with Playground:

* Publish a stream
* Subscribe to a stream
* Signaling API
* Archiving
* Broadcasting
* Dial SIP

## Demo

I’m going to show you how to publish a stream and subscribe to it using Playground.

1. In a desktop browser, go to [Playground](https://tokbox.com/developer/tools/playground/).

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.07.48.png)

2. Click on **Create** to create a session. 

> Optional: Change account ID or project API key, JS SDK version, Media mode, Token role or expiration time.

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.11.23.png)

3. Click on **Connect** to connect to a session.

> Optional: You can configure your own TURN server or IP Proxy from the **Advanced Firewall Control** menu.

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.12.04.png)

4. Click on **Publish Stream**.

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.13.05.png)

5. Click on **Publish**.

> Optional: Change publisher option. Choose videoSource as ‘screen’ for screen-sharing.

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.15.18.png)

Now you are publishing a stream in the session. You can also use Signaling, Archiving, Broadcasting, and SIP here.

6. Next, let’s join the session from another device. Click on **Session information** and copy the session ID.

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.17.13.png)

7. From another device, browse to <https://tokbox.com/developer/tools/playground/> 

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot-2021-07-30-at-12.23.35.png)

8. Click on **Join existing session** and paste the session ID. Click on **Join Session**. 


9. Publish a stream from **Publish Stream**. Now you are interactively publishing and subscribing.

![](/content/blog/explore-vonage-video-api-capabilities-with-vonage-video-api-playground/screenshot_20210730-122136.png)

Once the session is finished, click on **Disconnect** to disconnect from the session.