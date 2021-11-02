---
title: "Restaurant Is Now Delivering: a Facebook Bot in Node.js"
description: "This javascript tutorial shows how to code a facebook bot which
  tells a user whether a restaurant is currently delivering. Using Vonage API
  and Node.js. "
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

Often when I'm coding I get hungry. And everyone knows the best developers are lazy, so instead of shopping and cooking *and* cleaning, I often use a food delivery app and simply order a tasty meal. The problem is that too often my favorite restaurants are offline. Sometimes they are closed for business; other times they are too busy and stop accepting online orders. So I'm forced to wait and remember to check if they're back online, and then actually open the app and look to see if they're back online. And sometimes check again, and again, and again. It's truly a grave injustice 😆.

There must be a better, smarter way! Thankfully [I recently discovered my favorite food delivery app, Wolt, has an API](https://medium.com/analytics-vidhya/exploring-the-api-of-a-website-8579b04df28f) that lets me know if a restaurant is online. So using the Vonage Messages API, I created a Facebook Messenger Bot that will alert me when my favorite restaurant is back online!

*(This example is built around a use case of food delivery, but the same code can be repurposed to build a Facebook Bot that will alert users for any boolean case change.)*

## Prerequisites

This app requires the following:

* [Node.js](https://nodejs.org/en/)

<sign-up></sign-up>

## Pseudo Code:

Before I get started with any coding task, I like to think out the logic. Let's breakdown the steps needed to build this app:

1. Setup an Express Server
2. Connect to Vonage Messages API Sandbox
3. Call the Wolt API for a requested restaurant
4. Check if the received restaurant is online
5. Send a message to the user based on the restaurant status
6. If the restaurant is offline, continue checking

## Setup our Project

### Create a Node Application

Let's begin will by creating our project:

```javascript
$ mkdir isItDelivering
```

Then move inside the project directory:

```javascript
$ cd isItDelivering
```

Initialize the node project:

```javascript
$ npm init
```

Install our required Node packages:

```javascript
$ npm install @vonage/server-sdk@beta express dotenv got -s
```

And finally create the files where our code will live:

```javascript
$ touch index.js .env
```

You'll notice that we are using the Vonage [Node Server SDK](https://github.com/vonage/vonage-node-sdk) to access the [Messages API](https://developer.vonage.com/messages/overview). Because the Messages API is currently in Beta, we require the Beta version of our SDK.

To set up our server, we're going to need some information from the [Vonage Developer Dashboard](https://dashboard.nexmo.com/). First, we'll create a [new Vonage Application](https://dashboard.nexmo.com/applications/new). Give it a nice name like isItDelivering. And then click "Generate public and private key".

![Generate Public/Private Key](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-1-25-.png "Generate Public/Private Key")

This will automatically generate a key for authentication which we will use later. Move the generated key into the root of your local project.  

At this point, your project should contain your index file, node modules, package.json, and your ENV file. IF you run the command `ls` your project should look like this:

![Project Should Include index.js, node_modules, pack.json, private.key](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-14.35.44.png "Project Should Include index.js, node_modules, pack.json, private.key")

As you can see our Vonage Application allows us to turn on/off various capabilities through the different Vonage APIs. We'll want to switch on the Messages capabilities. We'll now be asked for two URLs corresponding to webhooks that the Messages API will use to interact with our bot application.

## Connect to the Outside World

### Setup ngrok

There are several ways to make our local development server externally accessible, but one of the simplest ways is with ngrok. You can read [this article](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/) for a more detailed explanation of how ngrok works.

For our purposes, we just need to get it running and copy the URL that it provides us.

After having ngrok installed on your machine, we'll need to start it. In order to start ngrok, open up a new terminal window and execute the following from the command line:

```javascript
$ ngrok http 3000
```

You will now see a ngrok logging interface in your terminal window. Near the top of the interface is a line that begins with `Forwarding` and contains two URLs. The first is the externally accessible ngrok URL, which ends with `ngrok.io` followed by `http://localhost:3000`, that being your local development server. Now, when you or Vonage contacts the `ngrok.io` URL, it will forward it to your local server.

Now in our Vonage Dashboard we will add our ngrok URLs and add the appropriate URL routes. Once your URLs look like this you can hit the "Generate new application" button.

![Webook URLs](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-14.40.42.png "Webhook URLs")

## Get Connected with Vonage

### Connect Your Vonage Account

In your ENV file in your project, you will need to add 3 environment variables; `API_KEY` , `API_SECRET`, and `APP_ID`.  

You can find your `API_KEY` and `API_SECRET` in the home page of your Vonage Dashboard:

![Dashboard ENV Variables](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-4-6-.png "Dashboard ENV Variables")

Your APP_ID is found on the configuration page for the application you generated. You will find your application under `Your Applications` in the left-hand navigation bar. Your APP_ID will look like this:

![APP_ID in Dashboard](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-5-7-.png "APP_ID in Dashboard")

Once you've copy/pasted those into your project, your ENV file should look something like this:

```javascript
API_KEY="XXXXXXXXX"
API_SECRET="XXXXXXXXX"
APP_ID="XXXXXXXXX"
```

## Getting Started with Messages API Sandbox

### Adding Users to Your Sandbox

We’ll be making use of the [Vonage Facebook Sandbox](https://developer.nexmo.com/messages/concepts/messages-api-sandbox). You can find the Sandbox in your Vonage Dashboard under Messages and Dispatch tab on the left-hand side, or click [here](https://dashboard.nexmo.com/messages/sandbox). Once you click `Add to Sandbox` for the Facebook Messenger tab, your screen should look like this:

![Set up your sandbox](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/group-6-10-.png "Set up your sandbox")

The Messages API Sandbox allows for quick testing of applications without having to wait for Business Account Approval. The Sandbox uses a whitelist approach for allowing test users. You can invite additional users to the whitelist through the `Send invite email button` or by sending them the hyperlink `click this link`. The link will open a Facebook Messenger session. The user will then need to send the passphrase to be added to the whitelist. Full details can be found [here](https://developer.nexmo.com/messages/concepts/messages-api-sandbox#approve-your-facebook-messenger-recipient-id).

### Connecting Your Application to Your Sandbox

We will now need to tell our Sandbox to listen to requests from our Application and deliver them to Facebook Messenger. This is done through our ngrok URLs. We'll need to add the same ngrok URLs from before, like this:

![Messages API Sandbox ngrok URLS](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-14.57.37.png "Messages API Sandbox ngrok URLS")

Once we hit the `Save webhooks` button, we're now complete with our setup and we can start to code!

## Setting Up an Express Server

### Build a Boilerplate Server with Dependencies

First, let's set up a boilerplate Express server that will import our required libraries and simply run on port 3000:

```javascript
// access our environment variables
require('dotenv').config();
// access the Vonage SDK so we can use the Voange object and API
const Vonage = require('@vonage/server-sdk');
// access Got library which allows us to make HTTP request to WOLT API
const got = require('got');

// boilerplate Express setup
const express = require('express');
const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.listen(3000);
```

## Sending Basic Facebook Message

### Sending A Facebook Message From Our Application

We need to initialize a Vonage instance, passing our ENV variables, and then tell it to use the Vonage Sandbox as the host used for making the HTTP requests.

```
// initialize a new Vonage instance, with ENV variables/keys
const vonage = new Vonage(
  {
    apiKey: process.env.API_KEY,
    apiSecret: process.env.API_SECRET,
    applicationId: process.env.APP_ID,
    privateKey: './private.key'
  },
  {
    apiHost: 'https://messages-sandbox.nexmo.com/',
  }
 );
```

Next, we actually use that Vonage object to send out a POST request on our `/inbound` route and we need to provide to minimal parameters: `type` and `text`.

```javascript
// Basic Sandbox Messaging
app.post('/inbound', (req, res) => {
  vonage.channel.send(
    req.body.from,
    req.body.to,
    {
      content: {
        type: 'text',
        text: 'You must be hungry! 🍕'
      },
    },
    (err, data) => {
      if(err){
        console.log(err);
      } else{
          console.log(data.message_uuid);
      }
    }
  );
  res.send('ok');
});

app.post('/status', (req, res) => {
  res.send('ok');
});
```

So now in a second terminal window, separate from our ngrok server, we just need to fire up our Express Server:

```javascript
$ node index.js
```

And we can interact with our Facebook Bot!

![Basic Facebook SandBox Message](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/ezgif.com-gif-maker-8-.gif "Basic Facebook SandBox Message")

## Receiving Restaurant Information From Wolt API

### Making HTTP Request

Using the `https://restaurant-api.wolt.com/v3/venues/slug/{restaurant}` endpoint, we know that we can receive all kinds of information about the restaurant. The returned JSON looks like this:

![Wolt Returned JSON](/content/blog/restaurant-is-now-delivering-a-facebook-bot-in-node-js/screen-shot-2021-09-22-at-15.44.41.png "Wolt Returned JSON")

We can see that inside index zero, there is a property called `name` of type array. At the zero index of `name` is a boolean called `online`, which gives the current delivery status of the restaurant. So we can create a function that takes the name of a restaurant and returns the restaurant object from Wolt:

```javascript
// call Wolt API for restaurant info
const getRestaurant = async (reqRestaurant) => {
  const response = await got.get(`https://restaurant-api.wolt.com/v3/venues/slug/${reqRestaurant}`)
      .json();
  return response.results[0];
}
```

## Check if the received restaurant is online

Using the property `online` inside the `restaurant` object, we want to create some logic that will determine what message we send to the user. We can write the following function:

```javascript
const sendStatusMessage = (restaurant, req, rest) => {
  if (restaurant.online) {
    sendFacebookMessage(`Hey, ${restaurant.name[0].value} is now accepting orders!!`, req, res);
  } else {
      sendFacebookMessage(`Sorry, ${restaurant.name[0].value} is currently offline. I'll ping you when it's open again!`, req, res);
    }
}
```

The `sendStatusMessage` function has abstracted our Vonage code into a function called `sendFacebookMessage` :

```javascript
const sendFacebookMessage = async (text, req, res) => {
  vonage.channel.send(
    req.body.from,
    req.body.to,
    {
      content: {
        type: 'text',
        text: text,
      },
    },
    (err, data) => {
      if (err) {
        console.log(err);
      } else {
        console.log(data.message_uuid);
      }
     }
   );
 }
```

## Send a message to the user based on the restaurant status

And now combining our new functionality we can update our simple Sandbox Messaging to tell the user whether the requested restaurant is online or not.

```javascript
// Enhanced Sandbox Messaging
app.post('/inbound', async(req, res) => {
  const requestedRestaurant = await req.body.message.content.text.split('/').pop();
  const restaurant = await getRestaurant(requestedRestaurant);
  sendStatusMessage(restaurant, req, res);
  res.send('ok');
});
```

## Loop if the Restaurant is Offline

Now that we've created the logic based on the restaurant's status, we want to continue to check that status until finally, the restaurant comes back online. So we know we need a function like this, which check the status every minute:

```javascript
while(RESTAURANT_IS_OFFLINE.status){
 setInterval(await theLoop(), 600000);
}
```

Now we know we need to create some sort of `RESTAURANT_IS_OFFLINE` global variable to store our restaurant information and we know we'll need to write a function called `theLoop` which will run our code every 60 seconds.

*The reason we need to store our restaurant information in a global variable is to keep track of the number of times we've checked its status. We want to only send an offline message to the user the first time. Imagine how annoying it would be for the user to receive an update every 60 seconds that the restaurant is still offline 😬.*

So we'll add the global variable `RESTAURANT_IS_OFFLINE` with properties `status`, `count`, and `name`.

```javascript
let RESTAURANT_IS_OFFLINE = {
  status: true,
  count: 0,
  name: ""
};
```

We'll now need to update our `sendStatusMessage`. The new function will continue to check a restaurant based on its delivery status. However, it will now only send the offline status message if this is the first time its status is reported as offline.

```javascript
const sendStatusMessage = (restaurant, req, res) => {
  if (restaurant.online) {
    RESTAURANT_IS_OFFLINE.status = false;
    sendFacebookMessage(`Hey, ${restaurant.name[0].name.value} is now accepting orders!!`, req, res);
  } else {
    if (RESTAURANT_IS_OFFLINE.count == 0){
        sendFacebookMessage(`Sorry, ${restaurant.name[0].value} is currently offline. I'll ping you when it's open again!`, req, res);
    }
    RESTAURANT_IS_OFFLINE = {status: true, count: 1};
  }
}
```

And now we can complete our `theLoop` function:

```javascript
const theLoop = async () => {
  const restaurant = away getRestaurant(RESTAURANT_IS_OFFLINE.name);
  const status = restaurant.online;
  sendStatusMessage(status);
}
```

And now we can run the program, seeing that when a restaurant is offline we receive a message notifying so, and when restaurants go online we are updated to the new status. I suggest trying the app in the morning and seeing as restaurants suddenly open for lunch. It's very fun to get the push notifications from Facebook Messenger to arrive on your phone!

## Some Code Clean Up

Although the code above works, it will act erratically. This is because the intervalID which is returned from setInterval() is never cleared. ([Read more here](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)) We can do a few things to clean up our code. And we'll create the code which will also allow us to keep track of multiple requests and implement jobs in the future.

First, let's modify our `RESTAURANT_IS_OFFLINE` global variable. We've been keeping track of quite a few pieces of information; the current requested restaurant's name, its online status, and whether we should ping the user. Additionally, we'll now keep track of our `intervalId`. To make our code easier to adjust, say we want to check every 30 seconds, we'll move our timeout limit into its own variable as well. So we can decouple our `RESTAURANT_IS_OFFLINE` variable into:

```javascript
const TIMEOUT = 600000;

let intervalId;
let shouldPing = true;
let CURRENT_RESTAURANT = {};
```

But where's the name or online status? We'll move that into a new function called `setNewCurrentRestaurant`:

```javascript
const setNewCurrentRestaurant = (restaurant) => {
	CURRENT_RESTAURANT = {
		name: restaurant.name[0].value,
		isOnline: restaurant.online,
	};
}
```

And we update our `sendStatusMessage` and `theLoop` functions accordingly: 

```javascript
const sendStatusMessage = (req) => {
	if (CURRENT_RESTAURANT.isOnline) {
		sendFacebookMessage(`Hey, ${CURRENT_RESTAURANT.name} is now accepting orders!!`, req);
	} else if (shouldPing) {
		sendFacebookMessage(`Sorry, ${CURRENT_RESTAURANT.name} is currently offline. I'll ping you when it's open again!`, req);
		shouldPing = false;
	}
}

const theLoop = async (req) => {
	const restaurant = await getRestaurant(CURRENT_RESTAURANT.name);
	setNewCurrentRestaurant(restaurant);
	sendStatusMessage(req);
	if (CURRENT_RESTAURANT.isOnline) {
		clearInterval(intervalId);
		intervalId = null;
	}
}
```

And lastly, our updated request looks like this:

```javascript
app.post('/inbound', async (req, res) => {
	const requestedRestaurant = await req.body.message.content.text.split('/').pop();
	const restaurant = await getRestaurant(requestedRestaurant);
	setNewCurrentRestaurant(restaurant);

	// When there is a new restaurant request, we set should Ping to true
	shouldPing = true;
	sendStatusMessage(req);

	// Start interval only if we havent yet started one
	if (!intervalId) {
		// Don't need to pass the entire req but it is simpler this way i believe
		intervalId = setInterval(await theLoop(req), TIMEOUT);
	}

	res.send('ok');
});
```

Our final `index.js` file will look like this:

```javascript
require('dotenv').config();
const Vonage = require('@vonage/server-sdk');
const got = require('got');


const express = require('express');
const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

const TIMEOUT = 600000;

let intervalId;
let shouldPing = true;
let CURRENT_RESTAURANT = {};

const vonage = new Vonage(
  {
    apiKey: process.env.API_KEY,
    apiSecret: process.env.API_SECRET,
    applicationId: process.env.APP_ID,
    privateKey: './private.key'
  },
  { apiHost: 'https://messages-sandbox.nexmo.com/'}
 );

// store current search status
const setNewCurrentRestaurant = (restaurant) => {
		CURRENT_RESTAURANT =  {
			name: restaurant.name[0].value, 
			isOnline: restaurant.online,  
		};
}

const getRestaurant = async (reqRestaurant) => {
	const response = await got.get(`https://restaurant-api.wolt.com/v3/venues/slug/${reqRestaurant}`)
		.json();
	return response.results[0];
}

const sendFacebookMessage = async (text, req, res) => {
	vonage.channel.send(
		req.body.from,
		req.body.to,
		{
			content: {
				type: 'text',
				text: text,
			},
		},
		(err, data) => {
			if (err) {
				console.error(err);
			} else {
				console.log(data.message_uuid);
			}
		}
	);
}


const sendStatusMessage = (req) => {
	if (CURRENT_RESTAURANT.isOnline) {
		sendFacebookMessage(`Hey, ${CURRENT_RESTAURANT.name} is now accepting orders!!`, req);
	} else if (shouldPing) {
		sendFacebookMessage(`Sorry, ${CURRENT_RESTAURANT.name} is currently offline. I'll ping you when it's open again!`, req);
		shouldPing = false;
	}
}


const theLoop = async (req) => {
	const restaurant = await getRestaurant(CURRENT_RESTAURANT.name);
	setNewCurrentRestaurant(restaurant);
	sendStatusMessage(req);
	if (CURRENT_RESTAURANT.isOnline) {
		clearInterval(intervalId);
		intervalId = null;
	}
}

app.post('/inbound', async (req, res) => {
	const requestedRestaurant = await req.body.message.content.text.split('/').pop();
	const restaurant = await getRestaurant(requestedRestaurant);
	setNewCurrentRestaurant(restaurant);
	shouldPing = true;
	sendStatusMessage(req);
	if (!intervalId) {
		intervalId = setInterval(await theLoop(req), TIMEOUT);
	}
	res.send('ok');
});

app.listen(3000);
```

# What's Next

* In this tutorial, we used the Facebook Messenger functionality of the Messages API but we could extend this application to provide omnichannel capabilities with WhatsApp and SMS. Imagine a very urgent use case (I have a particular bagel shop, on Saturday mornings in mind) that you would want to know immediately a status change, omnichannel alerts would be useful.
* We could extend this code to make the alerts smarter based on delivery schedules, user proximity to restaurants, and more. We could also hold multiple jobs.
* We could take the app out of the Sandbox and connect it to a business Facebook Account.

The final code for the tutorial can be found on GitHub.\
\
I would love to hear what you built using the Vonage Messages API! Please join the conversation on our [Community Slack](https://developer.nexmo.com/community/slack) and share your story!