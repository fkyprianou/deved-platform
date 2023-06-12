---
title: Evolving the Vonage Laravel Helpdesk with OpenAI
description: In part one of this series, we introduced Helpdesk with SMS. Now,
  we move on to Voice
thumbnail: /content/blog/evolving-the-vonage-helpdesk-with-laravel-deepgram/laravel-vonage-helpdesk.png
author: james-seconde
published: true
published_at: 2023-05-30T09:58:49.801Z
updated_at: 2023-05-30T09:58:49.831Z
category: tutorial
tags:
  - laravel
  - voice-api
  - open-ai
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
This is the second part of a continuous series about an evolving [Laravel](https://laravel.com/) application that leverages Vonage APIs to replicate common real-world use cases.

[In the first part](https://developer.vonage.com/en/blog/introducing-the-laravel-vonage-helpdesk), we created a new Laravel application, pulled in the [vonage-laravel](https://github.com/Vonage/vonage-laravel) library, and created a Helpdesk ticket view. The customer selects their chosen communication method (in this case, it was SMS only) so that any message posted by an admin would be sent to the customer's mobile phone. Replying to the message would then be added using incoming webhooks and written to the ticket conversation.

In this article, we're going to add the capability of using Vonage's Text-to-speech (TTS) Voice capabilities using the [Voice API](https://developer.vonage.com/en/voice/voice-api/overview), with the ability of the customer to speak a response that gets transcribed back to the ticket conversation.

### Prerequisites

We'll assume that the first tutorial has been completed, which will give us:

* The [helpdesk](https://github.com/Vonage-Community/sample-messages_voice-php-helpdesk) repository, cloned locally from GitHub
* [Laravel Sail](https://laravel.com/docs/10.x/sail) up and running, to dockerise the local development environment
* [Migrations run](https://laravel.com/docs/10.x/migrations)
* [Vite](https://vitejs.dev/) development server running to build [Laravel Breeze](https://laravel.com/docs/10.x/starter-kits)'s boilerplate assets
* [Ngrok](https://ngrok.com/) running locally, and a Vonage application instance configured to send webhooks to it

### How Does It Do That? Part 2: Voice

OK, so it's time to go through Voice capabilities. The flow of how conversations happen here is exactly the same: you create a new ticket as a customer and it opens the conversation view. However, when we create it this time, we're going to set up the ticket as a voice conversation. 

### Enabling Voice

Before this works, we will need a Voice-enabled application ID in the [Vonage Dashboard](https://dashboard.nexmo.com/). You can edit the previous application from the last tutorial or create a new one. Enabling the application ID is all you need: don't worry about using the UI to send webhooks to the correct local route (we'll go through this later) as the code constructs the webhook reply URL for you (this is different from how we've set up SMS, and I'll go through why when we review the code later in the article).

### What is OpenAI?

The most common product you may have heard of in association with this name is [ChatGPT](https://chat.openai.com/).

ChatGPT is a cutting-edge language model developed by [OpenAI](https://openai.com/), designed to engage in natural language conversations with users. As an AI-based assistant, ChatGPT can provide information, answer questions, and assist with various tasks. It uses deep learning techniques to understand and generate human-like text, making the interactions feel more personalized.

Yes, that paragraph was written by ChatGPT. But what you might not know is the company behind it, OpenAI, has [several other products](https://openai.com/product) that can all be accessed via. their API. One such product is [Whisper](https://platform.openai.com/docs/guides/speech-to-text), which we're going to use to transcribe what a customer says as a recorded message in response to a ticket entry from the Helpdesk app.

### Setting up the OpenAI API

Firstly, you'll need an OpenAI account. [Follow this link to create an account](https://auth0.openai.com/u/signup), and when you're done you'll need to head to 'Manage Account' under your top right hand profile menu. Opening this up will show you the following screen - head to API Keys and set up a new key. The end result should look something like this:

![](/content/blog/evolving-the-vonage-laravel-helpdesk-with-openai/screenshot-2023-06-02-at-10.22.45.png)

When you create the key, you'll have one chance to copy it - make sure you do that.

We need to add that secret to our `env` file. You can see in the `example.env` file in the repo that we have a placeholder for it:

![Screenshot of Helpdesk's environment variables example file](/content/blog/evolving-the-vonage-laravel-helpdesk-with-openai/screenshot-2023-06-02-at-10.43.28.png)

I've included the others in the screenshot because it's important to note that this feature won't work without all of these environment variables set:

* `VONAGE_SMS_FROM` is re-used as the outbound calling number
* `PUBLIC_URL` is your [Ngrok](https://ngrok.com/) (or any other tool such as Beyond Code's [Expose](https://expose.dev/docs/introduction)) public-facing address. This is essential, as the code will stitch together the response URL to the API when making a call
* `VONAGE_APPICATION_ID` and `VONAGE_PRIVATE_KEY`. In the last tutorial, we could have used basic authentication, but for webhooks to work, they need to be tied to an application ID. For using the Vonage Voice API, we have to have a private key and application ID, which the Vonage PHP SDK will use to generate and handle JWT authorisation for us.

### Under the Hood

The functionality we're going to look at lives in the `update()` method in the `TicketController`. We only want to make an outgoing call if the ticket is being updated by an admin user (as opposed to the customer), and the customer has chosen voice as their communication preference.

```php
if ($userTicket->notification_method === 'voice') {
    $currentHost = config('helpdesk.public_url');
    $outboundCall = new OutboundCall(
        new Phone($userTicket->phone_number),
        new Phone(config('vonage.sms_from'))
    );
    $outboundCall
        ->setAnswerWebhook(
            new Webhook($currentHost . '/webhook/answer/' . $ticketEntry->id, Webhook::METHOD_GET)
        )
        ->setEventWebhook(
            new Webhook($currentHost . '/webhook/event/' . $ticketEntry->id, Webhook::METHOD_POST)
        );
    Vonage::voice()->createOutboundCall($outboundCall);
}
```

Here's a synopsis of what the code is doing here:

* We know we want to make an outbound call in this logic block, so we create a new `OutboundCall` that pulls in the customers' phone number from the ticket, and the sending number from the config.
* This is the neat bit. You know how in part one of this tutorial, we set an Ngrok URL in the Vonage Dashboard for SMS webhooks? We've not done that here, because each call using the Voice SDK can be configured to *use a specific callback URL for this call we're making*. This part is really important because it *allows us to configure state.* In this case, we take the Ngrok public URL from `$currentHost` (i.e. the constant `PUBLIC_URL`), a route defined by us for our application (`/webhook/answer/`) and the key to making this work: the ticket entry ID as part of the route. Later, in the `WebhookController` we can pull the parent ticket out, plus the owner of that ticket.

So, now we need a new controller to handle what comes in when the customer has completed their ticket call. The two parts to this are:

* Read out a response when the customer picks up their phone call (this will be set in the controller assigned to the route).
* Have a route to read incoming answer events (we set these when setting up the outbound call).
* From the recording event generated after the call is complete, fetch a voice recording of the customers' response, transcribe it using OpenAI, and write it as a new `TicketEntry.`

Phew! Quite a bit to digest here, so let's start tucking in:

### Using NCCOs to TTS

NCCOs are JSON payloads that instruct Vonage services what 'to do' i.e. an action, record something etc. When the customer picks up the phone, we want to read them the latest ticket update made by the admin, then give them a prompt to answer it. Here is the route:

```php
Route::post('/webhook/answer/{ticketEntry:id}', [WebhookController::class, 'answer'])->name('voice.answer');
```

The route points to `WebhookController::answer()`, so our TTS response looks like this:

```php
public function answer(TicketEntry $ticketEntry): JsonResponse  
{  
    if (!$ticketEntry->exists) {  
        return response()->json([  
            [                'action' => 'talk',  
                'text' => 'Sorry, there has been an error fetching your ticket information'  
            ]  
        ]);    }  
    return response()->json([  
        [            'action' => 'talk',  
            'text' => 'This is a message from the Vonage Helpdesk'  
        ],  
        [            'action' => 'talk',  
            'text' => $ticketEntry->content,  
        ],        [            'action' => 'talk',  
            'text' => 'To add a reply, please leave a message after the beep, then press the pound key',  
        ],        [            'action'    => 'record',  
            'endOnKey'  => '#',  
            'beepStart' => true,  
            'eventUrl' => [config('helpdesk.public_url') . '/webhook/recordings/' .  $ticketEntry->id]  
        ],        [            'action' => 'talk',  
            'text' => 'Thank you, your ticket has been updated.',  
        ]    ]);}
```

Each array gives a payload of instructions that are fairly straightforward, but the important glue here to answer the question "How do we capture the customer's response?" is in the `record` action. You can see it gives a `beepStart` prompt, and most importantly we define the behaviour after the call is complete. The `eventUrl` will be hit with a webhook that will contain a URL of this recording.

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

And here is our route:

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
    Storage::put('call_recording.mp3', $audio);  
  
    $ticketContent = $this->transcribeRecordingOpenAi();  
  
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

This uses Route Model Binding to take the relevant `TicketEntry` and inject it as a dependency, then pulls out the `recording_url`. The Vonage SDK has a useful method named `getRecording()` that will return a `StreamInterface` contained in the body. 

For security reasons, you cannot write out the stream from the audio straight to the OpenAI request we will send for transcription, so we need to save the file temporarily. Once we've saved it, we can use the `Storage` facade to read it back out during the transcription request and then delete it.

The `transcribeRecording()` is a custom method in this class controller that we'll get to in a moment, but assuming that a string comes back from the transcription, we create a new `TicketEntry`, associate it with the ticket owner (we know this is the customer as it's an incoming webhook route), and save it to the `Ticket`.

### OpenAI Transcription

This is the last part - getting our transcription. There are ways to do this asynchronously, but I've opted to do this in a synchronous way to keep it simpler. If you want to implement this in an async way (after all, this is data processing so it's good practice to do so), you can use a Laravel Job Queue worker, but be warned that you could well run into race condition problems (which I have done in the past).

The transcription in the method controller is handled by the `transcribeRecording()` function, so let's take a look at that:

```php
public function transcribeRecordingOpenAi(): string  
{  
    $client = new Client([  
        'base_uri' => 'https://api.openai.com/v1/',  
    ]);  
    
    $audioPath = Storage::path('call_recording.mp3');  
  
    $multipart = new MultipartStream([  
        [            'name'     => 'file',  
            'contents' => fopen($audioPath, 'rb'),  
            'filename' => basename($audioPath),  
        ],        [            'name'     => 'model',  
            'contents' => 'whisper-1',  
        ],    ]);  
    $response = $client->request('POST', 'audio/transcriptions', [  
        'headers' => [  
            'Authorization' => 'Bearer ' . config('helpdesk.open_ai_secret'),  
            'Content-Type'   => 'multipart/form-data; boundary=' . $multipart->getBoundary(),  
        ],        'body' => $multipart,  
    ]);  
    Storage::delete('call_recording.mp3');  
  
    $responseBody = json_decode($response->getBody()->getContents(), true, 512, JSON_THROW_ON_ERROR);  
  
    return $responseBody['text'];  
}
```

This has been hacked together for demonstration reasons, so firstly I'd like to make it clear that if your application has a dependency on a 3rd party API such as this one (or Vonage), you should wrap this client and its configuration [as a service provider](https://laravel.com/docs/10.x/providers).

The method creates a new [Guzzle](https://docs.guzzlephp.org/en/stable/) Client, and prepares the request as a `MultipartStream`, as OpenAI requires the request to be an encoded form. We set the base url and fetch our temporary file we created earlier (`call_recording.mp3`). We can now use `fopen()` to write the file out, and then delete it after the request has been completed. 

All being well, you'll get a transcription array back, which will contain the key `text`, which is sent back for updating the `TicketEntry`. Congratulations: we now have a working TTS ticketing system!

### And, Stay Tuned

There are two more tutorials forthcoming, so keep an eye out for them. None of our code is tested, so [PEST](https://pestphp.com/) will come to the rescue for that. We also are using plain [Blade](https://laravel.com/docs/10.x/blade) templates - what if we want to instantly update the page as if it was a JavaScript Single-Page-Application (SPA)? Well, we have [Livewire](https://laravel-livewire.com/) for that. Until the next time, enjoy helping your customers!