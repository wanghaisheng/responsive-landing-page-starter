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

There's got to be a better, smarter way! Thankfully my favorite food delivery app, Wolt, has a public API that let's me know if a restaurant is online. So using the Vonage Messages API, I created Facebook Messenger Bot that will alert me when my favorite restaurant is back online! 

*(This example is built around a use case of food delivery but the same code can be repurposed to build a Facebook Bot that will alert users for any boolean case change.)* 

## Pseudo Code:

Before I get started with any coding task, I like to think out the logic. Let's breakdown the steps needed to build this app:

1. Setup an Express Server
2. Connect to Vonage Messages API Sandbox
3. Call the Wolt API for a requested restaurant 
4. Check if the received restaurant is online
5. Send a message to the user based on the restaurant status
6. Loop if the restaurant is offline

## Setup

Let's begin will by creating our project. First w

```
mkdir isItDelivering
cd isItDelivering
npm init
npm install @vonage/server-sdk@beta express dotenv got -s
touch index.js .env

```