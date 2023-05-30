---
title: Evolving the Vonage Helpdesk with Laravel & Deepgram
description: In part one of this series, we introduced Helpdesk with SMS. Now,
  we move onto Voice
thumbnail: /content/blog/evolving-the-vonage-helpdesk-with-laravel-deepgram/laravel-vonage-helpdesk.png
author: james-seconde
published: true
published_at: 2023-05-30T09:58:49.801Z
updated_at: 2023-05-30T09:58:49.831Z
category: tutorial
tags:
  - php
  - laravel
  - voice-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
This is the second part to a continuous series about an evolving Laravel application that leverages Vonage APIs to replicate common real-world use cases.

In the first part, we created a new Laravel application, pulled in the vonage-laravel library and created a Helpdesk ticket view, where the customer selects their chosen communication method (in this case, it was SMS only), so that any message posted by an admin would be sent to the customers' mobile phone. Replying to the message would then be read in using webhooks and written to the ticket conversation.

In this article, we're going to add the capability of using Vonage's Text-to-speech (TTS) Voice capabilities using the Voice API, with the ability of the customer to speak a response that gets transcribed back to the ticket conversation.

### Prerequisites

We'll assume that the first tutorial has been completed, which will give us:

* The helpdesk repository, cloned locally from Github
* Laravel Sail up and running, to dockerise the local development environment
* Migrations run
* Vite development server running to build Laravel Breeze's boilerplate assets
* Ngrok running locally, and a Vonage application instance configured to send webhooks to it

### How does it do that? Part 2: Voice

OK, so it's time to go through Voice capabilities. The flow of how conversations happen here is exactly the same: you create a new ticket as a customer and it opens the conversation view. However, when we create it this time, we're going to set up the ticket as a voice conversation. Head to your preferences in the dashboard, and change the notification method to Voice.

![Helpdesk user settings with Voice chosen as notification method](/content/blog/evolving-the-vonage-helpdesk-with-laravel-deepgram/pasted-image-20230526154132.png)

This switch is all there is to it from the frontend perspective, but we have two important steps to complete first to make this work.

### Setting up Voice Webhooks

