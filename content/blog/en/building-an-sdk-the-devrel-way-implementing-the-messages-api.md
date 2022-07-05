---
title: "Building an SDK the DevRel Way: Implementing the Messages API"
description: This article provides an inside look into the design of the Java
  SDK's implementation of the Vonage Messages v1 API.
author: sina-madani
published: false
published_at: 2022-07-04T11:48:15.975Z
updated_at: 2022-07-04T11:48:16.003Z
category: release
tags:
  - java
  - messages-v1
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
The [Vonage Messages v1 API](https://developer.vonage.com/messages/overview) was recently promoted to General Availability status. Following this announcement, the Developer Relations Tooling Team have been hard at work on implementing support for it in our server SDKs. Collectively, we decided that each SDK should implement the API in whatever way makes the most sense, given the language, community and general approach used to support other APIs in the SDK, for consistency. Our goal is to ensure that each SDK feels idiomatic, rather than a one-size-fits-all approach where the implemetnation feels auto-generated. The Messages v1 API is a good way to illustrate this, hence this article. Naturally, as the Java Developer Advocate on the team, I'm going to focus on the Java SDK's implementation and describe the rationale behind its design. But first, let's examine the [Messages API specification](https://developer.vonage.com/api/messages-olympus), which is what we need to implement.

The API has a single endpoint, which is to send a message via a POST request. The complexity arises from the schema of messages. There are two main concepts here: *channels* and *message types*. The former refers to the service that is used to send the message (SMS, MMS, WhatsApp, Viber, Facebook Messenger). The latter describes the medium of the message (text, image, audio, video, file etc.). Each channel supports a subset of message types. For example, you can't send a video via SMS. Some channels, like WhatsApp, support templates and custom message types, which allow you to send more complex messages or [share your location](https://github.com/Vonage/vonage-java-code-snippets/blob/master/src/main/java/com/vonage/quickstart/messages/whatsapp/SendWhatsappLocation.java). Furthermore, each channel places unique restrictions on message contents. For example, the length limit on texts varies by channel, as do supported file types for media messages - for instance, WhatsApp supports a wide range of audio formats, whilst Messenger only supports MP3. For media messages, some channels allow an optional text description (referred to as a *caption* in the API specification) to accompany the file, whilst others do not. This may even vary within channels - for example, one can include a caption with an MMS, except if it's a vCard (which, by the way, is unique to the MMS channel). From an implementor's perspective, the challenge then is to find a suitably structured and efficient way to capture these requirements whilst minimising confusion to consumers of the API.

As a Java developer, of course the natural thing to do is start with the data model. Inevitably we end up with an abstract type to represent a message, which is accepted by the `sendMessage` method of [`MessagesClient`](https://github.com/Vonage/vonage-java-sdk/blob/main/src/main/java/com/vonage/client/messages/MessagesClient.java). Since the response we get back from sending a message is the same regardless of the message type and channel, we don't need to care about the message itself, we just forward it as a JSON payload and what we get back is a unique identifier of the message that was sent. With that in mind, our abstract [`MessageRequest`](https://github.com/Vonage/vonage-java-sdk/blob/main/src/main/java/com/vonage/client/messages/MessageRequest.java) factors out the fields common to all messages - namely: the type, channel, sender, recipient and an optional client reference string. Since the channels and message types are pre-defined, we represent them as enumerations. There are a mixture of optional and mandatory parameters (for example, client reference is optional, but sender and recipient are mandatory). There are more parameters too, depending on the channel and message type combinations. Since Java does not have named arguments, we use the builder pattern to emulate this. You can read more about this here (link to other article).

We want to enforce that only valid combinations of message type and channel can be constructed, so we define this matrix in the [`Channel`](https://github.com/Vonage/vonage-java-sdk/blob/main/src/main/java/com/vonage/client/messages/Channel.java). We validate all arguments in constructors of MessageRequest and its subtypes. Herein lies what I believe is the main advantage of the design of the Java SDK's implementation of the Messages API: everything is ***correct by construction*.** 

What does this mean exactly? It means that in theory, it is difficult, ideally, impossible to construct an invalid message. I challenge anyone reading this to try to "misuse" the Messages API using the Java SDK. Let's try it together.

`MessageRequest` is abstract, so we have to use one of its concrete subtypes. All of these subtypes (for example, `MmsImageRequest`) already set the Channel and MessageType. Besides, even if we created our own subclass of `MessageRequest`, we still can't bypass the MessageType and Channel validation check, since it's done in the constructor of `MessageRequest`. Furthermore, enums are implicitly final so we can't add our own MessageType or Channel, or override the validation behaviour. Furthermore, the constructors for all sublcasses of `MessageRequest` are either protected (for abstract classes) or package-private (for concrete classes), and none of the concrete classes can be extended since they are `final` (same is true of the builders). So structurally, we cannot create a "bad" MessageRequest class.

Okay, let's try something else. If we can't structurally misuse `MessageRequest`, perhaps we can pass in invalid values to the builders. For example, could we send an empty text? Or what about passing in a nonsensical string in the `to` (recipient) field instead of a number? Or what if we omit mandatory parameters (such as the sender and recipient)? Oh here's another one: all of the file-based message types (those that accept an image, audio, video etc.) have a URL field. Could we not pass in an invalid URL string? Or what about that Time-to-Live field in Viber messages? We could set a number that's out of the acceptable bounds, since it's just an `int`, right? Oh here's a niche one: in Messenger requests, both `Category` and `Tag` are optional, however the API requires that if the Category has a value of `message-tag`, then the tag must be present. And what about...

Let's stop there. All of these cases have been considered, and none of them are possible. Yes, your code will compile if you try to pass in a malformed URL, an invalid number or neglect to set mandatory parameters on the builder. But at runtime, you will not be able to send the message. You won't even get as far as constructing the `MessageRequest` object! Why? Because every parameter is validated in the constructor. The moment you call `build()` on the Builder, the constructor is invoked, and if the parameters used to construct the message are missing or invalid, then you'll get an `IllegalArgumentException` thrown at you explaining the problem. This is what I meant when I said "correct by construction". If you can construct a `MessageRequest`, then as far as we can tell, it's not obviously wrong, and you are clear to proceed to sending it. Of course, that doesn't mean that the message is completely valid or that you won't get back an error response from the server. You could, for example, try sending a text to a phone number that doesn't exist, yet is an E164-compliant number. This kind of validation is beyond the scope of the SDK and is performed by the backend service that the API communicates with. What the SDK guards against is essentially the 422 errors. So, what is this in contrast to? Where am I going with this? To answer that, let's look at the diametric opposite: cURL.

Why do I need the SDK when I can use the API directly? That's fine, here's an example of sending an image over MMS:

```
curl -X POST https://api.nexmo.com/v1/messages \
  -H 'Authorization: Bearer '$JWT\
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d $'{
            "message_type": "image",
            "image": {
                "url": "https://example.com/image.jpg"
            },
            "to": "'$TO_NUMBER'",
            "from": "'$FROM_NUMBER'",
            "channel": "mms"
}'
```
