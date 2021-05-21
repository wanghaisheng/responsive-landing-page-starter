---
title: Best practices to get started with the Vonage Video API
description: "Best practices we recommend for consideration, before you start
  building your feature-rich video application powered by Vonage Video API. "
thumbnail: /content/blog/best-practices-to-get-started-with-the-vonage-video-api/videoapi_sdk-release_1200x600.png
author: tony-chan
published: false
published_at: 2021-05-24T09:00:13.510Z
updated_at: 2021-05-21T12:57:13.538Z
category: tutorial
tags:
  - video
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
This document describes some of the best practices we recommend for consideration, before you start building your feature-rich video application powered by Vonage Video API. 

[Visit our website](https://www.vonage.com/communications-apis/video) to set up your new account; it’s free and your account will automatically be credited ten US dollars ($10).

# Where to get more help?

Very detailed developer documentation on the Vonage Video API is publicly available on our [Video API Developer Site](https://tokbox.com/developer) - here you will find all the details you need for basically any question you might have, sample codes, release notes. There is also a great section called “[Get answers to your Video API questions](https://support.tokbox.com/hc/en-us)”.

To help us better assist you, please send us your feedback via email at: [video.api.support@vonage.com](video.api.support@vonage.com).

# Video Platform

Vonage video uses webRTC for audio-video communications and consists of client libraries for web, IOS and Android, as well as server SDKs and REST API. More information can be found here <https://tokbox.com/developer/guides/basics/#opentok-platform>.

Key Terms:

* Session:  a logical group of connections and streams. Connections within the same session can exchange messages. Think of a session as the “room” where participants can interact with each other.
* Connection: an endpoint that participates in a session and is capable of sending and receiving messages. A connection has presence, it is either connected and can receive messages, or it’s disconnected. 
* Stream: media stream between two connections. This means that actual bytes containing media are being exchanged. 
* Publisher: clients publishing stream.
* Subscriber: clients receiving stream.

# Environment

When designing a video application, consider having two environments; one for testing and one for production. To test simple items, you can also use [our playground](https://tokbox.com/developer/tools/playground_doc/) or you can use the [OpenTok command line interface](https://www.npmjs.com/package/opentok-cli).

For Enterprise server customers, it is important to note that newly added API keys will be using the Standard environment by default. If you need to switch an API key’s environment from Standard to Enterprise, you can do so on our account portal. 

The Enterprise JS SDK is available at <https://enterprise.opentok.com/v2/js/opentok.min.js>. For more info, visit <https://tokbox.com/developer/enterprise/content/enterprise-overview.html>.

# API key/secret, Tokens and Session IDs

* API Key and secret

  * Keep secret/key private by NOT exposing them to public repos.
  * Do NOT save secret/key in client libraries/compiled mobile SDKs.
  * Use HTTPS only to make REST calls.
* Session ID

  * Always generate a new sessionId for every new session created.  
  * Sessions’ quality scores and data are indexed by sessionId. If there are multiple conversations per sessionId, it will be difficult to debug using Vonage’s inspector tool as reused sessionIds tend to report lower aggregate quality scores than the actual call quality experienced by end users.
* Tokens

  * Server that generates tokens must be placed behind a secured/authenticated endpoint
  * Always generate new tokens for each participant
  * Do not store or reuse tokens. 
  * By default, tokens expire after 24 hours, this is checked at connection time. Adjust the expiration as needed, depending on your use case and application. 
  * Add additional information to tokens (using the data parameter) such as usernames or other information that can identify participants, but NEVER use personal information. This will help in troubleshooting.
  * Set roles when applicable such as moderator, publisher and subscriber.
  * More information about tokens can be found at <https://tokbox.com/developer/guides/create-token/>.

# Media Modes

* **Relayed** - this media mode does not use Vonage media servers. Before deciding whether to use relayed mode or not, be sure to check the following:

  * That recording is not needed
  * One-to-one session ONLY
  * End-to-end media encryption is required

Note that media quality will not be managed on relayed mode since media is exchanged  between clients. Therefore, setting subscriber’s frame rate and/or resolution will not work. See <https://tokbox.com/developer/guides/scalable-video/> for more information.

* **Routed** - this media mode uses Vonage media servers. Before deciding whether to use routed mode or not, be sure to check the following:

  * Three or more participants
  * May have a need to archive
  * Needs media quality control (audio fallback and video recovery)
  * May have a need to use SIP interconnect
  * May have a need to use interactive or live streaming broadcast

More information about media modes can be found at: <https://tokbox.com/developer/guides/create-session/>.

# Broadcast

* **Interactive** - this type of broadcast allows clients to interact with each other by subscribing to each other's stream. Important to note that this type of broadcast can only support up to 3,000 subscribers; anything above that will generate an error message. Below are things to consider when using this broadcast:

  * Contact support and have them enable simulcast. Visit <https://support.tokbox.com/hc/en-us/articles/360029733831-TokBox-Scalable-Video-Simulcast-FAQ> to learn more about Simulcast. By default Simulcast is set to heuristic in all API keys, this means that Simulcast will only kick in after the third connection joins the call (this is done to avoid Siimulcast in one-to-one calls). Important to note that the first two connections won’t use Simulcast if it is set to heuristic.
  * Allow no more than five publishers. Keep in mind that the maximum number of subscribers will be impacted when streams increase. To get the max subscribers based on the number of publishers, divide the max participants (3,000) by the number of publishers. Example: two publishers can have 1,500 subscribers (3,000 divided by 2).
  * Suppress connection events. 
  * See <https://tokbox.com/developer/guides/broadcast/live-interactive-video/>for more information
* **Live Streaming** - this type of broadcast allows for more than 3,000 subscribers to subscribe to streams. There are two types of protocols available to broadcast video, RTMP (Real Time Messaging Protocol) and HLS (HTTP Live Streaming). Regardless of which one you choose, limit the number of publishers to five for better experience.

## HLS vs RTMP

* HLS supports unlimited number of subscribers where RTMP is limited by the RTMP delivery platform
* HLS is delayed by 15-20 seconds where RTMP, from Vonage’s platform, is delayed by five seconds; this does not include the delay from RTMP delivery platform however as they too will induce delay based on how they process video.
* HLS playback is not supported on all browsers but there are plugins that you can use such as flowplayer. Playback allows users to go back, video scrubbing (rewind/fast forward) if you will, from the beginning of the livestream then back to the current live stream.
* HLS/RTMP has a max duration of two hours. If broadcast needs to go longer, change the max duration property (max is 10 hours).
* HLS/RTMP stream automatically stops sixty seconds after the last client disconnects from the session

To learn more about live streaming such as layouts, max duration and how to start/stop live stream, visit <https://tokbox.com/developer/guides/broadcast/live-streaming/>.