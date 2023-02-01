---
title: Send WhatsApp Messages in Laravel with Vonage's native SDK
description: Sending WhatsApp messages in Laravel is now easier than ever!
author: ashley-allen
published: true
published_at: 2023-02-01T13:28:08.265Z
updated_at: 2023-02-01T13:28:08.305Z
category: tutorial
tags:
  - php
  - laravel
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
In your Laravel applications, you may want to contact your users via WhatsApp. Whether it be to send notifications to your users or maybe broadcast sales promotions to potential leads. Being able to send messages directly to your users' phones and communicate with them can be an extremely powerful tool in your arsenal.

However, communicating directly with the WhatsApp API can sometimes be a bit overwhelming. So instead of doing this, we can use Vonage to handle sending the messages and communicating with the WhatsApp API.

Vonage is a cloud communications platform that provides a variety of APIs that allow you to communicate with your users via SMS, Voice, WhatsApp, and more. It provides a convenient way to send WhatsApp messages without having to worry about the underlying complexity of the WhatsApp API. [It also has a Laravel package](https://github.com/Vonage/vonage-laravel) that makes it super easy to integrate into your Laravel application.

In this tutorial, we will look at how you can send WhatsApp messages using Laravel and Vonage.

## Things To Be Aware Of

When working with WhatsApp, there are several things to be aware of with regard to what types of messages you can send, and when you can send them.

### Message Templates

In order to try and reduce the amount of spam that businesses can send to people, a business (such as yourself) can **only** start a conversation with a user by sending a "message template" (MTM). MTMs are pre-approved messages that you need to create in the WhatsApp Business Manager and that need to be approved by WhatsApp before you can use them.

The MTMs support parameters that allow you to customise the messages that you send to the users. For example, you can create an MTM that has the following contents:

```text
Hello {{1}}. Your order {{2}} has been shipped.
```

If we were to use `"John"` as the first parameter and `"1234"` as the second parameter, the message that would be sent to the user would be:

```text
Hello John. Your order 1234 has been shipped.
```

However, if the user messages you first, you will be able to reply to the user within a 24-hour window without having to use an MTM.

### 24-Hour Customer Care Window

WhatsApp has a concept of a 24-hour customer care window. Within this window, a business can freely message an end user without the need for sending a templated message.

The 24-hour window can be started in two different ways:

* The user replies to an MTM sent by the business.
* The user sends a message to the business.

So, as you can see, the 24-hour window requires the user's participation in the conversation.

If no message is sent within the 24-hour window, the window will expire and you will need to start a new one again.

## API Keys and Connecting WhatsApp

Before we get started with anything code-related, you'll first need to make sure that you have an active WhatsApp Business account with a phone number configured.

You'll also need to have a Vonage account. If you don't have one, you can sign up for an account [here](https://dashboard.nexmo.com/sign-up). Once you have an account, you'll need to grab your API key and secret from your Vonage dashboard.

Once you've done this, you'll be able to head to the "External Accounts" page on your Vonage dashboard. Here, you'll be able to connect your WhatsApp account to your Vonage account. You'll need to follow the steps to link your account together.

After doing this, your Vonage account should now be connected to WhatsApp and ready to start using.

If you have any issues setting up your Vonage account or connecting your WhatsApp Business account, it's worth checking out the [Vonage documentation](https://developer.vonage.com/messages/concepts/whatsapp) for more information.

## Installation and Configuration

Now that our Vonage account is ready to use, we can start setting up our Laravel application to use it. To get started, we'll need to install the [Vonage Laravel package](https://github.com/Vonage/vonage-laravel/) using Composer. You can do this by running the following command in your project root:

```bash
composer require vonage/vonage-laravel
```

Once the package has been installed, we can publish the configuration file for the package. This will allow us to configure our Vonage API key and secret. You can do this by running the following command:

```bash
php artisan vendor:publish --provider="Vonage\Laravel\ServiceProvider"
```

For the purpose of this tutorial, I'm going to be making the assumption that the same phone number is used each time we want to send a WhatsApp message, so we'll set our WhatsApp phone number in my `.env` file. To be able to do this, we'll also need to add a new config field so we can retrieve the phone number in our application's code. I'll place this new config field in my `config/services.php` file:

```php
return [

	// ...
    
	'vonage' => [
    	'whatsapp' => [
        	'from_phone_number' => env('WHATSAPP_PHONE_NUMBER'),
    	],
	],

	// ...

];
```

We'll then be able to retrieve this value in our application's code by using `config('services.vonage.whatsapp.from_phone_number')`.

Now that we've created our new config fields, we can add our phone number, API key, and API secret to our `.env` file:

```dotenv
VONAGE_KEY=YOUR-API-KEY-HERE
VONAGE_SECRET=YOUR-VONAGE-SECRET-HERE
VONAGE_WHATSAPP_FROM=YOUR-WHATSAPP-PHONE-NUMBER
```

That's it! We're ready to start sending WhatsApp messages.

## Sending WhatsApp Messages Using Vonage

Now that we have our application ready, we can now start sending WhatsApp messages to our users.

### Sending WhatsApp Message Templates (MTM)

As we've discussed earlier, if a business wants to start a new 24-hour window and open a conversation with the user, they'll first need to send an MTM.

To send an MTM, you'll need to find your "template namespace" in the WhatsApp Business Manager. You need this so that you can identify which template you want to send, as the same template name could be used by other people around the world.

When sending your message, you'll need to concatenate the template namespace with the template name, using `:` as a separator. For example, let's say that your template namespace is `abc123` and your template name is `sample_issue_resolution`. You would then concatenate these two values together to get `abc123:sample_issue_resolution` as your full template name which you'll pass to Vonage.

As an example, let's take one of the sample message templates that WhatsApp provide for us. The template is called `sample_issue_resolution` and has the following structure:

```text
Hi {{1}}, were we able to solve the issue that you were facing?
```

This template allows us to pass the user's name as a single parameter. So, if we wanted to send this message to the user, we would need to do the following:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppTemplate;
use Vonage\Messages\MessageObjects\TemplateObject;

$toNumber = '447123456789';
$locale = 'en_US';
$templateName = 'abc_123:sample_issue_resolution';
$templateParams = [
	'John',
];

$whatsAppMessage = new WhatsAppTemplate(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	templateObject: new TemplateObject(
    	name: $templateName,
    	parameters: $templateParams,
	),
	locale: $locale,
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running this code would result in a message being sent to the phone number `+447123456789` that looks like this:

```text
Hi John, were we able to solve the issue that you were facing?
```

As you can see from the code example, we first need to create a new `WhatsAppTemplate` object and specify:

* The phone number the template is being sent to.
* The phone number the template is being sent from.
* The template object, which contains the template name and parameters.
* The locale of the template.

Once we've created our `WhatsAppTemplate` object, we can then send it using the Vonage `Client`.

You may have also noticed that we are passing a `locale` parameter to the `WhatsAppTemplate` object. This is because WhatsApp allows you to create message templates in multiple languages. For example, if we had the same template in both English and Spanish, we could send the English version by passing `en_US` as the locale (as shown in the example above), and the Spanish version by passing `es` as the locale. So if we passed `es` as the locale, the user would receive the following message instead:

```text
Hola, Ashley. ¿Pudiste solucionar el problema que tenías?
```

This is a really powerful feature that can be helpful if you need to provide international communication from your application. For example, you could this for things like providing support for your users in multiple languages.

### Sending WhatsApp Text Messages

Assuming that you are within your 24-hour customer care window, you can also send free-form text messages from your Laravel application. To do this, you'll need to create a new `WhatsAppText` object and pass it to the Vonage `Client`:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppText;

$toNumber = '447123456789';
$message = 'Hello, this is a test message.';

$whatsAppMessage = new WhatsAppText(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	text: $message,
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running the above code would result in a WhatsApp message being sent to the phone number `+447123456789` that looks like this:

```text
Hello, this is a test message.
```

### Sending WhatsApp Image Messages

Using Vonage, you are also able to send images to your users. To do this, you'll need to create a new `WhatsAppImage` object and pass it to the Vonage `Client`:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppImage;
use Vonage\Messages\MessageObjects\ImageObject;

$toNumber = '447123456789';
$imageUrl = 'https://example.com/image.png';
$caption = 'This is an example image.';

$whatsAppMessage = new WhatsAppImage(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	image: new ImageObject(
    	url: $imageUrl,
    	caption: $caption,
	),
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running the following code would result in a WhatsApp message being sent to the phone number `+447123456789` that contains an image with the caption "This is an example image.".

### Sending WhatsApp Video Messages

You can also send video messages from your Laravel application. To do this, you'll need to create a new `WhatsAppVideo` object and pass it to the Vonage `Client`:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppVideo;
use Vonage\Messages\MessageObjects\VideoObject;

$toNumber = '447123456789';
$videoUrl = 'https://example.com/video.mp4';

$whatsAppMessage = new WhatsAppVideo(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	videoObject: new VideoObject(
    	url: $videoUrl,
	),
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running the following code would result in a WhatsApp message being sent to the phone number `+447123456789` that contains a video.

### Sending WhatsApp Audio Messages

Using Vonage, you can also send audio messages from your Laravel application. To do this, you'll need to create a new `WhatsAppAudio` object and pass it to the Vonage `Client`:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppAudio;
use Vonage\Messages\MessageObjects\AudioObject;

$toNumber = '447123456789';
$audioUrl = 'https://example.com/audio.mp3';

$whatsAppMessage = new WhatsAppAudio(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	audioObject: new AudioObject(
    	url: $audioUrl,
	),
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running the following code would result in a WhatsApp message being sent to the phone number `+447123456789` that contains an audio file.

### Sending WhatsApp File Messages

You may also want to send files to your users from your Laravel application. To do this, you'll need to create a new `WhatsAppFile` object and pass it to the Vonage `Client`:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppFile;
use Vonage\Messages\MessageObjects\FileObject;

$toNumber = '447123456789';
$fileUrl = 'https://example.com/file.pdf';
$caption = 'This is an example file.';

$whatsAppMessage = new WhatsAppFile(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	fileObject: new FileObject(
    	url: $fileUrl,
    	caption: $caption,
	),
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running the following code would result in a WhatsApp message being sent to the phone number `+447123456789` that contains a file with the caption "This is an example file.".

### Sending WhatsApp Location Messages

In some scenarios (such as providing directions), you may also want to send locations to your users. To do this, you'll need to create a new `WhatsAppLocation` object and pass it to the Vonage `Client`:

```php
use Vonage\Client;
use Vonage\Messages\Channel\WhatsApp\WhatsAppCustom;

$toNumber = '447123456789';
$longitude = -40.34764;
$latitude = -74.18875;
$name = Vonage HQ';
$address = '23 Main Street, Holmdel, NJ 07733';

$whatsAppMessage = new WhatsAppCustom(
	to: $toNumber,
	from: config('services.vonage.whatsapp.from_phone_number'),
	custom: [
    	'type' => 'location',
    	'location' => [
        	'longitude' => $longitude,
        	'latitude' => $latitude,
        	'name' => $name,
        	'address' => $address,
    	],
	],
);

app(Client::class)
	->messages()
	->send($whatsAppMessage);
```

Running the following code would result in a WhatsApp message being sent to the phone number `+447123456789` that contains a location with the name "Vonage HQ" and the address "23 Main Street, Holmdel, NJ 07733".

## Conclusion

Hopefully, this article should have given you an insight into how you can send different types of WhatsApp messages from your Laravel application using Vonage. You should also have an understanding of the WhatsApp message templates and the 24-hour window that you have to send messages.