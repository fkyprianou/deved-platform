---
title: How to Send SMS Messages with PHP
description: Find out how to send SMS messages using PHP via the Vonage API.
  Learn more in this tutorial from Vonage Developer.
thumbnail: /content/blog/sending-sms-messages-with-php-dr/sending-sms-featured.png
author: mheap
published: true
published_at: 2017-09-20T13:29:00.000Z
updated_at: 2020-11-06T13:29:25.414Z
category: tutorial
tags:
  - php
  - sms-api
  - slimphp
comments: true
redirect: ""
canonical: ""
---
The [Vonage SMS API](https://developer.vonage.com/en/messaging/sms/overview) makes it easy for you to send and receive SMS with PHP.

You can [read the docs](https://developer.vonage.com/en/api/sms) if you're interested, but there's no need to thanks to the [PHP client](https://github.com/Nexmo/nexmo-php), which handles talking to the API for you!

In this tutorial, you’ll see how easy learning how to send and receive SMS using PHP is with the Vonage SMS API.

## Prerequisites

To send an SMS via PHP using the steps in this tutorial, you won’t need any outside tools or programs. All you need is a working version of PHP and access to the Vonage API.

Below is a full list of what you’ll need to start sending SMS messages via PHP:

* A [Vonage API account](https://developer.vonage.com/sign-up). Access your Vonage API Dashboard to locate your API Key and API Secret, which can be found at the top of the page.
* A Vonage phone number. To purchase one, go to Numbers > Buy Numbers and search for one that meets your needs.
* [PHP](https://www.php.net/downloads) (5.6 or above)
* [Composer](https://getcomposer.org/) to install the PHP client

<sign-up number></sign-up>

## Instructions

### How to Send an SMS with PHP Using nexmo-php

One easy way to send an SMS with PHP is by using nexmo-php, a custom PHP client. Below is a breakdown on how to use nexmo-php to start sending text messages.

#### 1. Install the PHP Client

The first thing we need to do is install `nexmo/client` using `composer`. This will install the PHP client and all of its dependencies.

```bash
composer require nexmo/client
```

Once this completes, we're only three lines of code away from sending an SMS using PHP.

After this, we're going to create a file called `send-sms.php`, provide our API key and secret, create a `Text` with `to`, `from` and a `message`. Then, we're going to call the `send` function. That's really all there is to it.

#### 2. Create the PHP File

Go ahead and create `send-sms.php` now with the following contents. Don't forget to replace `NEXMO_TO_NUMBER` with your own phone number, making sure that it starts with a country code e.g. `14155550100` rather than `(415) 555-0100`. 

You'll also need to change `NEXMO_FROM_NUMBER` to set your SenderID. This generally has to be a Vonage number, but in some countries you can use an alphanumeric sender ID. You can [read more about sender IDs in the Vonage API knowledge base](https://help.nexmo.com/hc/en-us/articles/204014573-Can-I-Change-the-Sender-ID-for-Nexmo-Outbound-SMS-) and [purchase a number](https://dashboard.nexmo.com/buy-numbers), if required.

```php
require_once 'vendor/autoload.php'; 
$client = new NexmoClient(new NexmoClientCredentialsBasic(API_KEY, API_SECRET)); 
$text = new NexmoMessageText(NEXMO_TO_NUMBER, NEXMO_FROM_NUMBER, 'How to send an SMS with PHP'); 

$response = $client->message()->send($text);
print_r($response->getResponseData());
```

#### 3. Run the PHP File to Send an SMS

Save this file, then run it with `php send-sms.php`. Once you run the PHP file, the file will send the SMS message.  You will receive a text message shortly to the number you provided in `NEXMO_TO_NUMBER`. Our script will also output the response from Vonage, which contains information about the message you just sent along with how much credit you have remaining on your account.

### How to Send an SMS with a PHP API Using SlimPHP

Sending an SMS with three lines of code is pretty awesome, but it's not very flexible. Wouldn't it be great if we could dynamically change the `NEXMO_TO_NUMBER` and the contents of the `message` by making calls to an API? Let’s do just that! 

By following the steps below, you can use SlimPHP to send an SMS with a PHP API. This will allow you to send text messages with dynamic content.

#### 1. Install the SlimPHP Client

We're going to be using [SlimPHP](https://www.slimframework.com/) to power our API. This means that the first thing we need to do is require it with `composer`. This will download and install `Slim` and all of it's dependencies.

```bash
composer require slim/slim "^3.0"
```

#### 2. Create a PHP API with SlimApp

Once SlimPHP is installed, we can create an API that responds to our requests. Create a file named `index.php` with the following contents:

```bash
require 'vendor/autoload.php'; 
$app = new SlimApp(); 

$app->post('/sms/{number}', function ($request, $response, $args) {
    return $response->write("Sending an SMS to " . $args['number']);
});

$app->run();
```

This creates a new instance of `SlimApp` and registers a route that we can call. This allows us to make a `POST` request to `/sms/{number}` and it'll send a response back to us (but it won't send an SMS yet!). 

Save your file and start up `PHP`'s built in server by running `php -S localhost:8000 -t .`.

We're going to make a HTTP `POST` request to `http://localhost:8000/sms/` using an application called [Postman](https://www.getpostman.com/).

When we click on `Send`, we should get a response that says "Sending an SMS to \[number]". This lets us know that our Slim application is running correctly. Now, we can start building our SMS functionality.

![Animation of a user in Postman selecting POST, entering  a URL, and submitting a request.](/content/blog/sending-sms-messages-with-php-dr/send-sms-postman.gif)

#### 3. Add PHP Code to Send SMS Messages

As we already have our route set up, we can take our PHP code that sends an SMS and add it into the API.

```php
$app->post('/sms/{number}', function ($request, $response, $args) {
    $client = new NexmoClient(new NexmoClientCredentialsBasic(API_KEY, API_SECRET));
    $text = new NexmoMessageText($args['number'], NEXMO_FROM_NUMBER, 'How to send an SMS with PHP');
    
    $client->message()->send($text);

    return $response->write("Sending an SMS to " . $args['number']);
});
```

#### 4. Make the SMS Message Customizable

At this point, we can use our PHP API to send a text message to any phone number we like. There's one last change to make though: Let's make that message customizable.

To customize the message, we need to read data from the request to our API. We can use the `$request-&gt;getParsedBody()` method to return the payload of the incoming request as an array. We could use a key to contain our data, but we're going to use `text` as that's the parameter name that the [Vonage API](https://developer.nexmo.com/api/sms) uses. In addition to reading the request body, we're going to perform some input validation to make sure that `text` has been provided before passing it in to our `Text` object.

```php
$app->post('/sms/{number}', function ($request, $response, $args) {
    $body = $request->getParsedBody();

    if (!isset($body['text'])) {
        return $response->withStatus(400)->write("No message provided");
    }

    $client = new NexmoClient(new NexmoClientCredentialsBasic(API_KEY, API_SECRET));
    $text = new NexmoMessageText($args['number'], NEXMO_FROM_NUMBER, $body['text']);
    $client->message()->send($text);

    return $response->write("Sending an SMS to " . $args['number']);
});
```

#### 5. Send an SMS Message with Your PHP API
At this point, we have everything we need to send an SMS with our PHP API. Give it a go yourself via Postman.

This is all we need to send an SMS with PHP via Vonage. Give it a go yourself via Postman.

We're all done! By using SlimPHP we quickly bootstrapped an API that can send an SMS to any phone number.

Want to learn other ways you can use PHP to handle SMS messages? Check out our tutorials on how to receive text messages and SMS delivery receipts with PHP.

Want to learn other ways you can use PHP to handle SMS messages? Check out our tutorials on how to [receive text messages](https://developer.vonage.com/en/blog/receiving-an-sms-with-php-dr) and [SMS delivery receipts](https://developer.vonage.com/en/blog/receiving-sms-delivery-receipts-with-php-dr) with PHP.

<script type="text/javascript" async src="https://platform.twitter.com/widgets.js"></script>

<script>
window.addEventListener('load', function() {
  var codeEls = document.querySelectorAll('code');
  [].forEach.call(codeEls, function(el) {
    el.setAttribute('style', 'font: normal 10pt Consolas, Monaco, monospace; color: #a31515;');
  });
});
</script>