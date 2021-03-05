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

![The Lex bot setup screen from Amazon Web Services](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-03a.png "The Lex bot setup screen from Amazon Web Services")

* Select the desired language
* Select *Yes* for Sentiment Analysis
* For this specific bot example, you may select *No* for COPPA

Once the settings are completed, click on **Create**.

![Final steps in the bot setup process](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-03b.png "Final steps in the bot setup process")

When the bot creation process has completed you will see a screen similar to the one shown below.

![The bot has been created successfully](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-04.png "The bot has been created successfully")

Next, in order to use your bot, you need to publish it by clicking **Publish**. You will then be asked to choose an alias for the bot. Aliases are good for naming different versions of your bot such as the one you use for development, or the one you use for production.

Make note of the alias name you have entered (e.g. staging). You will need it later.

![Adding an alias name to your Amazon Lex Bot](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-05b.png "Adding an alias name to your Amazon Lex Bot")

Clicking **Publish** once more will finalize the setup of your sample bot.

![Notification that the bot has been published](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-05c.png "Notification that the bot has been published")

## Set up the Lex Reference Connection

* Go to <https://github.com/><public-repo-org>/lex-reference-connection,
* Follow instructions as described in this repository.
* Select one of the deployment types as explained in the Running Lex reference connection code section.
* Make note of the `LEX_REFERENCE_CONNECTION` argument (e.g. xxxx.ngrok.io, or myserver.mycompany.com:40000), as it will be needed later.

## Set up the Lex sample Voice API Application

* Go to <https://github.com/><public-repo-org>/lex-sample-voice-application and follow the instructions as described in this repository.
* Select one of the deployment types as explained in the Running Lex sample Voice API application section,
* Make note of the phone number linked to your Voice API application as explained in the Set up your Vonage Voice API application credentials and phone number section as it will be needed for the next section Interact via voice with the Lex bot.

## Interact Via Voice With the Lex Bot

* Call the phone number linked to your Voice API application to interact with the Lex bot.
* You will see on the Lex reference connection application console as well as the Lex sample voice API application console, the transcriptions, and sentiment analysis results.

## Improving the Lex Bot

When calling the Lex bot using the linked phone number, you may have noticed that the bot does not play a greeting, so any caller would be unsure when to start to speak once connected to the bot. Additionally, the bot does not play a confirmation message after you verbally confirmed that you want the appointment.

Let’s improve our example Lex bot by addressing both of those issues.

### Get the Lex Bot to Play a Greeting

On your AWS console, go to Amazon Lex, then your Lex bot. Click on the \[+] icon next to Intents.

![The Lex Dashboard](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-a.png "The Lex Dashboard")

Next, click on **Create Intent**.

![Creating an intent for a Lex bot](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-b.png "Creating an intent for a Lex bot")

You'll be prompted to enter a name for your new intent. Once you've chosen one you can click on **Add** to proceed to the next step.

![Entering a name for the intent](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-c.png "Entering a name for the intent")

Under **Sample utterances**:

Enter exactly `Good morning`, as that is what is currently coded in the Lex sample Voice API application as TTS played to Lex bot when the WebSocket is established,

Then click on the \[+] icon to add the new utterance.

![Adding new utterances to a Lex bot](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-d.png "Adding new utterances to a Lex bot")

Next, under **Response** click on **Add Message**.

![Adding a new message to the bot responder](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-f.png "Adding a new message to the bot responder")

Enter one or more greeting messages. The content of the message, and how many you add, are completely up to you e.g. *Hi, how may I help you?*, or *Hello, what is the reason you are calling?*

Do not forget to click on the \[+] icon for each entered message.

![Adding a message response to a Lex bot](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-g.png "Adding a message response to a Lex bot")

After adding all the messages you want to add, click on **Save Intent**.

![Listing message responses](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-h.png "Listing message responses")

Next, you will need to rebuild the bot. You can do this by clicking on **Build**. After the build completes, go ahead and **Publish** your bot one more time as you did before.

You will be asked to enter an alias once again, you can use the same alias as you did earlier for the sample bot.

![Entering 'staging' as the alias for the Lex bot](/content/blog/connecting-voice-calls-to-an-amazon-lex-bot/screen-1-n.png "Entering 'staging' as the alias for the Lex bot")

## Testing the Update to the Lex Bot

Call the phone number you have linked to the application from the previous steps and you will hear “Please wait”, that TTS from the Vonage API platform is played as soon as your call is answered by the platform.

Once the WebSocket is set up, the Vonage API platform plays the TTS “Good morning” to the Lex bot (you as the caller will not hear that).

The Lex bot plays one of the response messages (of the Greeting intent), you hear it over the phone, confirming the connection with the Lex bot.

You may then ask to set up an appointment (with this example Lex bot).

## Play an Appointment Confirmation Message