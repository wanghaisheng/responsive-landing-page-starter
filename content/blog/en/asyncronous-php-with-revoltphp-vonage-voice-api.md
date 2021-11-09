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

#### PHP 8.1
You'll need PHP 8.1 for this, which has not officially been released. Mac users can find it under [shivammathur's homebrew repository](https://github.com/shivammathur/homebrew-php), Linux users can find it on [ondrej's apt PPA](https://launchpad.net/~ondrej/+archive/ubuntu/php/) and Windows users can find it on the QA section of [PHP for Windows](https://windows.php.net/qa/).

#### Composer
We need composer, PHP's de-facto dependency manager, so [follow the installation instructions for that here](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos) if you've not already got it.

#### Project space
The following requirements will need your project space, so create a new directory where the code will sit and use composer to create a `composer.json` configuration. Do this by running the following in your blank directory:

```composer init```

#### Slim Framework
To have a truly non-blocking Event Loop _and_ have HTTP request handling, you'd want to using something like [ReactPhp's HTTP client](https://reactphp.org/http/). For this example though, we need some routes open for the Voice API handling, and Slim is a quick way to do this. To get it, we use composer:

```composer require slim/slim```

We also need a PSR-7 compliant library to handle requests/responses (I've gone with Guzzle's, but several options are available):

```composer require guzzlehttp/psr7```

#### ngrok

If you've not come across ngrok before, it's a super useful tool for creating secure URL tunnels into your localhost. We'll need this for Vonage's webhooks to work. Check out the [installation instructions here](https://ngrok.com/download) and create yourself an account.

#### Vonage Voice API

Vonage provides a fully-featured API for sending and receiving calls, so we're going to use the core PHP SDK to send outbound calls. Install it with composer:

```composer require vonage/client-core```

### RevoltPhp

Finally, we need to get the Event Loop from RevoltPhp. It's currently still pre-release, so you'll need to specify the dev branch:

```composer require revolt/event-loop:dev-main```

## Setting up Vonage Applications & Numbers

In order to create outbound calls to warn the blissfully ignorant park workers of the danger at bay, you'll need to set up your Vonage account accordingly.

<sign-up number></sign-up>

## Make that call!

OK, let's get going on the Slim application. Create a directory in your project route named `public` and create a new php file in it named `index.php`. Our file will look like this:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;
use Vonage\Client;
use Vonage\Client\Credentials\Keypair;
use Vonage\Voice\Endpoint\Phone;
use Vonage\Voice\OutboundCall;
use Vonage\Voice\Webhook;

require __DIR__ . '/../vendor/autoload.php';

$keypair = new Keypair(
    file_get_contents('../revolt_php_example.key'),
    '940597b9-7f52-416f-8fd4-a19e0f689602'
);

$vonage = new Client($keypair);

$faker = Faker\Factory::create('en_GB');

$phoneNumbers = [];

for ($i = 1; $i < 1201; $i++) {
    $phoneNumbers[] = $faker->phoneNumber();
}

$app = AppFactory::create();

$app->get('/code32', function (Request $request, Response $response) use ($phoneNumbers, $vonage) {
    foreach ($phoneNumbers as $outboundNumber) {

        $outboundCall = new OutboundCall(
            new Phone($outboundNumber),
            new Phone('999999')
        );

        $outboundCall
            ->setAnswerWebhook(
                new Webhook('/webhook/answer', 'GET')
            )
            ->setEventWebhook(
                new Webhook('/webhook/event', 'GET')
            );

        $vonage->voice()->createOutboundCall($outboundCall);
    }

    $response->getBody()->write('Park employees notified.' . PHP_EOL);

    return $response;
});

$app->run();
```

There's a lot to digest here, so let's break it down.

Firstly we're setting up our Vonage client with credentials needed to make Voice calls, using a `Keypair` object and reading in the SSH file you can download when creating your Vonage application as the first argument, with the application ID as the second:

```php
$keypair = new Keypair(
    file_get_contents('../my-example-app.key'), //  <- SSH key downloaded from vonage and put in the root directory
    '9999999-7f52-416f-8fd4-a19e0f689602' // <- application key here
);

$vonage = new Client($keypair);
```

Next, we simulate a payload of phone numbers to call by using the [faker](https://github.com/FakerPHP/Faker/) library, set to a variable named `$phoneNumbers`.

```
$faker = Faker\Factory::create('en_GB');

$phoneNumbers = [];

for ($i = 1; $i < 2001; $i++) {
    $phoneNumbers[] = $faker->phoneNumber();
}
```
Faker allows you to set a locale, so in this case I chose UK numbers by setting it to 'en_GB'. If you want to set a different locale, [have a look at the faker documentation here](https://fakerphp.github.io/).

We're using a classic `for` loop to create the phone numbers into an array here, so we now have 2000 phone numbers ready to get their dino warnings. How do we do it? With a `foreach` loop in the endpoint:

```php
$app->get('/code32', function (Request $request, Response $response) use ($phoneNumbers, $vonage) {
    foreach ($phoneNumbers as $outboundNumber) {

        $outboundCall = new OutboundCall(
            new Phone($outboundNumber),
            new Phone('MY_VIRTUAL_NUMBER') // <- this is dummy phone number, make it your virtual number on your app
        );

        $outboundCall
            ->setAnswerWebhook(
                new Webhook('/webhook/answer', 'GET')
            )
            ->setEventWebhook(
                new Webhook('/webhook/event', 'GET')
            );

        $vonage->voice()->createOutboundCall($outboundCall);
    }

    $response->getBody()->write('Park employees notified.' . PHP_EOL);

    return $response;
});
```
So, we have an endpoint to hit on our app. It will loop through all the phone numbers to call, but there are two things needed to complete our **synchronous** warning. You see that `setAnswerWebhook()` method in the code above? Well, once we make that outbound call, Vonage needs to know what to do with it. This is where ngrok and our webhooks come in.

## Fire ngrok

Ngrok will open a tunnel up and give you a URL to localhost when you launch it. PHP has a built in web server, so we'll use that for localhost and then fire ngrok to open the tunnel.

