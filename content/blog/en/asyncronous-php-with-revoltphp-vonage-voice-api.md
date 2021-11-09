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

PHP developers tend not to think in terms of asynchronous programming due to the nature of the request/response lifecycle (with encapsulated state) we are comfortable working with. Something has happened that might just change that though: [the introduction of native fibers to PHP8.1]().