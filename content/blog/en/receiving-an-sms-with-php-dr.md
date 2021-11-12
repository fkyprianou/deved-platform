---
title: Receiving an SMS with PHP
description: Go from zero to being able to receive inbound SMS messages in your
  PHP application—in just 20 lines of code—with the Vonage SMS API.
thumbnail: /content/blog/receiving-an-sms-with-php/blog_php_sms_1200x600.png
author: mheap
published: true
published_at: 2018-06-19T09:20:00.000Z
updated_at: 2020-11-03T10:20:57.522Z
category: tutorial
tags:
  - php
  - sms-api
comments: true
redirect: ""
canonical: ""
---
## Intro

We've previously covered [sending an SMS with PHP](https://learn.vonage.com/blog/2017/09/20/sending-sms-messages-with-php-dr/), but that's only half of the conversation. In this post we're going to look at allowing people to send you an SMS.

The source code for this blog post is available [on Github](https://github.com/nexmo-community/nexmo-php-quickstart/blob/master/sms/receive-with-slim/).

## Prerequisites

You’ll need PHP installed before working through this post. I’m running PHP 7.4, but the code here should work on PHP 7.3 and above. You'll also need [Composer](http://getcomposer.org/) to download our dependencies.

Finally, you'll need the [Vonage CLI](https://github.com/vonage/vonage-cli) installed. We'll use it to purchase a phone number and configure our Vonage account to point at our new application.

<sign-up number></sign-up>

## Receiving an SMS with PHP

When Vonage receives an SMS for a phone number that you own, they make a HTTP request to a URL that you've configured containing all of the information about the SMS. (Don't worry about configuring this URL yet, we'll get to it a little later on)

To receive the incoming SMS content, we're going to be using the [Slim framework](https://www.slimframework.com/) Let's install it now with `composer`:

```bash
composer require slim/slim:"4.*"
```

When we receive an SMS, we're going to log out all of the information that Vonage provide to the console. In the real world, you could store this in a file or a database.

Vonage will make either a `GET` or a `POST` request to your application with the data, depending on how your account is configured (you can see this under `HTTP Method` [in the dashboard](https://dashboard.nexmo.com/settings)). In this post, we'll write an application that can handle both HTTP methods:

Create a file named `index.php` with the following contents. We bootstrap our `Slim` app, define a handler that returns a HTTP `204` response and then instruct Slim to use this handler whenever we receive a `GET` or a `POST` to `/webhooks/inbound-sms`:

```php
<?php
use \Psr\Http\Message\ServerRequestInterface as Request;
use \Psr\Http\Message\ResponseInterface as Response;
use Slim\Factory\AppFactory;

require 'vendor/autoload.php';

$app = AppFactory::create();

$handler = function (Request $request, Response $response) {
    return $response->withStatus(204);
};

$app->map(['GET', 'POST'], '/webhooks/inbound-sms', $handler);

$app->run();
```

With this code all we're doing is returning with a `204` response code, which says that everything is OK. To log the parameters we received we need to check if there is any data returned by `\Vonage\SMS\Webhook\Factory::createFromRequest($request)`.

At this point, all of the parameters are stored in a variable named `$sms` and we can output them to the terminal using 
`error_log('From: ' . $sms->getMsisdn() . ' message: ' . $sms->getText());`. Putting that all together, your `$hander` should look like the following:

```php
$handler = function (Request $request, Response $response) {
    $sms = \Vonage\SMS\Webhook\Factory::createFromRequest($request);
    error_log('From: ' . $sms->getMsisdn() . ' message: ' . $sms->getText());

    return $response->withStatus(204);
};
```

Save this file and then open up a new terminal window. Let's start the built in PHP server and serve our application on port 8000.

```php
php -t . -S localhost:8000
```

If you visit [http://localhost:8000/webhook/inbound-sms?msisdn=14155550100&text=Hello+World](http://localhost:8000/webhook/inbound-sms?msisdn=14155550100&text=Hello+World), you should see `from` and `text` in the same terminal that you started the PHP server in.

That's really all there is to it. Receiving an SMS with Vonage is really easy due to the fact they transform an SMS in to a HTTP request for us.

## Exposing your application with ngrok

Whilst our application is complete, our job isn't quite finished yet. To send a HTTP request to our application, Vonage needs to know which URL our application is running on.

We're going to use [ngrok](/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/) to expose our local application to the internet. Run `ngrok http 8000` and make a note of the `ngrok` URL generated (it'll look something like `http://abc123.ngrok.io`).

## Configure your Vonage account

With Vonage, each phone number you own can have a different callback URL that they use to send inbound SMS to. 

Let's start by purchasing a phone number using the Vonage CLI that we can use to test. Firstly pick an available number to buy:

```bash
vonage numbers:search US
```

And then buy it:

```bash
vonage numbers:buy <number> US
```

Now create a new application with the URL to send the inbound SMS to:
```bash
vonage apps:create --messages_inbound_url=http://abc123.ngrok.io/webhooks/inbound-sms

Now we link our number to our application:

```bash
vonage apps:link <application_id> --number=14155550100

At this point, you can send an SMS to your Vonage number and watch as it appears in your terminal. It may take a few minutes due to network latency, but it should arrive soon!

## Conclusion

In just 20 lines of code, we went from zero to being able to receive inbound SMS messages in our application.

The developer docs have more information about [receiving inbound SMS messages](https://developer.nexmo.com/messaging/sms/guides/inbound-sms) with PHP, including a description of all of the available parameters that Vonage may send to you.

If you have any questions about this post feel free to [join the Vonage community Slack channel](https://developer.nexmo.com/community/slack), where we're happy to help.