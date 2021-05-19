---
title: Build a Conference Call with the Nexmo Voice API and ASP.NET Core
description: Step by step guide on how to build a conference call "conversation"
  for multiple participants using ASP.NET Core and Nexmo's Voice API.
thumbnail: /content/blog/build-a-conference-call-with-nexmo-voice-api-and-csharp-dr/csharp-conference-call-1.png
author: bibi
published: true
published_at: 2019-05-16T09:46:00.000Z
updated_at: 2021-05-13T20:43:02.698Z
category: tutorial
tags:
  - dotnet
  - voice-api
  - csharp
comments: true
redirect: ""
canonical: ""
---
This is the sixth tutorial on how to use Voice APIs with ASP.NET series._ In the previous tutorial, we learnt [how to forward a call via voice proxy with ASP.NET Core](https://www.nexmo.com/blog/2019/04/18/forward-a-call-via-voice-proxy-with-asp-net-core-dr/). In today's tutorial, we will learn how to setup a conference call so multiple people can join the same call.

## Learning objectives

In this tutorial, we will:

* Create an ASP.NET Core app.

* Use NancyFX with ASP.NET Core.

* Create a Nexmo voice application.

* Create and return NCCOs.

* Run and test the code using Ngrok.

## Prerequisites


* Visual Studio 2017 or higher.

* A Nexmo account, which you can [sign up for here](https://dashboard.nexmo.com/sign-up).

* A project setup for this tutorial series, which you can find on [Github](https://github.com/nexmo-community/nexmo-dotnet-quickstart/tree/ASPNET/NexmoDotNetQuickStarts).

* Optional: [The Nexmo CLI](https://github.com/Nexmo/nexmo-cli).

<sign-up></sign-up>

## Configuration

To use [The Nexmo Voice API](https://developer.nexmo.com/voice/voice-api/overview), we need to create [a voice application](https://developer.nexmo.com/concepts/guides/applications).

The configuration steps are detailed in the [Nexmo Voice API with ASP.NET: Before you start](https://www.nexmo.com/blog/2017/07/28/nexmo-voice-api-asp-net-configure-dr/) post.

Once the configuration is done successfully, we can move on to setting up a conference call.

## Building a Conference Call

When a user calls the Nexmo number, the Nexmo Voice API will make a request to the application to figure out how to respond using a [Nexmo Call Control Object (NCCO)](https://developer.nexmo.com/voice/voice-api/ncco-reference).

The user will be greeted then will join the conference call.

For that purpose, we are going to use [NancyFX](https://github.com/NancyFx/Nancy) along side our ASP.NET Core project.

First of all, we need to add Nancy to our project :

```csharp
PM> Install-Package Nancy
PM> Install-Package Microsoft.AspNetCore.Owin
```

To allow Nancy to handle any HTTP requests, we need to tell ASP.NET Core to use Nancy via `Owin` in the `Configure` method of `Startup.cs`.

```csharp
using Microsoft.AspNetCore.Builder;
using Nancy.Owin;

namespace NexmoVoiceASPNetCoreQuickStarts
{
    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            app.UseOwin(x => x.UseNancy());
        }
    }
}
```

The next step is to create a Nancy module in which we setup a route to `/webhook/answer` which will respond with the `ncco` returned by `GetConferenceCallNCCO()`

```csharp
using Nancy;
using Newtonsoft.Json.Linq;

namespace NexmoVoiceASPNetCoreQuickStarts.Modules
{
    public class ConferenceCallModule : NancyModule
    {
        public ConferenceCallModule()
        {
            Get["/webhook/answer/"] = x => GetConferenceCallNCCO();
            Post["/webhook/event"] = x => Request.Query["status"];
        }

        private dynamic GetConferenceCallNCCO()
        {
            dynamic TalkNCCO = new JObject();
            TalkNCCO.action = "talk";
            TalkNCCO.text = "Hello. You will now be added to the conference call.";
            TalkNCCO.voiceName = "Emma";

            dynamic ConferenceNCCO = new JObject();
            ConferenceNCCO.action = "conversation";
            ConferenceNCCO.name = "conference-call";

            JArray nccoObj = new JArray
            {
                TalkNCCO,
                ConferenceNCCO
            };

            return nccoObj.ToString();
        }
    }
}
```

The above code will do the following:

When a call is received, the user will hear "Hello. You will now be added to the conference call." then they will be added to the conference call.

Multiple callers can be added to the conference until they all have disconnected.

We are done! To test this sample app, some more configuration steps are required.

<h2>Linking Your App to Nexmo</h2>

If you've been following along so far, you've already configured your Nexmo account and created a voice app as shown in [this post](https://www.nexmo.com/blog/2017/07/28/nexmo-voice-api-asp-net-configure-dr/). We need to link this app to a the Nexmo phone number that we are going to call.

If you don't have a number, you can purchase one [using the dashboard](https://dashboard.nexmo.com/buy-numbers) or the by using the [Nexmo CLI](https://github.com/Nexmo/nexmo-cli):

```bash
nexmo number:buy --country_code US
```

Similarly to link the number, you can [use the dashboard](https://dashboard.nexmo.com/your-numbers) or the CLI:

```bash
nexmo link:app NEXMO_PHONE_NUMBER NEXMO_APP_ID
```

We need to tell Nexmo which URL to make a request to when a call is received - this is called the `answer_url`. For me, this url is [http://localhost:63286/webhook/answer](http://localhost:63286/webhook/answer) and that's only running locally.

To expose our webhook answer url, we will use [Ngrok](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/).

```bash
ngrok http 63286 
```

We now have a new url (mine is http://<SUBDOMAIN>.ngrok.io) that can be used as the `answer_url` for the voice application.

Update your application with your new `answer_url`. It should look like `http://subdomain.ngrok.io/webhook/answer`. Run the app and give it a go by calling the TO_NUMBER.

## Learn more

### API References and Tools

* [Application API](https://developer.nexmo.com/concepts/guides/applications).

* [Voice API](https://developer.nexmo.com/voice/voice-api/overview).

* [Nexmo REST client for .NET](https://github.com/Nexmo/nexmo-dotnet).

<h3>Nexmo Getting Started Guides for ASP.NET</h3>

* [How to Send SMS Messages with ASP.NET](https://www.nexmo.com/blog/2017/03/23/send-sms-messages-asp-net-mvc-framework-dr/).

* [How to Receive SMS Messages with ASP.NET](https://www.nexmo.com/blog/2017/03/31/recieve-sms-messages-with-asp-net-mvc-framework-dr/).

* [How to Get an SMS Delivery Receipt in ASP.NET](https://www.nexmo.com/blog/2017/07/21/get-sms-delivery-receipt-asp-net-mvc-dr/).

* [How to make a Text-to-Speech phone call with ASP.NET](https://www.nexmo.com/blog/2017/07/28/text-to-speech-phone-call-dr/).

* [How to play Audio to a Caller in ASP.NET](https://www.nexmo.com/blog/2017/11/29/how-to-play-audio-to-a-caller-in-asp-net-core-dr/).

* [How to Receive a Phone Call with Nexmo Voice API, ASP.NET Core and NancyFX](https://www.nexmo.com/blog/2018/11/21/how-to-receive-a-phone-call-with-nexmo-voice-api-asp-core-core-and-nancyfx-dr/).

* [how to handle user input with ASP.NET Core](https://www.nexmo.com/blog/2019/01/10/how-to-handle-user-input-with-asp-net-core-dr/)

* [how to forward a call via voice proxy with ASP.NET Core](https://www.nexmo.com/blog/2019/04/18/forward-a-call-via-voice-proxy-with-asp-net-core-dr/)

* [Getting Started with Nexmo Number Insight APIs and ASP.NET](https://www.nexmo.com/blog/2018/05/22/getting-started-with-nexmo-number-insight-apis-and-asp-net-dr/).
