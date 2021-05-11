---
title: How to Handle User Input With ASP.NET Core
description: Learn how to handle an inbound call then respond to user input
  using ASP.NET Core and Nexmo Voice API.
thumbnail: /content/blog/how-to-handle-user-input-with-asp-net-core-dr/How-to-handle-User-Input-with-ASP.NET-Core.png
author: bibi
published: true
published_at: 2019-01-10T10:06:55.000Z
updated_at: 2021-05-11T10:09:00.368Z
category: tutorial
tags:
  - voice-api
comments: true
redirect: ""
canonical: ""
---
*This is the fourth tutorial on how to use Voice APIs with ASP.NET series.* In the previous tutorial, we learnt [how to Receive a Phone Call with Nexmo Voice API, ASP.NET Core and NancyFX](https://www.nexmo.com/blog/2018/11/21/how-to-receive-a-phone-call-with-nexmo-voice-api-asp-core-core-and-nancyfx-dr/). This is a great start, but in a real life scenario we expect some sort of interaction with the caller. Maybe they will be prompted to pick an option or enter a PIN. We need a way to handle user input. That's exactly the aim of this tutorial; we will create an ASP.NET app that handles inbound voice calls and respond to user input using the [Nexmo Voice API](https://developer.nexmo.com/voice/voice-api/overview).

## Learning objectives

In this tutorial, we will: 

* Create an ASP.NET Core app. 
* Use NancyFX with ASP.NET Core. 
* Create a Nexmo voice application. 
* Receive inbound calls within the app. 
* Create and return NCCOs. 
* Handle user input. 
* Run and test the code using Ngrok.

## Prerequisites

<sign-up></sign-up>

* Visual Studio 2017.  
* A project setup for this tutorial series, which you can find on [Github](https://github.com/nexmo-community/nexmo-dotnet-quickstart/tree/ASPNET/NexmoDotNetQuickStarts). 
* Optional: [The Nexmo CLI](https://github.com/Nexmo/nexmo-cli).

## Configuration

To use [The Nexmo Voice API](https://developer.nexmo.com/voice/voice-api/overview), we need to create [a voice application](https://developer.nexmo.com/concepts/guides/applications).The configuration steps are detailed in [the “Nexmo Voice API with ASP.NET: Before you start” post](https://www.nexmo.com/blog/2017/07/28/nexmo-voice-api-asp-net-configure-dr/). Once the configuration is done successfully, we are ready to receive an inbound call and handle user input with The Nexmo Voice API!