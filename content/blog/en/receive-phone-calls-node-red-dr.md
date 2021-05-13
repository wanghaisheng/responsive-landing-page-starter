---
title: How to Receive Phone Calls with Node-RED
description: In this tutorial, you'll learn about handling inbound calls using
  Node-RED and the Nexmo Voice API.
thumbnail: /content/blog/receive-phone-calls-node-red-dr/inbound-calls-node-red.png
author: julia
published: true
published_at: 2019-05-09T08:00:44.000Z
updated_at: 2021-05-13T20:28:08.345Z
category: tutorial
tags:
  - voice-api
  - javascript
  - text-to-speech
comments: true
redirect: ""
canonical: ""
---

_This is the third article in a series of "Getting Started with Nexmo and Node-RED" tutorials._

In the [previous tutorials](https://www.nexmo.com/blog/tag/node-red/) you've learnt how to send and receive SMS messages programatically using the Nexmo SMS API and how to handle delivery receipts.

Next, you'll be moving on to the next chapter, exploring the Nexmo Voice API. 

By the end of this article, you'll have handled your first inbound call with Node-RED.

## Prerequisites

Before getting started, you’ll need a few things:

- [Node.js](https://nodejs.org/en/) and [Node-RED](https://nodered.org/docs/getting-started/installation) installed on your machine
- A Nexmo account—[create one for free](https://dashboard.nexmo.com/sign-up?utm_source=DEV_REL&utm_medium=nexmoblog&utm_campaign=receive-calls-nodered) if you haven't already
- Optional: [ngrok](https://ngrok.com/download)—get up to speed with [Aaron's blog post](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/)

### Getting Your Credentials

In order to interact with the Voice API, you'll need to make note of a couple of things. Once you've created a Nexmo account, go to the [dashboard](https://dashboard.nexmo.com) to find your API key and secret.

Next, you'll need a Voice enabled virtual number. Go to Numbers > [Buy numbers](https://dashboard.nexmo.com/buy-numbers) to get one.

<img src="https://www.nexmo.com/wp-content/uploads/2019/04/buy-number-nexmo-dashboard.gif" alt="" width="100%" height="100%" class="alignnone gif-player size-full wp-image-28897" />

### Setting Up Your Node-RED Editor

First, you’ll need to [install](https://nodered.org/docs/getting-started/installation) the runtime and editor. This could be done either on your local machine, on a Single Board Computer (eg Raspberry Pi), or a number of cloud-hosted options. This example will be using your local machine, so once you've installed Node-RED globally, just type the command below in your terminal to get started.

```bash
$ node-red
```
You can then access the Node-RED editor by pointing your browser at [http://localhost:1880](http://localhost:1880).

Once you have your editor open, you'll need to install the Nexmo nodes. You can do so under the _Manage palette_ menu, by searching for the `node-red-contrib-nexmo` package and clicking install. 

<img src="https://www.nexmo.com/wp-content/uploads/2019/05/install-nexmo-nodered.gif" alt="" width="1200" height="690" class="alignnone gif-player size-full wp-image-29232" />

Now you should see all of the Nexmo nodes appear on the left side of your screen, among the other default nodes.

## Handle an Inbound Phone Call with Node-RED

### Exposing Your Local Server to the Internet

First, you'll have to expose your local server to the internet, so that Nexmo can access it. If you’re running Node-RED on a public webserver instead of your local machine, you can skip this stage. 

Otherwise, a convenient way to do this is by using a tunneling service like [ngrok](https://ngrok.com).

[Download](https://ngrok.com/download) and install **ngrok**, then run it in the terminal to start a tunnel on port `1880`.
```bash
$ ./ngrok http 1880
```
Navigate to the URL displayed to find your Node-RED Editor.

<img src="https://www.nexmo.com/wp-content/uploads/2019/05/ngrok-inbound-call.png" alt="" width="1462" height="970" class="alignnone size-full wp-image-29234" />

### Creating a Nexmo Application

In the previous SMS tutorials, you were able to configure a phone number directly with an endpoint; however, this is not always the case.

Some of Nexmo’s APIs, including the Voice API, use Nexmo Applications to hold security and config information needed to connect to Nexmo endpoints. 

In the Nexmo Node-RED palette, several nodes have the ability to create these applications: `getrecording`, `earmuff`, `mute`, `hangup`, `transfer`, `createcall`, `playaudio`, `playtts` and `playdtmf`.

Drag any of these nodes into your workspace, then double-click on it to open up the node editor.

Next to the `Nexmo Credentials`, select "Add new nexmovoiceapp..." from the drop-down menu and click the edit button. Fill in the details below and click `Create New Application`.

| KEY           | DESCRIPTION |
| --------------- | --- |
| `Name`     | Choose a name for your Voice Application, for example `inbound call`.
| `API Key`     | Your Nexmo API key, shown in your [account overview](https://dashboard.nexmo.com/getting-started-guide). |
| `API Secret`  | Your Nexmo API secret, shown in your [account overview](https://dashboard.nexmo.com/getting-started-guide).    |
| `Answer URL`     | YOUR_URL/answer, you'll be hosting a Nexmo Call Control Object (NCCO) here. - more about this later on.|
| `Event URL`     | YOUR_URL/event, you'll need to reference this when setting up the event handler.| 

Node-RED will then create a new Nexmo Application on your account and fill in the App ID and Private Key fields for you to save. After this step, feel free to delete the Nexmo node you used, as a `nexmovoiceapp` config node has been created, and that contains all the Nexmo credentials this flow needs.

<img src="https://www.nexmo.com/wp-content/uploads/2019/05/create-voiceapp.gif" alt="" width="1200" height="675" class="alignnone gif-player size-full wp-image-29229" />

Next, you'll have to link your virtual number to this application.

Find the Voice Application you've just created in your Nexmo Dashboard by navigating to _Voice_ > _[Your Applications](https://dashboard.nexmo.com/voice/your-applications)_.

Click on the name of this application, then under the _Numbers_ tab click on the **Link** button next to the virtual number you've rented earlier.

Alternatively, if the number you'd like to use is already linked to another app, click on **Manage number** and configure it to forward incoming calls to your app.

<img src="https://www.nexmo.com/wp-content/uploads/2019/05/link-number-voiceapp.png" alt="" width="3104" height="1880" class="alignnone size-full wp-image-29233" />

### Build the Nexmo Call Control Object (NCCO)

Nexmo calls are controlled using _Nexmo Call Control Objects_, also known as NCCOs. An NCCO defines a list of actions to be followed when a call is handled. There are lots of different actions available, find the corresponding nodes under the Nexmo palette in your Node-RED editor or check out the [NCCO Reference](https://developer.nexmo.com/api/voice/ncco) to find out more about them.

For this tutorial, you'll be using the `talk` action. 

Drag and drop the **`talk`** node into your workspace, then connect it to a **`voice webhook`** input node and a **`return NCCO`** output node. 

Next, in the **`voice webhook`** node, select `GET` as a method and type something like `/answer` in the answer URL field. 

Finally, go to the **`talk`** node properties and set the `Text{}` field to the message you'd like to be read out when the call is answered. Note the `{}` sign next to the `Text` label, indicating that this value can be set dynamically, using [Mustache templating](https://mustache.github.io/). You can also select a `Voice Name`, see the [Text to Speech Guide](https://developer.nexmo.com/voice/voice-api/guides/text-to-speech#voice-names) for the full list of options.

<img src="https://www.nexmo.com/wp-content/uploads/2019/05/create-tts-ncco.gif" alt="" width="1200" height="750" class="alignnone gif-player size-full wp-image-29228" />

### Setting Up a Handler for the Event URL

Connect a `http` input node to a `http response` node, as well as to a `debug` node, so that you can view your delivery receipt in the debug area.

In the **`http`** input node, select `POST` as a `Method` and fill in the `URL` field with something like `/event`.

The **`http response`** node should have `200` set as `Status code`, but don't worry about it, this is the default value as well.

Now hit **Deploy**, call your virtual number and follow the flow of your call in the debug sidebar.

<img src="https://www.nexmo.com/wp-content/uploads/2019/05/inbound-call-debug.png" alt="" width="3104" height="1880" class="alignnone size-full wp-image-29230" />

## Next Steps

In this tutorial, you've learnt how to play a text-to-speech message to a caller. In a quite similar manner, you could also play an audio file to them, or forward the call to a phone number. If you'd like to take it further, why not record the conversation or set up your custom voicemail? Stay tuned to find out how!

## Resources

- More about the [Voice API](https://developer.nexmo.com/voice/voice-api/overview)
- Check out the [NCCO Reference](https://developer.nexmo.com/voice/voice-api/ncco-reference) to learn about the many ways to control your call.
- [Text to Speech Guide](https://developer.nexmo.com/voice/voice-api/guides/text-to-speech#voice-names)

- [Announcing the Nexmo Node-RED Package](https://www.nexmo.com/blog/2019/02/21/nexmo-node-red-package-dr/)
- [How to Send SMS Messages with Node-RED](https://www.nexmo.com/blog/2019/04/17/send-sms-messages-node-red-dr/)
- [How to Receive SMS Messages with Node-RED](https://www.nexmo.com/blog/2019/04/24/receive-sms-messages-node-red-dr/)
- Have a closer look at [Node-RED](https://nodered.org/docs/)

<script>
window.addEventListener('DOMContentLoaded', (event) => {
    document.querySelectorAll(".gif-player").forEach(image => {
        image.src = image.src.replace(/\.gif$/g, ".png")
        image.addEventListener("click", (event) => {
            if (event.target.src.indexOf(".gif") > 0) {
                image.src = image.src.replace(/\.gif$/g, ".png")
            } else {
                image.src = image.src.replace(/\.png$/g, ".gif")
            }
        })
    })
});
</script>
<style>
.gif-player {
  cursor: pointer;
}
img.alignnone {
  border-width: 0px !important;
}
</style>