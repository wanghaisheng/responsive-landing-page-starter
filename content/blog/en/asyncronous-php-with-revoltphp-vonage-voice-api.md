---
title: Asyncronous PHP with RevoltPHP & Vonage Voice API
description: Think async PHP doesn't exist? It sure does, and now it's native!
author: james-seconde
published: false
published_at: 2021-11-12T20:08:25.763Z
updated_at: 2021-11-08T20:08:25.796Z
category: tutorial
tags:
  - php
comments: false
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
It may surprise some readers that asynchronous PHP is nothing new. PHP5.5 introduced generators way back in 2014, and since then we have seen the creation of [amphp](), [ReactPhp]() and [Swoole]()/[OpenSwoole]().

## Hello, fibers!

PHP developers tend not to think in terms of async programming due to the nature of the request/response lifecycle (with encapsulated state) we are comfortable working with. Something has happened that might just change that though: [the introduction of native fibers to PHP8.1](). While fibers may not be "true" async execution while runtimes like [node.js]() and [Go]() are, it certainly can give you a massive performance boost if executed without any blocking I/O.

## Hello, RevoltPhp!

A new project has been created off the back of the release of PHP8.1, [RevoltPhp](), which is a collaboration from the creators of amphp & ReactPhp, aiming to bring their experience in async PHP to fiber implementation. While it's best to think of it as more an "underlying library" for a framework to use on top of it (concepts such as Read/Writeable Stream callbacks can be pretty difficult to navigate), I'm going to show you a small taster of how you can learn this concept.

## Emergency! Asset out of containment!

OK, what I really mean is that I'm going to introduce our use-case, but I like being a tad dramatic at times. Let's say we have our real-world dinosaur park. The workforce need to be notified when a furious, human-eating lizard escapes out of its pen. Thing is, the communications system was written in <insert your favourite PHP framework of choice>, and therefore is technically in a blocking I/O language. You need to use Vonage to call 2000 park workers simultaneously with a text-to-voice warning, right? Let's get to making an asynchronous code thread.

## Setting up: PHP 8.1, Composer, Slim, ngrok, Vonage, RevoltPhp

### PHP 8.1
You'll need PHP 8.1 for this, which has not officially been released. Mac users can find it under [shivammathur's homebrew repository](https://github.com/shivammathur/homebrew-php), Linux users can find it on [ondrej's apt PPA](https://launchpad.net/~ondrej/+archive/ubuntu/php/) and Windows users can find it on the QA section of [PHP for Windows](https://windows.php.net/qa/).

### Composer
We need composer, PHP's de-facto dependency manager, so [follow the installation instructions for that here](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos) if you've not already got it.

### Slim Framework
To have a truly non-blocking Event Loop _and_ have HTTP request handling, you'd want to using something like [ReactPhp's HTTP client](https://reactphp.org/http/). For this example though, we need some routes open for the Voice API handling, and Slim is a quick way to do this. To get it, we use composer:

```composer require slim/slim```

We also need a PSR-7 compliant library to handle requests/responses (I've gone with Guzzle's, but several options are available):

```composer require guzzlehttp/psr7```

### ngrok

If you've not come across ngrok before, it's a super tool for creating SSL pipelines into your localhost. We'll need this for Vonage's webhooks to work. Check out the [installation instructions here](https://ngrok.com/download) and create yourself an account.

### RevoltPhp

The last requirement is to get the Event Loop from Revolt. It's currently still pre-release, so you'll need to specify the dev branch:

```composer require revolt/event-loop:dev-main

