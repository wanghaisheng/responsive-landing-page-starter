---
title: How to Send SMS Messages with Node.js and Express
description: A step-by-step tutorial on how to send SMS messages with Node.js
  and Express using the Nexmo SMS API and Node.js client library.
thumbnail: /content/blog/how-to-send-sms-messages-with-node-js-and-express-dr/sms-send-node.png
author: tomomi
published: true
published_at: 2016-10-19T16:53:39.000Z
updated_at: 2021-05-17T12:49:00.983Z
category: tutorial
tags:
  - node
  - express
  - sms-api
comments: true
redirect: ""
canonical: ""
---
> This article is now out of date and will soon be removed. Please check out the [updated version](https://learn.vonage.com/blog/2019/09/16/how-to-send-and-receive-sms-messages-with-node-js-and-express-dr) instead.

The [Nexmo SMS API](https://docs.nexmo.com/messaging/sms-api) allows you to send and receive a high volume of SMS anywhere in the world. Once you get your virtual phone number, you can use the API to manage outbound messages ("sending") and inbound messages (“receiving”). In this article, you will learn how to send SMS messages with Node.js and express.

All the step-by-step articles I am going to post in this *Getting Started* series are written from my experiences as a new employee at Nexmo! Whenever I try a new thing, technical or not, I tend to write down how I did it whether I succeed or fail. Working with Nexmo APIs is not an exception - I have been writing down every step I took to work with each API from scratch. Now I am posting my notes with a bunch of screenshots to share with you, so I hope you find them helpful. So let’s walk through with me!

**View** **[the source code on GitHub](https://github.com/nexmo-community/nexmo-node-quickstart/blob/master/sms/send-express.js)**

### Prerequisites

Before starting this tutorial, make sure you have:

* the basic understanding of JavaScript and Node.js
* [Node.js](https://nodejs.org/en/) installed on your machine

<sign-up number></sign-up>

## Using the Nexmo REST API Client for Node.js

First, use npm to install `nexmo`, the REST API client for Node.js in your working directory:

`$ npm install nexmo --save`

Create a `.js` file, let’s call it `index.js`, and in the file, initialize a `Nexmo` instance with your credentials:

```javascript
const Nexmo = require('nexmo');
const nexmo = new Nexmo({
  apiKey: YOUR_API_KEY,
  apiSecret: YOUR_API_SECRET
});
```

There are also optional params such as, `applicationId`, `privateKey`, and `options` object. You can find out more on [the **node-nexmo** library repo on GitHub](https://github.com/Nexmo/nexmo-node).

## Send SMS Messages with Node.js

To send a message, use the `nexmo.sms.sendSms` function and pass your virtual number you are sending the message from, a recipient number, and the message to be sent.

Also, you can pass [optional params](https://docs.nexmo.com/messaging/sms-api/api-reference#request), and a callback.

Let's try hard-code the phone number (which should start with a country code, e.g. "15105551234") and a message for now to try the API:

```javascript
nexmo.message.sendSms(
  YOUR_VIRTUAL_NUMBER, '15105551234', 'yo',
    (err, responseData) => {
      if (err) {
        console.log(err);
      } else {
        console.dir(responseData);
      }
    }
 );
```

Let's run this, and see if you get a SMS to your mobile phone.

`$ node index.js`

![SMS sent via Nexmo on Android](/content/blog/how-to-send-sms-messages-with-node-js-and-express/screenshot-sms.png)

I hope it worked! You have learned how to send an SMS message with Nexmo Node.js Library.

You can stop right here, or proceed to play with Express.js to be able to take the queries dynamically from POST requests!

### Building a Bare Minimal SMS App with Express.js

Let’s write a very simple app using [Express](http://expressjs.com/) to send an SMS.

Install Express and body-parser as a dependency:

```bash
$ npm install express body-parser --save
```

In the **index.js**, add the following code to start a server and listens on port 3000 for connections: 

```javascript
const express = require('express');
const bodyParser = require(body-parser);
const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
const server = app.listen(3000);
```

Now, wrap the `nexmo.message.sendSms()` with the Express `post` route method. Let’s set the `type` to `'unicode'` so you can send some emoji too! Also, print out the success response at the callback.

```javascript
app.post('/send', (req, res) =>; {
  // Send SMS
  nexmo.message.sendSms(
    config.number, req.body.toNumber, req.body.message, {type: 'unicode'},
    (err, responseData) => {if (responseData) {console.log(responseData)}}
  );
});
```

Now, try sending an SMS to any mobile phone number (including Google Voice numbers) using your app. 

In this tutorial, we are not going to create an HTML with the form UI where a user can fill out a phone number and message (I will write a full tutorial including a front-end code sometimes!), so let’s pretend we are sending data from a web interface by using [Postman](https://www.getpostman.com/) to make requests to your app. Postman is a good tool to have when you develop apps with REST APIs!

1. Launch Postman and Select **POST**, and enter *http://localhost:3000/send*. 
2. At **Headers**, Set *Content-Type: application/json*

![Send a post request to your app using postman](/content/blog/how-to-send-sms-messages-with-node-js-and-express/postman-headers.png)

3. At **Body**, type a valid JSON with "toNumber" and its value (use your mobile phone number! To receive an SMS message!), also “message” and its value. 

![Send a post request to your app using postman](/content/blog/how-to-send-sms-messages-with-node-js-and-express/postman-body.png)

4. Press the blue **Send** button

Once you made a successfully POST to your app, you will get a text message to your phone from the virtual number! 

![SMS on Android](/content/blog/how-to-send-sms-messages-with-node-js-and-express/screencast-sms.gif)

Also, you will see the response printed on your terminal.

![Response from Nexmo SMS API](/content/blog/how-to-send-sms-messages-with-node-js-and-express/terminal-response.png)

You can view [the code sample](https://github.com/nexmo-community/nexmo-node-quickstart/blob/master/sms/send-express.js) used in this tutorial on GitHub.

In the next tutorial, you will learn how to receive SMS messages to your virtual number. Stay tuned!

## References

* Nexmo SMS REST API <https://docs.nexmo.com/messaging/sms-api>
* Nexmo REST API client for Node.js <https://github.com/Nexmo/nexmo-node>
* [ExpressJS](https://expressjs.com)