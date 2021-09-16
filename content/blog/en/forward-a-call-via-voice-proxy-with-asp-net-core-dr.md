---
title: Forward a Call via Voice Proxy with ASP.NET Core
description: Learn how to forward a call using the voice proxy technique with
  Nexmo APis within your ASP.NET apps
thumbnail: /content/blog/forward-a-call-via-voice-proxy-with-asp-net-core-dr/voice-proxy-asp-net.png
author: bibi
published: true
published_at: 2019-04-18T09:24:18.000Z
updated_at: 2021-05-13T19:43:17.502Z
category: tutorial
tags:
  - voice-api
  - ASP.NET
  - voice-proxy
comments: true
redirect: ""
canonical: ""
---

<em>This is the fifth tutorial on how to use Voice APIs with ASP.NET series.</em>

In the previous tutorial, we learnt [how to handle user input with ASP.NET Core](https://www.nexmo.com/blog/2019/01/10/how-to-handle-user-input-with-asp-net-core-dr/).

Nowadays, we use a lot of apps and services that require us to communicate with another party, usually a stranger, via phone calls or messages. Think about food delivery or taxi booking apps.

One way to protect both parties is to mask their phone numbers by using an intermediary number. This is known as Voice Proxy.  

Let's see how we can implement this technique using [Nexmo Voice APIs](https://developer.nexmo.com/voice/voice-api/overview).

<h2>Learning objectives</h2>

In this tutorial, we will:

* Create an ASP.NET Core app.

* Use NancyFX with ASP.NET Core.

* Create a Nexmo voice application.

* Create and return NCCOs.

* Run and test the code using Ngrok.

<h2>Prerequisites</h2>

<sign-up></sign-up>

* Visual Studio 2017 or higher.

* A Nexmo account, which you can [sign up for here](https://dashboard.nexmo.com/sign-up).

* A project setup for this tutorial series, which you can find on [Github](https://github.com/nexmo-community/nexmo-dotnet-quickstart/tree/ASPNET/NexmoDotNetQuickStarts).

* Optional: [The Nexmo CLI](https://github.com/Nexmo/nexmo-cli).

<h2>Configuration</h2>

To use [The Nexmo Voice API](https://developer.nexmo.com/voice/voice-api/overview), we need to create [a voice application](https://developer.nexmo.com/concepts/guides/applications). 
The configuration steps are detailed in [the “Nexmo Voice API with ASP.NET: Before you start” post (https://www.nexmo.com/blog/2017/07/28/nexmo-voice-api-asp-net-configure-dr/).

Once the configuration is done successfully, we are ready to forward a call.

<h2>Forwarding a Phone Call</h2>

When a call is received, the Nexmo Voice API will make a request to your application to figure out how to respond.

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

The next step is to create a Nancy module in which we set up a route to `/webhook/answer` which will respond with the `ncco` returned by `GetConnectNCCO()`

```csharp
using Nancy;
using Newtonsoft.Json.Linq;

namespace NexmoVoiceASPNetCoreQuickStarts
{
    public class VoiceModule : NancyModule
    {
        public VoiceModule()
        {
            Get["/webhook/answer"] = x => { var response = (Response)GetConnectNCCO();
                                            response.ContentType = "application/json";
                                            return response;
                                          };
        }

        private string GetConnectNCCO()
        {
            dynamic Endpoint = new JObject();
            Endpoint.number = "TO_NUMBER";
            Endpoint.type = "phone";

            dynamic ConnectNCCO = new JObject();
            ConnectNCCO.action = "connect";
            ConnectNCCO.from = "NEXMO_NUMBER";
            ConnectNCCO.endpoint = new JArray(Endpoint);

            JArray NCCObj = new JArray();
            NCCObj.Add(ConnectNCCO);

            return NCCObj.ToString();

        }
    }
}
```

The above code will do the following:

When a call is received, Nexmo will mask the original caller's number and instead uses a virtual number as a facade to this phone call.

We are done! To test this sample app, some more configuration steps are required.

If you've been following so far, you've already configured your Nexmo account and created a voice app as shown in [this post](https://www.nexmo.com/blog/2017/07/28/nexmo-voice-api-asp-net-configure-dr/). We need to link this app to a the Nexmo phone number that we are going to call. If you don't have a number, you can purchase one [using the dashboard](https://dashboard.nexmo.com/buy-numbers) or the CLI:

```javascript
nexmo number:buy --country_code US
```

Similarly to link the number, you can [use the dashboard](https://dashboard.nexmo.com/your-numbers) or the CLI:

```javascript
nexmo link:app NEXMO_PHONE_NUMBER NEXMO_APP_ID
```

We need to tell Nexmo which URL to make a request to when a call is received - this is called the `answer_url`. For me, this url is [http://localhost:63286/webhook/answer](http://localhost:63286/webhook/answer) and that's only running locally.
To expose our webhook answer url, we will use [Ngrok](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/).

```csharp
ngrok http 63286 
```

We now have a new url (http://userSubdomain.ngrok.io) that can be used as the `answer_url` for the voice application.
Update your application with your new `answer_url`. It should look like `http://userSubdomain.ngrok.io/webhook/answer`

Tada! Run the app and give it a go by calling the TO_NUMBER; you will notice that on the other side, you won't see the phone number you're using but instead your NEXMO_NUMBER.

<h2>Learn More</h2>

<h3>API References and Tools</h3>

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

* [Getting Started with Nexmo Number Insight APIs and ASP.NET](https://www.nexmo.com/blog/2018/05/22/getting-started-with-nexmo-number-insight-apis-and-asp-net-dr/).