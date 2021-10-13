---
title: Build a Web Application to Chat With Your Facebook Page Users
description: Use the the Client SDK's new Messages API integration to build a
  Web Application to chat with your Facebook Page users
author: dwanehemmings
published: true
published_at: 2021-08-25T20:59:42.499Z
updated_at: 2021-08-25T20:59:42.519Z
category: announcement
tags:
  - messages-api
  - javascript
  - client-sdk
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

The [Messages API](https://developer.nexmo.com/messages/overview) is being integrated into the [Client SDK](https://developer.nexmo.com/client-sdk/overview). This will provide a straightforward method where your customers, via Facebook Messenger, WhatsApp, Viber, and more, can communicate with an application you create.

By the end of this blog post, you’ll have a web application capable of sending and receiving messages from your Facebook page and your page’s Messenger. Sample code will be provided and the relevant parts to the Messages API integration will be explained.

## Create a Facebook page

Log into Facebook and [create a Test Facebook page](https://www.facebook.com/pages/creation/). You can also test with a page that already exists.

## Set up the sample web application

Make your copy of the sample web application by [remixing this Glitch](https://glitch.com/edit/#!/remix/messages-inbound-demo).  To get your application set up, follow the steps in the readme file. The sample application follows the scenario of an agent signing into a dashboard, with current conversations between Facebook page customers and the agent. On the left side is where all the conversations are happening, and the agent can join one.

![The agent's dashboard with all conversations in a column. Join buttons are on the left in a gray box. The rest of the page is space for conversations the agent is already a part of with an open link for each.](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/web-dashboard.png "Sample application's agent dashboard")

When the agent clicks a conversation, a chat application with the Facebook User will open in a new window. The chat application is based on the one created in the [Creating a chat app tutorial](https://developer.nexmo.com/client-sdk/tutorials/in-app-messaging).

![Screenshot of a chat room where the Agent and Facebook user are having a conversation.](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/demo-chat.png "Chat application demo")

## Link your Facebook page to your Vonage application

All that is left of the setup is to connect your Facebook page to the web application so they can communicate back and forth. Here are the steps:

* Log into the [Vonage Dashboard](https://dashboard.nexmo.com)
* Under “Messages and Dispatch” click “Social channels”

![Screen shot of Messages and Dispatch menu with submenu of Sandbox and Social channels](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/01-messages-and-dispatch-menu.png "Vonage Dashboard Messages and Dispatch menu")

* On the “Social channels” page, click Facebook Messenger

![Screen shot showing the options in the Social channels section of the Vonage Dashboard, WhatsApp, Viber, and Facebook Messenger](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/02-social-channels.png "Vonage Dashboard Social channels section")

* Follow the steps to connect your Facebook page to your Vonage Account

![Screen shot of steps to connect a Facebook Page](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-masked.png "Connect Facebook Page")

* Select the Facebook page(s) to connect to Vonage 

![Screen shot of a Facebook Page selector](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-2.png "Select a Facebook Page")

* Verify you are using the correct Facebook account

![Screen shot of a confirmation popup to continue as a user](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-3.png "Continue as user")

* Review the permissions granted to Vonage

![Screen shot of the list of permissions that can be granted to Vonage](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-4.png "Ask for permissions")

* Confirmation that Vonage was able to be successfully linked

![Confirmation of Facebook Page being successfully linked](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-5.png "Confirmation of Facebook Page being successfully linked")

* Select a Facebook page that was linked to Vonage and complete the setup

![Screen shot of Connect Facebook Page where the page can be selected.](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-6.png "Select your Facebook Business Page")

* Congratulations, your Facebook page was successfully connected

![Screen shot of Social Channels section of Vonage Dashboard with a success alert.](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/03-connect-facebook-page-7.png "Confirmation that Facebook Business page was successfully connected")

* Now that Vonage knows about your Facebook page, let’s connect to your Vonage application that was created when you set up the sample with Glitch. Either click “Link to an application” or go to the Applications section of the dashboard.

![Screen shot of Your applications section of the Vonage dashboard](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/04-your-applications-masked.png "Your applications")

* Select the application you created when setting up the Glitch sample and then click “Link”.

![Screen shot of Application details page with the Facebook page listed](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/05-application-selected.png "Application details")

![Screen shot of Application details page with the Facebook page linked to the application](/content/blog/build-a-web-application-to-chat-with-your-facebook-page-users/05-application-selected-2.png "Application details with Facebook page linked")

## Try it out

Place the web application in one browser window and your Facebook page in another. If you haven't already, enter a name in the web application to enter the dashboard. Think of this as your name or an agent's name, it's just a simple way to "log in". Now, in the window with the Facebook Page, send a message from your Facebook account. A little card should appear in the all conversations section of the web application. Click join, a chat application will open up, and you should see the message in the chat. Send a message from the chat application, and it should appear in your Facebook Messenger.

## What’s Happening

Let’s take a look at the code involved to make the above happen.
When a user sends a message to your Facebook page, it gets sent by Vonage to your web application’s inbound webhook. The webhook returns an object that lets Vonage know how to handle the message. In this case, we are sending back information the Client SDK Messages API Integration needs to connect the Facebook User with your web application to have a conversation. This includes the Facebook User’s id and the conversation name (which we set as the Facebook User’s id so that it’s unique).

```javascript
// server.js
app.post("/webhooks/inbound", (request, response) => {
  response.status(200).send([{ "action": "message", "conversation_name": request.body.from.id, "user": request.body.from.id, "geo": "region-code" }]);
});
```

If this is the first time the Facebook User sends a message, a new conversation is created. This emits a `conversation:created` event that we listen for on the events webhook. The web application’s backend takes this event and repackages it as a custom event, `custom:new_conv`, that can be used to notify the agent’s dashboard to display the new conversation. 

```javascript
// server.js
app.post("/webhooks/rtcevent", (request, response) => {
   ...
  // If Facebook user is new, a new conversation should be created, so listen for it here 
  // and then send the custom event to the Agents Conversation
  if (request.body.type === "conversation:created"){
    vonage.conversations.events.create(process.env.AGENTS_CONV_ID, {
      "type": "custom:new_conv",
      "from": "system",
      "body": request.body
    },
    (error, result) => {
      if (error) {
        console.error(error);
      } else {
        console.log(result);
      }
    });
  }
  response.status(200).send({code:200, status: 'got rtcevent request'});
});
```

In the code for the chat application, there is a `message:received` event listener that fires when a message is received from the Facebook User. It then takes the message and adds it to the chat display.

```javascript
// public/chat.js
   // Adding conversation.id here in the on. means that we're filtering events to only the ones regarding this conversation. (it's called grouping)
    conversation.on('message:received', conversation.id, (from, event) => {
      console.log('message-received sender: ', from);
      console.log('message-reveived event: ', event);
      const formattedMessage = formatMessage(from, event, conversation.me);
      // Update UI
      messageFeed.innerHTML = messageFeed.innerHTML + formattedMessage;
      messagesCountSpan.textContent = messagesCount;
    });
```

When the agent responds to the Facebook User, that is an outbound message. Full implementation of outbound messages into the Client SDK will be completed in a future release. Until then, just like listening for when a new conversation is created, we can listen for when a `custom:chat` event is sent to the events webhook. The Vonage Node SDK is used to relay the agent’s message to the Facebook User.

```javascript
// server.js
app.post("/webhooks/rtcevent", (request, response) => {
  ...
  // We currently do not support the outbound part of the integration so you should take
  // the text message from the chat and relay to Messenger using the messages API 
  // (here, using the Server SDK wrapping of it)
  if (request.body.type === "custom:chat"){
    const FB_RECIPIENT_ID = request.body.body.to;
    const FB_SENDER_ID = request.body.body.from;
    vonage.channel.send(
      { type: 'messenger', id: FB_RECIPIENT_ID },
      { type: 'messenger', id: FB_SENDER_ID },
      {
          content: {
              type: 'text',
              text: request.body.body.text,
          },
      },
      (err, data) => {
          if (err) {
              console.error("error sending outgoing message: ", err);
          } else {
              console.log("message sent successfully: ", data.message_uuid);
          }
      }
    );
  }
  ...
});
```

For those wondering, `FB_RECIPIENT_ID` is also the Conversation Name that was set when the initial message was sent from the Facebook User. Getting the value for `FB_SENDER_ID`, the Facebook Page Id, is a little more involved. When an agent opens a chat, a request is made to the server's `getChatAppAccounts` endpoint which makes a call to Vonage's `chatapp-accounts` API with an admin JWT. The Facebook Page Id is in the response, which we send back to the client. The good news is, when Outbound Messages gets integrated, you won't have to worry about any of this, and the Client SDK will take care of everything. This is an example of how the Client SDK can be very helpful when developing applications.

## Small Gotcha

If your outbound messages stop working all of a sudden, check your server log for any errors. If you come across an error that has this:

```json
status: 'rejected',
error: { reason: 'Expired access Token', code: 1370 }
```

That means your Facebook Page token has expired and will need to be refreshed. Log into the Vonage dashboard and go to [Messages and Dispatch, then Social Channels](https://dashboard.nexmo.com/messages/social-channels). A button should be next to your Facebook Page to refresh your token. 

## Conclusion

That’s it! With the Messages API integrated into the Client SDK, it is a lot easier to communicate with a Facebook User from your own web application.

## Next Steps

Please have a look at our [Client SDK documentation](https://developer.nexmo.com/client-sdk/overview). There’s more information on the methods used to create the Agent Dashboard, along with Tutorials, Guides, and more.
Ran into any issues with the demo application? Looking to add new functionality? Any questions, comments, and/or feedback, please let us know in our [Community Slack Channel](https://developer.nexmo.com/slack)
