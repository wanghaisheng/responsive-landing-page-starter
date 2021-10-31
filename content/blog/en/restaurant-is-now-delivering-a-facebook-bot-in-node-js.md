---
title: "Restaurant Is Now Delivering: a Facebook Bot in Node.js"
description: This tutorial is about the blah blah blah
author: benjamin-aronov
published: true
published_at: 2021-10-13T16:57:51.724Z
updated_at: 2021-10-13T16:57:51.751Z
category: tutorial
tags:
  - messages-api
  - node
  - javascript
comments: false
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Often times when I'm coding I get hungry. And everyone knows the best developers are lazy so instead of shopping and cooking and cleaning, instead I often use a food delivery app and just order a tasty meal. The problem is that too often my favorite restaurants are offline. Sometimes they are closed for business or other times they are too busy and stopped accepting online orders. So I'm forced to wait, and remember to check if they're back online, and then actually open the app and check if they're back online. And sometimes check again, and again, and again. It's truly a grave injustice 😆.

There's got to be a better, smarter way! Thankfully [I recently discovered my favorite food delivery app, Wolt, has an API](https://medium.com/analytics-vidhya/exploring-the-api-of-a-website-8579b04df28f) that let's me know if a restaurant is online. So using the Vonage Messages API, I created Facebook Messenger Bot that will alert me when my favorite restaurant is back online!

*(This example is built around a use case of food delivery but the same code can be repurposed to build a Facebook Bot that will alert users for any boolean case change.)*

## Pseudo Code:

Before I get started with any coding task, I like to think out the logic. Let's breakdown the steps needed to build this app:

1. Setup an Express Server
2. Connect to Vonage Messages API Sandbox
3. Call the Wolt API for a requested restaurant
4. Check if the received restaurant is online
5. Send a message to the user based on the restaurant status
6. Loop if the restaurant is offline

## Setup our Project

### Create a Node Application

Let's begin will by creating our project:

`mkdir isItDelivering`

Then move inside the project directory:

`cd isItDelivering`

Initialize the node project:

`npm init`

Install our required Node packages:

`npm install @vonage/server-sdk@beta express dotenv got -s`

And finally create the files where our code will live:

`touch index.js .env`

You'll notice that we are using the Vonage [Node Server SDK](https://github.com/vonage/vonage-node-sdk) to access the[ Messages API](https://developer.vonage.com/messages/overview). Because the Messages API is currently in Beta, we require the Beta version of our SDK.

