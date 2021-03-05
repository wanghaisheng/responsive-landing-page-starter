---
title: Connecting Voice Calls to an Amazon Lex Bot
description: "This tutorial will help you to start with an example Amazon Lex
  bot and interact with it from Voice Calls using provided sample reference
  codes using Vonage Voice API. "
thumbnail: /content/blog/connecting-voice-calls-to-an-amazon-lex-bot/amazonlex_voiceapi_1200x627.png
author: martyn
published: true
published_at: 2021-03-05T14:16:40.166Z
updated_at: ""
category: tutorial
tags:
  - voice-api
  - lex
  - aws
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Today’s AI platforms offer advanced capabilities with voice and text bots, transcription engines, and sentiment analysis engines.

All of these capabilities help developers to build bots that serve a multitude of functions, including helping callers in placing orders, making a booking, getting assistance with issues, and much more.

In this tutorial, we will guide you on setting up a basic Amazon Lex bot, setting up a Lex reference connection, and a sample Vonage Voice API application, which will allow you to call a phone number and interact with the Lex bot via voice.

In the first section, we will set up an example Amazon Lex bot.

Next, we will set up a Lex reference connection. This Lex reference connection allows 2-way audio streaming of the caller’s voice and Lex bot responses via [WebSockets](https://www.vonage.com/communications-apis/platform/websockets/). It also relays the transcriptions and sentiment scores from Lex bot to the Voice API application.

After that, we will set up a sample application using [Vonage Voice API](https://developer.nexmo.com/voice/voice-api/overview) that will handle an incoming call, establish the WebSocket, receive the caller and bot transcriptions, and the sentiment analysis scores of a caller’s sentences.

The diagram below shows an overview of this sample integration architecture.

![A diagram that outlines how Amazon Lex and the Vonage Voice API work together to create a voice bot](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/lex.png "A diagram that outlines how Amazon Lex and the Vonage Voice API work together to create a voice bot")

## Prerequisites

To perform the actions outlined in the following steps you will need an Amazon Web Services account. If you don't have one, you can [create one](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html?nc2=h_ct&src=header_signup) before getting started.

<sign-up number></sign-up>

## Set up an Example Amazon Lex Bot

[Log in to your AWS Management Console](https://console.aws.amazon.com) or [create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html?nc2=h_ct&src=header_signup) if you do not yet have one.

Search for and then go to Amazon Lex service, click on **Create** to launch the bot setup process.

![The Amazon Lex service dashboard](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-01.png "The Amazon Lex service dashboard")

As part of the setup process, you will be presented with the option to **Create Your Own** bot, or to **Try a Sample.** You'll need a sample bot for this example, so click on **ScheduleAppointment**.

![The Amazon Lex Bot Creation Dashboard](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-02.png "The Amazon Lex Bot Creation Dashboard")

Next, you can set up the bot.

* You may keep the pre-set Bot name or enter a different one
* Make note of the bot name (e.g. ScheduleAppointment), you will need it later
* Select the desired language
* Select Yes for Sentiment Analysis.