Before this will work, we're going to need a Voice-enabled application ID in the Vonage Dashboard. You can either edit the previous application from the last tutorial or create a new one. Enabling the application ID is all you need: don't worry about using the UI to send webhooks to the correct local route (we'll go through this later) as the code constructs the webhook reply URL for you (this is different from how we've set up SMS, and I'll go through why later).

### What is Deepgram?

Deepgram is a product/company that focuses on automatic speech recognition (ACR). This can be used for transcribing (which is what we're going to do), but also for cases where real-time, low latency responses are needed (i.e. live captioning). What makes it quite interesting is that is uses Machine Learning through neural networks to improve transcriptions. Deepgram is going to be handling our transcriptions, so let's get to it.

### Setting up Deepgram

Setting up a Deepgram account is free, and you'll have a decent amount of starting credit added. Follow this guide for setting up your API secret. Once you have your key, you should be able to access it from this screen:

![Deepgram dashboard showing an API key](/content/blog/evolving-the-vonage-helpdesk-with-laravel-deepgram/screenshot-2023-05-26-at-21.12.01.png)

Once you've created your key, we need to add it to our `env` file. You can see in the `example.env` file in the repo that we have a placeholder for it:

![Screenshot from code editor of example environment variables required by Helpdesk](/content/blog/evolving-the-vonage-helpdesk-with-laravel-deepgram/pasted-image-20230526211505.png)

I've included the others in the screenshot because it's important to note that this feature won't work without all of these environment variables set:

* `VONAGE_SMS_FROM` is used as the outbound calling number (yes, it has SMS in it, I should change that!)
* `PUBLIC_URL` is your ngrok (or any other tool such as Beyond Code's Expose) public facing address. This is essential, as the code will stitch together the response URL to the API when making a call
* `VONAGE_APPICATION_ID` and `VONAGE_PRIVATE_KEY`. In the last tutorial, we could have used basic authentication, but for webhooks to work they need to be tied to an application ID. For using the Vonage Voice API, we have to have a private key and application ID, which the Vonage PHP SDK will use to generate and handle JWT authorisation for us.

### Under the Hood

The functionality we're going to look at lives in the `update()` method in the `TicketController`. We only want to make an outgoing call if the ticket is being updated by an admin user (as opposed to the customer), and the customer has chosen voice as their communication preference.

```
if ($userTicket->notification_method === 'voice') {  
    $currentHost = env('PUBLIC_URL', url('/'));  
    $outboundCall = new OutboundCall(  
        new Phone($userTicket->phone_number),  
        new Phone(config('vonage.sms_from'))  
    );
    
    $outboundCall  
        ->setAnswerWebhook(  
            new Webhook($currentHost . '/webhook/answer/' . $ticketEntry->id)  
        )        ->setEventWebhook(  
            new Webhook($currentHost . '/webhook/event/' . $ticketEntry->id)  
        );    Vonage::voice()->createOutboundCall($outboundCall);  
}
```

We'll skip over the fact that I'm calling the `env()` method (never do this in Laravel on account of config caching, I left this in on purpose to demonstrate this common mistake) and instead explain what the Vonage PHP SDK code is doing.

* We know we want to make an outbound call in this logic block, so we create a new `OutboundCall` that pulls in the customers' phone number from the ticket, and the sending number from the config.
* This is the neat bit. You know how in part one of this tutorial, we set an ngrok URL in the Vonage Dashboard for SMS webhooks? We've not done that here, because each call using the Voice SDK can be configured to *use a specific callback URL for this call we're making*. This part is really important, because it *allows us to configure state.* In this case, we take the ngrok public URL from `$currentHost` (i.e. the constant `PUBLIC_URL`), a route defined by us for our application (`/webhook/answer/`) and the key to making this work: the ticket ID as part of the route.

So, now we need a new controller to handle what comes in when the customer has competed their ticket call. The two parts to this are:

* Read out a response when the customer picks up their phone call (this will be set in the controller assigned to the route)
* Have a route to read incoming events (we've set the event webhook in the voice call, you can read more about events here)
* From the event, fetch a voice recording of the customers' response, transcribe it using Deepgram and write it as a new `TicketEntry`.

Phew! Quite a bit to digest here, so let's start tucking in:

### Using NCCOs to TTS

NCCOs are JSON payloads that instruct Vonage services what 'to do' i.e. an action, record something etc.  When the customer picks up the phone, we want to read them the latest ticket update made by the admin, then give them a prompt to answer it. Here is the route:

```php
Route::post('/webhook/answer/{ticketEntry:id}', [WebhookController::class, 'answer'])->name('voice.answer');
```

The route points to `WebhookController::answer()`, so our TTS response looks like this:

```php
public function answer(TicketEntry $ticketEntry): JsonResponse
{
    if (!$ticketEntry->exists) {
        return response()->json([
            [
                'action' => 'talk',
                'text' => 'Sorry, there has been an error fetching your ticket information'
            ]
        ]);
    }

    return response()->json([
        [
            'action' => 'talk',
            'text' => 'This is a message from the Vonage Helpdesk'
        ],
        [
            'action' => 'talk',
            'text' => $ticketEntry->content,
        ],
        [
            'action' => 'talk',
            'text' => 'To add a reply, please leave a message after the beep, then press the pound key',
        ],
        [
            'action' => 'record',
            'endOnKey' => '#',
            'beepStart' => true,
            'eventUrl' => [env('PUBLIC_URL') . '/webhook/recordings/' . $ticketEntry->id]
        ],
        [
            'action' => 'talk',
            'text' => 'Thank you, your ticket has been updated.',
        ]
    ]);
}
```

Each array gives a payload of instructions that are fairly straightforward, but the important glue here to answer the question "how do we capture the customer's response?" is in the `record` action. You can see it gives a `beepStart` prompt, and most importantly we define the behaviour after the call is complete. The `eventUrl` will be hit with a webhook that will contain a URL of this recording.

### Processing the Recording

Our next route is the one that will contain a link to the customer's response as a recorded MP3, as well as the ticket ID so we know what entity it belongs to. Here's an example payload we can expect to receive:

```json
{
  "start_time": "2020-01-01T12:00:00.000Z",
  "recording_url": "https://api.nexmo.com/v1/files/bbbbbbbb-aaaa-cccc-dddd-0123456789ab",
  "size": 12222,
  "recording_uuid": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
  "end_time": "2020-01-01T12:00:00.000Z",
  "conversation_uuid": "CON-aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
  "timestamp": "2020-01-01T12:00:00.000Z"
}
```

Here is our route:

```php
Route::post('/webhook/recordings/{ticketEntry:id}', [WebhookController::class, 'recording'])->name('voice.recording');
```

And the `recording()` method to handle it:

```php
public function recording(TicketEntry $ticketEntry, Request $request): Response|Application|ResponseFactory  
{  
    $params = $request->all();  
    Log::info('Recording event', $params);  
  
    $audio = Vonage::voice()->getRecording($params['recording_url']);  
    $ticketContent = $this->transcribeRecording($audio);  
  
    $newTicketEntry = new TicketEntry([  
        'content' => $ticketContent,  
        'channel' => 'voice',  
    ]);  
    $parentTicket = $ticketEntry->ticket()->get()->first();  
    $newTicketEntryUser = $parentTicket->user()->get()->first();  
    $newTicketEntry->user()->associate($newTicketEntryUser);  
    $newTicketEntry->ticket()->associate($parentTicket);  
    $newTicketEntry->save();  
  
    return response('', 204);  
}
```

This uses Route Model Binding to take the relevant `TicketEntry` and inject it as a dependency, then pulls out the `recording_url`. The Vonage SDK has a useful method named `getRecording()` that will return a `StreamInterface` contained in the body. The `transcribeRecording()` is a custom method in this class controller that we'll get to in a moment, but assuming that a string comes back, we create a new `TicketEntry`, associate it with the ticket owner (we know this is the customer as it's an incoming webhook route) and save it to the `Ticket`

### Deepgram Transcription

This is the last part - getting our transcription. There are ways to do this asynchronously, but I've opted to do this in a synchronous way to keep it simpler. If you want to implement this in an async way (after all, this is data processing so it's good practise to do so), you can use a Laravel Job Queue worker, but be warned that you could well run into race condition problems (which I did).

The transcription in the method controller is handled by the `transcribeRecording()` function, so let's take a look at that:

```php
public function transcribeRecording($audio)  
{  
    $client = new GuzzleHttp\Client([  
        'base_uri' => 'https://api.deepgram.com/v1/'  
    ]);  
  
    $transcriptionResponse = $client->request(  
        'POST',  
        'listen?punctuate=true',  
        [            'headers' => [  
                'Authorization' => 'Token ' . env('DEEPGRAM_API_SECRET'),  
                'Content-Type' => 'audio/mpeg',  
            ],            'body' => $audio  
        ]);  
  
    if ($transcriptionResponse->getStatusCode() !== 200) {  
        Log::error('Transcription service failed, check your credentials');  
        return false;  
    }  
    $transcriptionResponseBody = json_decode($transcriptionResponse->getBody(), true);  
    Log::info($transcriptionResponseBody);  
    $transcription = $transcriptionResponseBody['results']['channels'][0]['alternatives'][0]['transcript'];  
  
    Log::info('Voice Response', [$transcription]);  
  
    return $transcription;  
}
```

This has been hacked together for demonstration reasons, so firstly I'd like to make it clear that if your application has a dependency on a 3rd party API such as this one (or Vonage), you should wrap this client and it's configuration as a service provider.

The method creates a new Guzzle Client, sets the Deepgram Token into the header and POSTs the `StreamInterface` object. All being well, you'll get a transcription data structure back which we parse, and send back to  update the ticket contents. Congatulations, we now have a working TTS ticketing system!