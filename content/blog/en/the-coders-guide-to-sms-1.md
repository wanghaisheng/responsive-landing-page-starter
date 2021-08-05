---
title: The Coder's Guide to SMS
description: " In this guide, you will learn what SMS is, how companies are
  using it, the difference between SMS, MMS, and OTT, and more!"
author: cory-althoff
published: true
published_at: 2021-07-28T22:29:00.943Z
updated_at: 2021-07-28T22:29:00.959Z
category: inspiration
tags:
  - python
  - sms-api
  - ""
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
As a programmer, you may have come across SMS (short message service), but you may not know much about it. SMS is a service for sending short messages over wireless networks using standardized communication protocols. Neil Papworth sent the first SMS message on December 3, 1992. He wrote Merry Christmas to his co-worker Richard Jarvis, despite Christmas being almost a month away. An SMS message is one of two technologies for sending a text message: the other is called MMS.

Today, over[ four billion people send text messages a year](https://www.smseagle.eu/2017/03/06/daily-sms-mobile-statistics/) using technologies like SMS and MMS. Companies are increasingly using SMS messages to reach their customers on their mobile phones because they are convenient, and customers prefer them. SMS messages also have a 98% open rate, which is significantly higher than other forms of communication like email. As a programmer, it is essential to understand how SMS messages work since so many companies use them.

In this guide, you will learn everything you need to know about SMS as a programmer. You will learn what SMS is, how it works, and how companies use SMS messages to communicate with their customers. You will learn the difference between SMS, MMS, and OTT applications, as well as a few cool features SMS has that not many people know about. Finally, I will also point you to resources that will teach you how to send a text message programmatically in just a few lines of code. 

### How Do Companies Use SMS?

![](/content/blog/the-coders-guide-to-sms/digital-marketing-1433427_640.jpg)

More and more companies are using SMS to communicate with their customers because customers prefer SMS over other forms of communication.

Some companies use SMS for two-factor authentication, which is a popular way to help confirm a customer possesses the phone number they signed up for a service with.

Companies also use SMS to send marketing messages. With a 98% open rate, SMS messages are one of the most effective ways to keep customers updated about the status of their order, sales, and any other essential things customers need to know. SMS messages also are read quickly: [90% of all text messages are read within three seconds](https://www.tatango.com/blog/sms-open-rates-exceed-99/), which means SMS is perfect for things like flash sales.

Some companies also use SMS for customer service. For example, many hotels are adopting SMS as a way to better communicate with their guests.

Another use case for SMS messages is alerts and reminders. For example, hair salons often remind customers about their upcoming appointments using SMS messages, and banks send alerts via SMS when they think a customer’s card might be compromised.

### The SMS Standard

![](/content/blog/the-coders-guide-to-sms/screen-shot-2021-07-28-at-3.36.51-pm.png)

SMS lets you send 160 characters of text or 70 characters in Unicode. But, of course, you’ve probably sent an SMS message longer than 160 characters before, so how is that possible? When you send an SMS message longer than 160 characters, your phone carrier breaks the messages up and sends multiple messages. However, your phone carrier makes sure they arrive in order, which is why you’ve probably never noticed the 160 character limit.

When you send an SMS message, it does not go directly to the recipient's phone. Instead, your message first gets sent to a Short Message Service Center (SMSC), which looks up the recipient and sends the message to them: similar to SMTP.

Here is a diagram that shows how it works:

![](/content/blog/the-coders-guide-to-sms/screen-shot-2021-07-27-at-5.18.39-pm.png)

The MS at the bottom left and right stands for mobile station: the mobile phones sending and receiving the SMS message. When you send a message, your phone first sends it to a BSS or base station sub-system. The BSS manages the radio network. The BSS then sends your message to an MSC or mobile switching center. The MSC is the phone exchange serving your area (your city, for example). Your text message then travels from the MSC to the SMCS or short message service center. Your phone has an SMCS address that looks like a phone number configured in it, sometimes on your SIM (although iPhones do it slightly differently). Your SMSC then uses an HLR or Home Location Register to find where the recipient is. It gets back an SMSC address for the recipient and sends it to their MSC and back the same path to the recipient's phone.

### SMS VS. MMS

![](/content/blog/the-coders-guide-to-sms/video-conference-5167472_640.jpg)

As you learned earlier, when people say the word text message, they are referring to SMS and MMS. MMS stands for multimedia messaging service and allows you to send messages that include multimedia content over wireless networks.

When you send a text message that only has text, your phone uses SMS, but it uses MMS if you include a picture or video in your message. Because SMS messages only contain text and have a 160 character limit, they are cheaper to send than MMS messages. In addition, many people in the U.S. also have unlimited texting plans, so using SMS often means your users won't have to pay anything. Unlike SMS, which has a 160 character limit, you can send an MMS message with up to 1,000 characters.

Another thing to keep in mind when deciding whether to use SSM or MMS messages is while most people have a smartphone, not everyone does, and your customers without one cannot receive MMS messages.

MMS messages do have some advantages, though. Because MMS messages can include videos and pictures, they often have higher engagement rates and may get shared more on social media.

### Do iMessage and WhatsApp Use SMS?

![](/content/blog/the-coders-guide-to-sms/whatsapp-892926_640.jpg)

Apple's iMessage, WhatsApp, WeChat, and Facebook Messenger are examples of "Over the Top" or OTT applications. Unlike SMS, OTT applications like WhatsApp do not require the user to connect to a cellular network.

OTT applications like iMessage and WhatsApp do not use SMS. However, that does not mean iPhones do not send SMS messages: they do. You can only send an iMessage if you and the recipient both have an iPhone. If you send a text message on your iPhone to another Apple user, Apple will send the text using iMessage, and your iPhone will highlight the message in blue. If you send a text message (without multimedia) to another device (like Android), Apple will send it using SMS, and the message will be green. Android devices, on the other hand, often use a service called Android Messages. 

The advantages of OTT applications are they are free, allow users to send videos and other multimedia, and often have additional features like video chatting.

SMS has several advantages over OTT, however. OTT applications are "walled gardens," which means someone on Facebook messenger cannot message someone on WhatsApp. It also means you cannot send messages to phone numbers with OTT applications: you can only send messages to people who have downloaded that app. With SMS, you can send a message to anyone with a phone number. Plus, SMS does not rely on internet connectivity as OTT applications do. Unlike OTT applications, you can send a message to anyone connected to a cellular network with SMS.

### SMS can do what?

![](/content/blog/the-coders-guide-to-sms/snowboard-688504_640.png)

SMS has many cool less-known features. For instance, did you know you can edit an SMS message after you've sent it? Well, you can! You can "overwrite" an SMS message you previously sent. However, you must set this up in advance. You cannot edit the text you already sent someone last night!

You can also use SMS to send a flash message, also called a class zero message. A flash message is a message that pops up on your phone but, by default, doesn't save to your inbox. The point of a flash message is to enable you to send something that the receiver won't save by default. In other words, a flash message is a precursor to Snapchat's famous ephemeral messages. However, just like how with Snapchat, it is possible to screenshot a message, it is also possible to download a class zero message (or screenshot it), so you cannot entirely rely on it for secrecy. Another use case for a flash message is sending a message you want the recipient to read immediately. 

### Is SMS Secure? 

As a programmer, you know security is important. So you may be wondering how secure SMS is? SMS messages are not encrypted, which means your cellular provider can read them. While your mobile provider may not read your messages, providers like ATT&T, Sprint, and T-Mobile [have sold customer's data in the past.]((https://www.howtogeek.com/402043/can-anyone-really-track-my-phones-precise-location/)) Your provider also stores your SMS messages for some time. Because mobile providers store SMS messages for varying amounts of time, they can and have been used in divorce and criminal proceedings.  

You may be wondering why companies use SMS for two-factor authentication if they are not encrypted? Companies use SMS for two-factor authentification because it helps verify the customer has access to the number they signed up for a service with. However, some security professionals recommend against using SMS for two-factor authentification because of the security risks. 

For example, hackers can intercept SMS messages. That means a bad actor could access an incoming authentication code without possessing the user's phone. Hackers can also get around SMS two-factor authentication using a SIM swap. A SIM card swap is when a bad actor convinces your mobile provider (by using various lies) to activate a new SIM card linked to your number on a new phone (that they possess). When they use the new SIM card, they then have possession of your phone number. 

Because of these problems, you should not treat SMS messages as private. If you need to communicate sensitive data, you should use a messaging app that provides end-to-end encryption like Signal. Just because the messages are not encrypted also does not mean two-factor authentication with SMS is useless. It still forces bad actors to take additional steps (like a SIM card swap) to hijack the account you are protecting with it. 

Whether or not SMS two-factor authentification provides enough protection depends on the application. If you are providing additional verification steps to a video streaming service, SMS may be OK. However, suppose you are setting up two-factor authentification for a bank. In that case, you may use something more secure like Google Authenticator, which would require a hacker to gain access to your physical device. 

### Sending an SMS message Programmatically

![](/content/blog/the-coders-guide-to-sms/work-731198_640.jpg)

You can easily send an SMS message programmatically using an API like the one we have at Vonage. 

Sending an SMS programmatically using our API is simple: you can send one in just a few lines of code.

You can learn how to send an SMS message with our API by [reading our SMS API documentation](https://developer.nexmo.com/messaging/sms/overview).

We also have many blog posts that walk you through sending SMS messages with various technologies. For example, this article shows you [how to send an SMS message using Python and Flask](https://learn.vonage.com/blog/2017/06/22/send-sms-messages-python-flask-dr/), and this article [teaches you how to send an SMS message using Node Red.](https://learn.vonage.com/blog/2019/04/24/receive-sms-messages-node-red-dr/)

You can also [browse through our entire list of SMS articles and tutorials here.](https://learn.vonage.com/tags/sms-api/)

### Final Thoughts

With its convenience, ubiquity, and high open rates, SMS is a cornerstone of business communication. Because of its frequent use in the business world, all programmers need to be familiar with SMS and how to send an SMS message.

Now that you've read this guide, I hope you have a basic understanding of SMS and are ready to continue learning more about this communication method that is only growing more important.

If you want to send SMS messages programmatically, the [Vonage SMS API documentation](https://www.vonage.com/communications-apis/sms/) is the best place to start. 

I hope you enjoyed this guide, and [please reach out to us on Twitter](https://twitter.com/VonageDev) if you have any questions!