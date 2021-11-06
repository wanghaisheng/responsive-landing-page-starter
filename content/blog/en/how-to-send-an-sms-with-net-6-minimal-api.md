---
title: How to Send an SMS with .Net 6 Minimal API
description: This tutorial will show you how to send an SMS with .Net 6 and
  Minimal API using the Vonage API.
author: matt-hunt
published: false
published_at: 2021-11-06T17:57:46.503Z
updated_at: 2021-11-06T17:57:46.531Z
category: tutorial
tags:
  - dotnet
  - sms-api
  - minimal-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
I’ve been looking at the preview releases of .Net 6 for some time now and one of the exciting features that a lot of people have been talking about is Minimal APIs. While it seems that opinion is very divided, I think they are a welcome addition and it removes a lot of the weight of using ASP.NET MVC and certainly lowers the bar to entry with a similar feel to ExpressJS in NodeJS.

A little over a year ago we released a blog post entitled “[How to Send an SMS With ASP.NET Core MVC](https://learn.vonage.com/blog/2020/07/09/how-to-send-an-sms-with-asp-net-core-mvc)” so with the Release of .Net 6 upon us I thought it would be a good idea to take inspiration and see what it would take to use the new Minimal API syntax to emulate the same functionality.

## Just give me code

You can skip straight to the code on GitHub

## Prerequisites

* .Net 6 RC 2 SDK
* Visual Studio 2022 Preview or Visual Studio Code

<sign-up></sign-up>

## Creating the project

The easiest way I’ve found to create a new Minimal API project is using the command below

```
dotnet new web -o SmsDotnetMinimalApi
```

Microsoft also have a [great tutorial](https://docs.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-6.0&tabs=visual-studio) on creating a new Minimal API project using Visual Studio.

We should now have an API with one endpoint, to this we will add two nuget packages, the first is Vonage’s SDK version 5.9.2 at the time of writing. As this will be an API we won’t have a UI so the second is Swashbuckle / Swagger that will enable us to try out any endpoints easily.

```
dotnet add package Vonage
dotnet add package Swashbuckle.AspNetCore
```

## Small is Beautiful

Along with the usual `appsettings.json` your newly created project will be just one file, `Program.cs`, this is truly minimal, for an ASP.Net project at least.

![]()

Let’s open `Program.cs`, it should look like this.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

This is all you need to have a fully-fledged .NET API all in a single file. This will provide a much lighter starting point to build a small API or microservice. And to be honest still blows my mind when I think back to how much code WebAPI would need to produce the same outcome.

## Configuration

Let’s make a start by adding some settings. Inside the `appsettings.json` file we need to add our Vonage key and secret, these are used to authenticate your application with Vonage’s services and can be found at the top of the Dashboard.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "Vonage_key": "ab12c3de",
  "Vonage_secret": "ZKSQ1vlzNvyZnQCI"
}
```

Currently, we do not have Dependency Injection configured so let’s add the VonageClient class to the services collection. This will allow it to be injected into any class or method that we will use further down the line.

Import a couple of namespaces at the top of the file.

```csharp
using Vonage;
using Vonage.Messaging;
using Vonage.Request;
```

Register the VonageClient with the services collection.

```csharp
builder.Services.AddSingleton<VonageClient>(provider =>
{
    var config = provider.GetRequiredService<IConfiguration>();
    var key = config.GetValue<string>("Vonage_key");
    var secret = config.GetValue<string>("Vonage_Secret");
    var credentials = Credentials.FromApiKeyAndSecret(key, secret);

    return new VonageClient(credentials);
});
```

Going line at a time we can see that we get an instance of IConfiguration, this enables us to access the app settings we need. The key and secret are then retrieved and so that we can create credentials that are required by the VonageClient constructor.

## Sending Out an SMS

Now that we have our VonageClient class configured and ready to be injected let us create a new endpoint that we can send requests. We need to pass in a data model to the endpoint so create a class in a new file called \`SmsModel\`. Then inside the class we want to add the following properties.

```csharp
public class SmsModel
{
    public string To { get;set; }

    public string From { get;set; }

    public string Text { get;set; }
}
```

With our model created we can go ahead and add a new POST method with the VonageClient, our SmsModel and