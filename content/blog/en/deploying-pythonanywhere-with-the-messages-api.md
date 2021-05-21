---
title: Deploying PythonAnywhere With the Messages API
description: Learn how to deploy a Python application using Vonage's Messages
  API to PythonAnywhere.
author: greg-holmes
published: true
published_at: 2021-05-21T15:36:36.119Z
updated_at: 2021-05-21T15:36:36.145Z
category: tutorial
tags:
  - messages-api
  - python
comments: true
spotlight: true
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
To make an application available for end-users to view and use, the application has to be deployed. Application or software deployment is one of the most important processes; the application becomes available to the intending users by this process.

Deployment involves the process of moving an application, updates, modules, and so on from the development server to production, where users can access this application via the internet or intranet.

Most software companies and developers deploy applications through a manual or automated process or, most times, both. The processes involved include software and package installation, testing, software release, and deployment for the deployment.

In this tutorial, we will be focusing on deploying Vonage Message API to production on PythonAnywhere. **[Vonage Message API](https://www.vonage.com/communications-apis/messages/)** integrates with SMS, MMS, and popular social chat apps—so users can communicate with your customers on whichever channel they choose. To access the API, you must create an account on the Vonage Developer Dashboard, where you are provided with credit to start testing the API.

First, we will be creating a simple application on PythonAnywhere that sends WhatsApp messages to users, which we will then deploy subsequently.

## Requirements

<sign-up number></sign-up>

* A PythonAnywhere account either free or paid account.
* Python application installed on your system.
* A basic knowledge of Python.

## What Is PythonAnywhere?

PythonAnywhere is a cloud-based development and hosting environment for python applications.

This platform is used for hosting both python web applications and scripts. PythonAnywhere serves as a hosting environment; developers can code, run and host python applications and scripts; these python applications include Django, Flask, web2py, and Bottle applications.

It is free, beginner-friendly, and provides a good number of python extensions installed by default. Another unique feature of PythonAnywhere that it is explicitly built for python projects; therefore, it does not provide empty Linux and Ubuntu servers, unlike other cloud service providers.

## Installing PythonAnywhere

To begin with, if you do not have a PythonAnywhere account, head over to <https://www.pythonanywhere.com/pricing/> and create a "Beginner account", this allows you to create a free account and will be sufficient to carry out this project. Fill the form and create your account.

You must choose a username wisely because the username will serve as your website domain name, i.e. [https://www.your-username.pythonanywhere.com](https://www.your-username.pythonanywhere.com/). Agree to the "Terms and Conditions" and then check your mail for a verification mail.

## Creating a Bash Console

Once you log in to your account, you will have a dashboard with different sections. Click on the "consoles" section and choose your preferred console, between python and bash console; this tutorial will be using the bash console. Click on the "$ bash", and a bash console will open up in your browser window.

![An example of the PythonAnywhere Bash Console](/content/blog/deploying-pythonanywhere-with-the-messages-api/console.png)

For this tutorial, we will be deploying a Whatsapp bot using the Vonage API. We will be creating a simple bot to send WhatsApp messages using the `Flask framework`. This bot will take the user's WhatsApp number and the Vonage number provided. We will be creating a flask file for this and making some imports but before we begin, I will briefly show you how to set up your Vonage account and add your number to the sandbox.

* [Flask](https://palletsprojects.com/p/flask/) is a lightweight python web application framework. Which we could be using to create our application
* [Vonage](https://www.vonage.com/) is an API library which offers flexible voice, messaging, video and Communication APIs

## Vonage Messages API - WhatsApp Channel

Vonage Messages API allows users to send messages to a WhatsApp number via the WhatsApp channel. Firstly, to access Vonage API, create a Vonage API account on the Vonage Developer dashboard. Afterwards, you'll get free credits to use the API and an API Key and API Secret at the top of the Vonage Dashboard. Lastly, to use the WhatsApp Channel, you need to set up a sandbox.

## Setting up Vonage Sandbox

To set up Vonage Message API sandbox, click on Messages and Dispatch, followed by Sandbox. The image below shows an example of the screen you'll see:

![A screenshot of the Vonage dashboard showing various social channels available for the Messages API](/content/blog/deploying-pythonanywhere-with-the-messages-api/sandbox.png)

Under the WhatsApp section, add WhatsApp to the sandbox.  Afterwards, this will be displayed on the screen.

![An example of the sandbox dashboard page with WhatsApp enabled](/content/blog/deploying-pythonanywhere-with-the-messages-api/sandbox_2.png)