In order to setup our server, we're going to need some information from the [Vonage Developer Dashboard](https://dashboard.nexmo.com/). First we'll create a [new Vonage Application](https://dashboard.nexmo.com/applications/new). Give it a nice name like isItDelivering. And then click "Generate public and private key".

![Generate Public/Private Key](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-1-25-.png "Generate Public/Private Key")

This will automatically generate a key for authentication which we will use later. Movie the generated key into the root of your local project. \
\
\
At this point your project should contain your index file, node modules, package.json, and your ENV file. IF you  run the command `ls` your project should look like this:

![Project Should Include index.js, node_modules, pack.json, private.key](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-14.35.44.png "Project Should Include index.js, node_modules, pack.json, private.key")

## Connect to the Outside World

### Setup ngrok

There are several ways to make our local development server externally accessible, but one of the simplest ways is with ngrok. You can read [this article](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/) for a more detailed explanation of how ngrok works.

However, for our purposes, we just need to get it running and copy the URL that it provides us.

In order to start ngrok, open up a new terminal window and execute the following from the command line:

```
$ ngrok http 3000
```

You will now see a ngrok logging interface in your terminal window. Near the top of the interface is a line that begins with `Forwarding` and contains two URLs. The first is the externally accessible ngrok URL, which ends with `ngrok.io` followed by `http://localhost:3000`, that being your local development server. Now, when you or Vonage contacts the `ngrok.io` URL, it will forward it to your local server.

Now in our Vonage Dashboard we will add our ngrok URLs and add the appropriate URL routes. Once your URLs look like this you can hit the "Generate new application" button.

![Webook URLs](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-14.40.42.png "Webhook URLs")

## Get Connected with Vonage

### Connect Your Vonage Account

In your ENV file in your project, you will need to add 3 environment variables; `API_KEY` , `API_SECRET`, and `APP_ID`. \
\
You can find your `API_KEY` and `API_SECRET` in the home page of your Vonage Dashboard:

![Dashboard ENV Variables](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-4-6-.png "Dashboard ENV Variables")

Your APP_ID is found in the configuration page for the application you generated. You will find your application under `Your Applications` in the left hand navigation bar. Your APP_ID will look like this:

![APP_ID in Dashboard](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-5-7-.png "APP_ID in Dashboard")

Once you've copy/pasted those into your project, your ENV file should look something like this:

```
API_KEY="XXXXXXXXX"
API_SECRET="XXXXXXXXX"
APP_ID="XXXXXXXXX"
```

## Getting Started with Messages API Sandbox

### Adding Users to Your Sandbox

We’ll be making use of the [Vonage Facebook Sandbox](https://developer.nexmo.com/messages/concepts/messages-api-sandbox). You can find the Sandbox in your Vonage Dashboard under Messages and Dispatch tab on the left-hand side, or click [here](https://dashboard.nexmo.com/messages/sandbox).  Once you click `Add to Sandbox` for the Facebook Messenger tab, your screen should look like this:

![](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-6-10-.png)

The Messages API Sandbox allows for quick testing of applications without having to wait for Business Account Approval. The Sandbox uses a whitelist approach for allowing test users. You can invite additional users to the whitelist through the `Send invite email button` or by sending them the hyperlink `click this link`. The link will open a Facebook Messenger session. The user will then need to send the passphrase to be added to the whitelist. Full details can be found [here](https://developer.nexmo.com/messages/concepts/messages-api-sandbox#approve-your-facebook-messenger-recipient-id).

### Connecting Your Application to Your Sandbox

We will now need to tell our Sandbox to listen to requests from our Application and deliver them to Facebook Messenger. This is done through our ngrok URLs. We'll need to add the same ngrok URLs from before, like this:

![Messages API Sandbox ngrok URLS](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-10-31-at-17.07.20.png "Messages API Sandbox ngrok URLS")

## Setup Complete, Let's Code!

## Setting Up an Express Server

### Build a Boilerplate Server with Dependencies

First, let's setup a boilerplate Express server that will import our required libraries and simply run on port 3000:

![Boilerplate Express Server](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.04.45.png "Boilerplate Express Server")

## Sending Basic Facebook Message

### Sending A Facebook Message From Our Application

We need to initialize a Vonage instance, passing our ENV variables and then tell it to use the Vonage Sandbox as the host used for making the HTTP requests.

![Initalize New Vonage Instance](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.06.39.png "Initalize New Vonage Instance")

Next, we actually use that Vonage object to send out a POST request on our `/inbound` route and we need to provide to minimal paramaters: `type` and `text`.

![Basic Sandbox Messaging](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-10-31-at-17.38.05.png "Basic Sandbox Messaging")

So now we just need to fire up our Express Server:

`node index.js`

And we can interact with our Facebook Bot! 

\[enter screen recordign of basic Messaging]

## Receiving Restaurant Information From Wolt API

### Making HTTP Request

Using the `https://restaurant-api.wolt.com/v3/venues/slug/{restaurant}` endpoint, we know that we can receive all kind of information about the restaurant. The returned json looks like this: 

![Wolt Returned JSON](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.44.41.png "Wolt Returned JSON")

We can see that inside index zero, there is a property called `name` of type array. At the zero index of `name` is a boolean called `online`, which gives the current delivery status of the restuarant. So we can create a function that takes the name of a restaurant and returns the restuarant object from Wolt:

![Get Restaurant Function](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.32.54.png "Get Restaurant Function")

## Check if the received restaurant is online

Using the property `online` inside the `restaurant` instance, we want to create some logic will determine what message we send to the user. We can write the following function:

![Send Status Message Function](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.42.32.png "Send Status Message Function")

The `sendStatusMessage` has abstracted our Vonage code into a function called `sendFacebookMessage` :

![sendFacebookMessage Function](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-10-31-at-18.45.01.png "sendFacebookMessage Function")

## Send a message to the user based on the restaurant status

And now combining our new functionality we can update our simple Sandbox Messaging to tell the user whether the requested restaurant is online or not. 

![Enhanced Sandbox Messaging](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-10-31-at-18.23.34.png "Enhanced Sandbox Messaging")

## Loop if the Restaurant is Offline

Now that we've created the logic based on the restaurant's status, we want to continue to check that status until finally the restaurant comes back online. So we know we need a function like this, which check the status every minute:

![Generic Loop Function](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.50.28.png "Generic Loop Function")

Now we know we need to create some sort of `RESTAURANT_IS_OFFLINE` global variable to store our restaurant information and we know we'll need to write a function called `theLoop` which will run our code every 60 seconds. 

*The reason we need to store our restaurant information in a global variable is to keep track of the number of times we've checked it's status. We want to only send an offline message to the user the first time. Imagine how annoying it would be for the user to receive an update every 60 seconds that the restaurant is still offline 😬.*

So we'll add the global variable `RESTAURANT_IS_OFFLINE` with properties `status`, `count`, and `name`. 

![RESTAURANT_IS_OFFLINE Global Variable](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.52.08.png "RESTAURANT_IS_OFFLINE Global Variable")

We'll now need to update our `sendStatusMessage`. The new function will continue to check a restaurant based on its delivery status. However it will now only send the offline status message if this is the first time its status is reported as offline.

![Enhanced sendStatusMessage Function](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.53.33.png "Enhanced sendStatusMessage Function")

And now we can complete our `theLoop` function:

![theLoop Function](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-10-31-at-18.54.42.png "theLoop Function")



And now we can run the program, seeing that when a restaurant is offline we receive a message notifying so, and when restaurants go online we are updated to the new status. I suggest trying the app in the morning and seeing as restaurants suddenly open for lunch. It's very fun to get the push notifications from Facebook Messenger arrive on your phone! 



# What's Next

* In this tutorial, we used the Facebook Messenger functionality of the Messages API but we could extend this application to provide omnichannel capabilities with WhatsApp and SMS. Imagine a very urgent use case (I have a particular bagel shop, on Saturday mornings in mind) that you would want to know immediately a status change, omnichannel alerts would be useful.
* We could extend this code to make the alerts smarter based on delivery schedules, user proximity to restaurants, and more. We could also hold multiple jobs more intelligently.
* We could take the app out of the Sandbox and connect it to a business Facebook Account.

The final code for the tutorial can be found on GitHub.