---
title: Two-way Messaging on Zendesk With Vonage Messages API
description: Connect WhatsApp with Zendesk via Vonage Messages API to create and
  update tickets.
author: amanda-cavallaro
published: true
published_at: 2022-07-12T16:56:50.285Z
updated_at: 2022-07-12T16:56:50.306Z
category: tutorial
tags:
  - javascript
  - zendesk
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Hi there! In this tutorial, we’ll outline a proof of concept of Zendesk integration with Vonage Messages APIs. A typical use case for such integration would arise from the need to introduce additional communication channels in the customer support department. 

Emailing your customers about Zendesk ticket updates not working well enough? Reach out to your users on SMS or, better yet, on WhatsApp!

## We’ll be using

* Vonage messages API, 
* Zendesk triggers/automations,
* A sample application as a middle layer. Here is the [available GitHub SDK](https://github.com/Vonage-Community/tutorial-messagesAPI-nodejs-vonageZendesk) (developed in collaboration with [Toni Kuschan](https://www.linkedin.com/in/kuschan))

## Objective

Connect WhatsApp with Zendesk via Vonage Messages API to create and update tickets.

## Workflow 

An incoming message on WhatsApp creates a Zendesk ticket (if there aren’t any open tickets from the same requestor). If there are open tickets, the requester receives a WhatsApp message with the ticket details. Additionally, ticket updates trigger a WhatsApp message to the requestor. Below is the workflow carried out by this use case.

![Step 1: User sends a message on preferred messaging channel (could be SMS or WhatsApp) Step 2: Inbound webhook added in Vonage application captures the message from step 1 Step 3: The back end application utilizes Zendesk create  or search ticket API to create or search a ticket based on the inbound message Step 4: Upon successful ticket creation or identification, use Vonage messaging API to update the requester on the ticket details](/content/blog/two-way-messaging-on-zendesk-with-vonage-messages-api/vonage-and-zendesk_createticket.jpg "Zendesk and Vonage high level architecture to create tickets")

![Step 1: Configure webhook and triggers on Zendesk to be triggered based on the required action. In this tutorial, the webhook is triggered when an agent updates a ticket Step 2: Utilize Vonage messages API to update the requester on the ticket details based on the original messaging channel (could be SMS or WhatsApp)](/content/blog/two-way-messaging-on-zendesk-with-vonage-messages-api/vonage-and-zendesk_updateticket.jpg "Zendesk and Vonage high level architecture to update ticket")

## Prerequisites

* Vonage application
* WhatsApp number
* Zendesk account (trial account will do as well)

## Create a Zendesk Ticket From WhatsApp

1. Deploy webhooks to perform the logic for inbound WhatsApp messages. You can deploy these webhooks on open source platforms such as ngrok or groom IDE
2. Inbound message webhook will do the following:

   1. Call Zendesk Search API to check if there’s an open ticket raised by the requestor (in this case, the requestor is the mobile number of the user messaging over WhatsApp)
   2. If the ticket is available, use messages API to send the requestor ticket details on WhatsApp
   3. If no open tickets are found, use Zendesk create ticket API to create a ticket based on the user’s inbound message. Use messages API to confirm back successful ticket creation

## Update a Zendesk Ticket

1. Deploy a webhook that will be triggered on a chosen action (the webhook is triggered when an agent adds a comment on a ticket)
2. From the Zendesk admin center, navigate to Apps and integrations, webhooks, and create a new webhook. Add details of your webhook as below

![1. Navigate to Apps and integrations from the left-hand menu options. 2. Select webhooks and click on create a new webhook 3. Add details of your webhook, including name, an optional description, endpoint URL, request method and  authentication mechanism](/content/blog/two-way-messaging-on-zendesk-with-vonage-messages-api/create-webhook.png "Create a new webhook")

3. Now it’s time to link the webhook to a trigger. From objects and rules, Business rules, choose Triggers.
4. Add a new trigger. Give a name to the trigger and select the conditions as per your use case. For this example, the webhook is triggered when a ticket is updated by an agent.

![1. From Objects and rules in the left-hand menu, navigate to Business rules 2. Select Triggers 3. Choose Add a new trigger 4. Add a name to the trigger and select an appropriate category 5. Select the required conditions for the trigger. In this tutorial it is if a ticket is updated by an agent](/content/blog/two-way-messaging-on-zendesk-with-vonage-messages-api/create-trigger_choosecondition.png "Create trigger")

5. Add an action and select Notify active webhooks
6. Choose from various placeholders the ticket details you want to send back to your webhook

## Conclusion and Next Steps

Today you learnt how to connect WhatsApp with Zendesk using the Vonage Messages API. This tutorial can be customized and tailored to your business requirements and use cases. Care to take it a step further! Check out our conversational AI platform, [Vonage AI studio](https://studio.docs.ai.vonage.com/), to build a complete workflow.