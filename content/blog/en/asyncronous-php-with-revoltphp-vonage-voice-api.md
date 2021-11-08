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
It may surprise some readers that asynchronous PHP is nothing new. Both [amphp]() and [ReactPhp]() have been using PHP's generators introduced in [PHP5.5]() way back in 2014. PHP developers tend not to think in terms of asynchronous programming due to the nature of the request/response lifecycle (with encapsulated state) we are comfortable with. While the amphp and ReactPhp libraries have introduced workarounds with co-routines, it's never quite been near-native as much as languages with runtimes that have been designed that way - for example, [node.js]() or [Go]().