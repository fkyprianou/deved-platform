---
title: How to Handle Inbound Phone Calls with Node.js
description: Learn how to handle Inbound Phone Calls with Node.js
author: michael-crump
published: true
published_at: 2023-02-22T22:21:14.829Z
updated_at: 2023-02-22T22:21:14.894Z
category: tutorial
tags:
  - voice
  - node
  - apis
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

The [Vonage Voice API](https://developer.vonage.com/en/voice/voice-api/overview) is the easiest way to build high-quality voice applications in the Cloud. With the Voice API, you can:

* Build apps that scale with the web technologies you are already using.
* Control the flow of inbound and inbound calls in JSON with [Nexmo Call Control Objects](https://developer.vonage.com/en/voice/voice-api/overview#ncco) (NCCO). (Note: Nexmo is now Vonage).
* Record and store inbound or inbound calls
* Create conference calls
* [Send text-to-speech messages](https://developer.vonage.com/en/blog/how-to-make-an-outbound-text-to-speech-phone-call-with-node-js) in 40 languages with different genders and accents.
* And more! 

In this tutorial, you will learn how to receive inbound calls by implementing a webhook using Node.js.

## Prerequisites

Before you begin, make sure you have the following:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. 
* [OPTIONAL - Vonage CLI](https://www.npmjs.com/package/@vonage/cli) Once Node.js is installed, install the CLI by typing `npm install -g @vonage/cli`. This tool allows you to create and manage Vonage applications from a command-line interface vs. the Vonage Developer Portal.

## Create a Voice-Enabled Vonage Application

To use the [Vonage Voice API](https://developer.vonage.com/voice/voice-api/overview), you must create a [Vonage Application](https://developer.vonage.com/application/overview) from the developer portal. If you don't have an account, then go ahead and create one, as we provide credits to get started. No credit card is required. 

We will need to configure the application's webhooks and more. Please note that this can be accomplished through the [Vonage Developer Portal](https://developer.vonage.com/) or the [Vonage CLI](https://developer.vonage.com/application/vonage-cli). For this tutorial, we'll use the Vonage Developer Dashboard. 

After creating an account, log into the Vonage Developer Dashboard, look for the [Application section](https://dashboard.nexmo.com/applications), and create a new application. Give your application a name, such as **InboundCall**.

![Creating the application](/content/blog/how-to-handle-inbound-phone-calls-with-node-js/inboundcall.png "InboundCall.png")

Scroll down the page and ensure that the **Voice** capability is toggled on. 

Make a note of the **Answer**, and **Event** URLs, as we will fill those in shortly. We will leave the **Fallback URL** blank. 

![Turning on the Voice Capability](/content/blog/how-to-handle-inbound-phone-calls-with-node-js/voicecapability.png "VoiceCapability.png")

Press **Generate new application** at the bottom of the page to continue. 

This tutorial also uses a virtual phone number. To purchase one, go to **Numbers** > **Buy Numbers** and search for one that meets your needs. Once you have a number, link it to the Vonage Developer Dashboard, as shown below.

![Adding a number to the application](/content/blog/how-to-handle-inbound-phone-calls-with-node-js/linkednumber.png "LinkedNumber.png")

Next, We'll use ngrok to expose our webhook endpoints on our local machine as a public URL.

## Run ngrok

ngrok is a cross-platform application that enables developers to expose a local development server to the Internet with minimal effort. We'll be using it to expose our service to the Internet. Once you have ngrok setup and are logged in (again, the free account is acceptable), then run the following command:

```
ngrok http 4001
```

After ngrok runs, it will give you a **Forwarding** URL that we'll use as the base for our Webhooks later in the article. Mine looks like the following:

![ngrok running successfully](/content/blog/how-to-handle-inbound-phone-calls-with-node-js/ngrok.png "ngrok.png")

Remember the **Answer** and **Event** URLs in the Vonage Developer Portal, as mentioned earlier? We will need to use the ngrok URL and fill in each field, appending `/answer` and `/event`, for the Answer URL and Event URL.

![ngrok running successfully](/content/blog/how-to-handle-inbound-phone-calls-with-node-js/webhooksection.png "webhooksection.png")

When you call the number associated with the application, and it answers, the webhook defined in the **Answer URL** triggers. Likewise, events are logged with a POST request and are triggered upon calling the number or if the number is busy, etc. 

## Node.js project setup

Now that we have created our Vonage Voice Application inside the developer dashboard, let’s look at how we should configure our Node.js application. 

Begin by going to a command/terminal prompt, creating a working directory, and initializing a Node.js project.

```bash
npm init -y
```

We will handle the requests with [Express](https://expressjs.com/) and use [body-parser](https://www.npmjs.com/package/body-parser) to parse incoming request bodies. Install both of these with:

```javascript
npm install express body-parser --save
```

Create an `index.js` file, instantiate express, and listen to the server to port 4001. Because you have set your ngrok to expose `localhost:4001`, you must stick with the same port.

```javascript
'use strict'
const app = require('express')();
const bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
const server = app.listen(process.env.PORT || 4001, () => {
  console.log('Express server listening on port %d in %s mode', server.address().port, app.settings.env);
});
```

Let’s define the endpoint for the Answer URL as `/answer` and the Event URL as `/event`.

Create an HTTP GET route to handle the requests for `/answer` to retrieve your NCCO:

```javascript
app.get('/answer', function (req, res) {
  const ncco = [
    {
      action: 'talk',
      voiceName: 'Jennifer',
      text: 'Hello, thank you for calling. This is the Jennifer voice from Vonage.'
    }
  ];
  res.json(ncco);
});
```

Define your text to be read by a synthesized voice in JSON (or JavaScript object, in this case). You can customize the NCCO object with [optional params](https://docs.nexmo.com/voice/voice-api/ncco-reference) with various agents by language, gender, and even accent.

The endpoint for the `event_url` needs to be POST, so let’s define `/event`:

```javascript
app.post('/event', function (req, res) {
  console.log(req.body);
  res.status(204).end();
});
```

Keep in mind that we are going to monitor the status of our terminal. View this [code sample](https://github.com/Vonage/vonage-node-code-snippets/blob/master/voice/receive-call-webhook-failover.js) for an example of a fallback that handles things such as a timeout, if the number is busy, unanswered, etc. 

## Run the Application

Enter the following at your command/terminal prompt to run the application:

```javascript
node index.js
```

Let's make a phone call to see if your application works! Call your virtual number from your physical phone. If everything works, you should hear the message you have defined in your NCCO.

Also, see your terminal to check the status of your call. Below is a sample of what mine looks like: 

```text
{
  headers: {},
  from: '19999999999',
  to: '19999999999',
  uuid: '912e...',
  conversation_uuid: 'CON-07f...',
  status: 'ringing',
  direction: 'inbound',
  timestamp: '2023-02-22T19:27:47.276Z'
}
```

The status will change depending on what event the call is currently in. For example, we start with `ringing`, then go to `started`, then `answered`, and finally `completed` if the call went through successfully.

## Wrap-up

Now that you have created an inbound call with the Vonage Voice API and Node.js, why not learn how to [send text-to-speech messages](https://developer.vonage.com/en/blog/how-to-make-an-outbound-text-to-speech-phone-call-with-node-js) in over 40 languages? You could also learn more about our [Voice API](https://developer.vonage.com/en/voice/voice-api/overview) and dive into several awesome [code snippets](https://developer.vonage.com/en/voice/voice-api/overview#code-snippets). 

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!