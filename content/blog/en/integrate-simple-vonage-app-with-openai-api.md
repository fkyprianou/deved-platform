---
title: Integrate Simple Vonage App With OpenAI API
description: The article will include workflow plus the simple app that uses
  Vonage Voice API and Message API to interact with 3d party Generative AI. In
  this case, I'll use image/text generation using OpenAI API.
author: oleksii-borysenko
published: true
published_at: 2023-01-27T11:52:19.543Z
updated_at: 2023-01-27T11:52:19.552Z
category: tutorial
tags:
  - javascript
  - node
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Generative AI has gone mainstream.. Over the past year, models and products like ChatGPT and DALL-E-2 have appeared that allow you to generate texts, images, and audio. The creators of ChatGPT and DALL-E-2, OpenAI, have opened up these powerful tools for developers to access and create imaginative new applications.

In this article, we will first consider how to integrate [Vonage Voice API](https://developer.vonage.com/voice/voice-api/overview) and [OpenAI API](https://openai.com/api/). We will create an app that will receive a prompt from a user, via call, call and send it to a generative AI service. We will then send the AI’s response to the user using [Vonage Messages API](https://developer.vonage.com/messages/overview).

The Vonage Messages API allows you to send and receive messages over SMS, MMS, Facebook Messenger, Viber, and WhatsApp! Ready to get started? Let's dive in! Remember to check out the [Messages API documentation](https://developer.vonage.com/messages/overview) for more information.

## Prerequisites

We've already developed a starter Vonage Voice application to receive a call, catch your response, and send it to a 3rd party (OpenAI).
Users can deploy the App using Github Codespaces.
Fork [this repository](https://github.com/obvonage/Simple-Vonage-App-With-OpenAI-API). Open it in Codespaces by clicking "Create codespace on main"

![Create Codespace interface](/content/blog/integrate-simple-vonage-app-with-openai-api/codespaces.png)

Besides, users can use their laptop or server to play with the App.
In this case, make sure you have the following:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [Vonage CLI](https://www.npmjs.com/package/@vonage/cli) - Once Node.js is installed, you can use `npm install -g @vonage/cli` to install it. This tool allows you to create and manage your Vonage applications.
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. See also: [Using ngrok in Rails in 2022](https://developer.vonage.com/blog/22/08/24/using-ngrok-in-rails-in-2022)

## Create a new Vonage app

Sign in/Sign up for free [developer.vonage.com](https://developer.vonage.com/)
In order to be able to use the [Vonage Voice API](https://developer.vonage.com/voice/voice-api/overview), you'll have to create a [Vonage Application](https://developer.vonage.com/application/overview) from the developer portal.

All requests to the Vonage Voice API require authentication. Therefore, you should generate a private key with the Application API, which allows you to create JSON Web Tokens (JWT) to make the requests. For demo purposes, we will use the API key and API Secret.

In the left menu [here](https://dashboard.nexmo.com/) click API Settings, left menu item.

![API Settings](/content/blog/integrate-simple-vonage-app-with-openai-api/settings.png)

Copy and paste in the `.env` file API key and API Secret

```
API_KEY=b**********
API_SECRET=******************
```

A Vonage application contains the security and configuration information you need to interact with the Vonage Voice APIs.

Let's create an Application using Vonage Developer Dashboard.

In the Application left menu item.
Create a new App. For example, `VoiceApp`. Generate a public and private key

![Create Vonage App](/content/blog/integrate-simple-vonage-app-with-openai-api/createapp.png)

We will create a bot to answer an inbound phone call. The bot will ask what image content you want to generate.

Check it using Vonage CLI.

```bash
npm install -g @vonage/cli
```

Set configuration

```bash
vonage config:set --apiKey=[API_Key] --apiSecret=[API_Secret]
```

Output

```bash
Configuration saved.
```

We need to buy a virtual number for our app to accept phone calls.
Search and buy virtual phone numbers. Choose the number with the mention `Voice` in the Capabilities column

```bash
vonage numbers:search GB
```

The following command can buy a related phone number

```bash
vonage numbers:buy **732**56** GB
```

Or search and buy virtual phone numbers using [Vonage Dashboard](https://dashboard.nexmo.com/buy-numbers). Select Voice feature from the dropdown menu.

Find related App

```bash
vonage apps
```

```bash
 Name                            Id                                   Capabilities 
 ─────────────────────────────── ──────────────────────────────────── ──────────── 
 VoiceApp                        4e15f46e-****-4a0d-9749-000000000000 voice          
```

Link phone number and the App

```bash
vonage apps:link [APP_ID] --number=[NUMBER]
```

For example

```bash
vonage apps:link 4e15f46e-****-4a0d-9749-000000000000 --number=44750385680
```

Expected response:

```bash
Number '**732**56**' is assigned to application '4e15f46e-****-4a0d-9749-000000000000'.
```

You can also link numbers using Vonage Dashboard, go to Applications, open related App, and click the 'Link' button in the list of numbers.

![link number with app](/content/blog/integrate-simple-vonage-app-with-openai-api/link-number-with-app.png)

## Create Call Control Object

Speech Recognition (ASR)
Automatic Speech Recognition (ASR) enables apps to support voice input for such use cases as IVR, identification and different kinds of voice bots/assistants. Using this feature, the app gets transcribed user speech (in the text form) once it expects the user to answer some question by saying it rather than entering digits (DTMF); and then may continue the call flow according to its business logic based on what the user said.

![ASR scheme](/content/blog/integrate-simple-vonage-app-with-openai-api/asr.png)

You can use the `input` action to collect digits or speech input by the application you are calling. This action is synchronous; Vonage processes the input and forwards it to the `eventUrl` webhook endpoint. You will configure to receive this input in your request. Your webhook endpoint should return another NCCO that replaces the existing NCCO and controls the Call based on the user input.

```
{
      action: 'talk',
      text: 'Hi, describe an image that you want to generate'
    },
    {
      eventMethod: 'POST',
      action: 'input',
      eventUrl: [
        '[Codespace-or-server-URL]/webhooks/asr'],
      type: [ "speech" ],
      speech: {
        language: 'en-gb',
        endOnSilence: 0.1
      }
    },
    {
      action: 'talk',
      text: 'Thank you'
    }
```

Useful links:

* [Validate Nexmo Call Control Object (NCCO)](https://dashboard.nexmo.com/voice/playground) 
* [Recognition settings](https://developer.vonage.com/voice/voice-api/ncco-reference#speech-recognition-settings)

## Configure Generative AI app, Open AI

OpenAI released new image generation capabilities with DALL·E models.

According to Your Content chapter in Terms of Use : "... OpenAI hereby assigns to you all its right, title and interest in and to Output."

New users can use credit (Free trial usage). With this credit, you can create or edit 900 images `1024x1024`

First, after registering and confirming your phone number, you need to [Generate your API key](https://beta.openai.com/account/api-keys)

With this API key, we can move forward.

Paste it to your `.env` file

```
API_KEY=b**********
API_SECRET=******************
OPENAI_API_KEY=sk-**************************************
EVENT_URL=https://******************************************/webhooks/asr
```

In the following JSON payload, you can manage parameter
`n` - the number of images generated. You can request 1-10 images at a time
`size` - available sizes 256x256, 512x512, or 1024x1024 pixels. Smaller sizes are faster to generate.

```
    .send(JSON.stringify({
      "prompt": promptText,
      "n": 1,
      "size": "1024x1024"
    }))
```

## Configure Vonage Message API

To Receive a message with content or link, we will use Vonage message API.

For WhatsApp, Scan the QR code and hit send on the pre-filled message.

![WhatsApp Sandbox QR](/content/blog/integrate-simple-vonage-app-with-openai-api/whatsapp_qr.png)

Open [Messages API Sandbox](https://dashboard.nexmo.com/messages/sandbox) if you need additional information.

## Deploy App in Codespace

After you open source in GitHub Codespace

Run the following command to install packages

```
npm install
```

Run the following command in the terminal to receive the GitHub Codespace URL for webhooks

```
echo "https://${CODESPACE_NAME}-3000.preview.app.github.dev/webhooks/asr" 
```

Copy and paste the output in `EVENT_URL=` in the `.env` file

```
API_KEY=b**********
API_SECRET=******************
OPENAI_API_KEY=sk-**************************************
EVENT_URL=https://******************************************-3000.preview.app.github.dev/webhooks/asr
```

Update App settings using Vonage CLI 

```bash
vonage apps:update 4e15f46e-****-4a0d-9749-000000000000 --voice_event_url=[Codespace-or-server-URL]/webhooks/event --voice_answer_url=[Codespace-or-server-URL]/webhooks/answer
```

Update App settings using Dashboard. Go to Application in the left menu. Choose a related app and  click the 'Edit' button

![Edit App](/content/blog/integrate-simple-vonage-app-with-openai-api/edit-app-urls.png)

Run the App

```bash
node index.js
```

In the terminal, open the `Port` tab. Click on `Private` in the `Visibility` column, and change it to `Public`

![codespace port public](/content/blog/integrate-simple-vonage-app-with-openai-api/codespace-port-public.png)

Everything is ready

* Try this out by calling the number that is linked with the app `**732**56**`
* Tell the bot your tip
* Wait for the content in the corresponding messenger
* Monitor the console

Following, you can find a sample image that you can receive on your telephone

## Wrap-up

Now that you have learned how to create a bot answering service for an inbound phone call, proceed with your request and send messages with the Vonage Messages API and Node.js, you could extend this project to making conversation using [Vonage AI Studio](https://studio.ai.vonage.com/agents) or integrate it with chatGPT.

Join the Conversation
Are you excited to get started with the Meetings API? Still unsure? Let us know what you're building and how we can help! Join the conversation on our [Vonage Community Slack](https://developer.vonage.com/community/slack) or send us a message on [Twitter](https://twitter.com/VonageDev).