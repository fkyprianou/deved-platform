---
title: Build an Interactive Voice Response Menu using Node.js and Express
description: This tutorial shows you how to build interactive voice response
  menus for your application using Vonage, Node.js and the Express framework.
author: michael-crump
published: true
published_at: 2023-03-13T22:21:34.323Z
updated_at: 2023-03-13T22:21:34.377Z
category: tutorial
tags:
  - voice-api
  - javascript
  - pbx
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

The [Vonage Voice API](https://developer.vonage.com/en/voice/voice-api/overview) is the easiest way to build high-quality voice applications in the Cloud. 

In this tutorial, we will build an interactive voice response menu using the Vonage Voice API and Node.js to receive inbound calls and capture user input via the keypad.

For reference, the source file should look something like [this one](https://github.com/Nexmo/nexmo-node-code-snippets/blob/main/voice/ivr-menu.js) once this exercise is complete.

## Prerequisites

Before you begin, make sure you have the following installed:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. 
* [OPTIONAL - Vonage CLI](https://www.npmjs.com/package/@vonage/cli) Once Node.js is installed, install the CLI by typing `npm install -g @vonage/cli`. This tool allows you to create and manage Vonage applications from a command-line interface vs. the Vonage Developer Portal.

## Create a Voice-Enabled Vonage Application

To use the [Vonage Voice API](https://developer.vonage.com/voice/voice-api/overview), you must create a [Vonage Application](https://developer.vonage.com/application/overview) from the developer portal. If you don't have an account, then go ahead and create one, as we provide credits to get started. No credit card is required. 

<sign-up></sign-up>

We will need to configure the application's webhooks and more. Please note that this can be accomplished through the [Vonage Developer Portal](https://developer.vonage.com/) or the [Vonage CLI](https://developer.vonage.com/application/vonage-cli). For this tutorial, we'll use the Vonage Developer Dashboard. 

After creating an account, log into the Vonage Developer Dashboard, look for the [Application section](https://dashboard.nexmo.com/applications), and create a new application. Give your application a name, such as **IVRMenu**.

![Creating the application](/content/blog/build-an-interactive-voice-response-menu-using-node-js-and-express/ivrmenucall.png "IVRMenuCall.png")

Scroll down the page and ensure that the **Voice** capability is toggled on. 

Make a note of the **Answer**, and **Event** URLs, as we will fill those in shortly. We will leave the **Fallback URL** blank for the duration of this tutorial. 

![Turning on the Voice Capability](/content/blog/build-an-interactive-voice-response-menu-using-node-js-and-express/voicecapability.png "VoiceCapability.png")

Press **Generate new application** at the bottom of the page to continue. 

This tutorial also requires a virtual phone number. To purchase one, go to **Numbers** > **Buy Numbers** and search for one that meets your needs. Once you have a number, link it to the Vonage Developer Dashboard, as shown below.

![Adding a number to the application](/content/blog/build-an-interactive-voice-response-menu-using-node-js-and-express/linkednumber.png "LinkedNumber.png")

Next, We'll use ngrok to expose our webhook endpoints on our local machine as a public URL.

## Run ngrok

ngrok is a cross-platform application that enables developers to expose a local development server to the Internet with minimal effort. We'll be using it to expose our service to the Internet. Once you have ngrok setup and are logged in (again, the free account is acceptable), then run the following command:

```
ngrok http 3000
```

After ngrok runs, it will give you a **Forwarding** URL that we'll use as the base for our Webhooks later in the article. Mine looks like the following:

![ngrok running successfully](/content/blog/build-an-interactive-voice-response-menu-using-node-js-and-express/ngrok.png "ngrok.png")

Remember the **Answer** and **Event** URLs in the Vonage Developer Portal, as mentioned earlier? We will need to use the ngrok URL and fill in each field, appending `/answer` and `/event`, for the Answer URL and Event URL.

So, for example, my **Answer** URL is pointing towards https://9ebc-2601-600-9580-d650-fd8d-62dc-f5e4-b88e.ngrok.io/answer and **Event** towards https://9ebc-2601-600-9580-d650-fd8d-62dc-f5e4-b88e.ngrok.io/event.

![ngrok running successfully](/content/blog/build-an-interactive-voice-response-menu-using-node-js-and-express/webhooksection.png "webhooksection.png")

When you call the number associated with the application, and it answers, the webhook defined in the **Answer URL** triggers. Likewise, events are logged with a POST request and are triggered upon calling the number or if the number is busy, etc. 

## Node.js project setup

Now that we have created our Vonage Voice Application inside the developer dashboard let’s look at how we should configure our Node.js application. 

Begin by going to a command/terminal prompt, creating a working directory, and initializing a Node.js project.

```bash
npm init -y
```

We will handle the requests with [Express](https://expressjs.com/) and use [body-parser](https://www.npmjs.com/package/body-parser) to parse incoming request bodies. Install both of these with the following:

```javascript
npm install express body-parser --save
```

Create an `index.js` file, instantiate Express as well as body-parser.

```javascript
const app = require('express')()
const bodyParser = require('body-parser')

app.use(bodyParser.json())
```

Let’s define the endpoint for the Answer URL as `/answer` and the Event URL as `/event`.

When a user presses a number on their keypad, you can collect which button they pressed via DTMF (Dual Tone Multifrequency). Whenever a DTMF input is collected from the user, this is sent to our `/dtmf` handler, which we'll define in the next step.

Create an HTTP GET route to handle the requests for `/answer` to retrieve your NCCO:

```javascript
app.get('/answer', (req, res) => {
  const ncco = [{
      action: 'talk',
      bargeIn: true,
      text: 'Hello! Please enter a digit to continue.'
    },
    {
      action: 'input',
      maxDigits: 1,
      eventUrl: [`https://${req.get('host')}/dtmf`]
    }
  ]

  res.json(ncco)
})
```

We'll use the `talk` action to greet the caller and ask them to press a digit, setting the `bargeIn` option to `true` so the user can enter a digit without waiting for the verbal message to finish.

Then, we'll add an `input` to the NCCO to capture the digit via DTMF. Set the `maxDigits` property to 1 and the `eventURL` to a handler to receive and handle the input. Please note that you can expand the number of key presses a user can do via the `maxDigits` property.

## Handle the User Input

Let's add the code to handle incoming DTMF in `index.js`. Vonage makes a `POST` request to our webhook, which we'll expose as an endpoint at `/dtmf`. When we receive the request, we will create another `talk` action that inspects the request object and reads back the digits that the caller pressed:

```javascript
app.post('/dtmf', (req, res) => {
  const ncco = [{
    action: 'talk',
    text: `You pressed ${req.body.dtmf}`
  }]

  res.json(ncco)
})
```

The endpoint for the `event_url` needs to be POST, so let’s define `/events`:

```javascript
app.post('/events', (req, res) => {
  console.log(req.body)
  res.send(200);
```

This will log all triggered events to our terminal/command prompt. 

## Run the Application

Enter the following at your command/terminal prompt to run the application:

```javascript
node index.js
```

Let's make a phone call to see if your application works! Call your virtual number from your mobile phone and press a digit on your keypad. If everything works, you should hear the message you defined in your NCCO: "You pressed X."

## Wrap-up

Now that you have built an interactive voice response menu using the Vonage Voice API and Node.js, why not learn how to [send text-to-speech messages](https://developer.vonage.com/en/blog/how-to-make-an-outbound-text-to-speech-phone-call-with-node-js) in over 40 languages? You could also learn more about our [Voice API](https://developer.vonage.com/en/voice/voice-api/overview) and dive into several awesome [code snippets](https://developer.vonage.com/en/voice/voice-api/overview#code-snippets). 

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!