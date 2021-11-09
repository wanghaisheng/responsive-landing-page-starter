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

OK, what I really mean is that I'm going to introduce our use-case, but I like being a tad dramatic at times. Let's say we have our real-world Jurassic Park. The 
