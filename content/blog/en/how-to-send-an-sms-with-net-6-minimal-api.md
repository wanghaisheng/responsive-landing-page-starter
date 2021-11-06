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