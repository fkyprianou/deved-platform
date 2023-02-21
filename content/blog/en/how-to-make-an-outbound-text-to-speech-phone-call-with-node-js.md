---
title: How to Make an Outbound Text-to-Speech Phone Call with Node.js
description: In this tutorial, you will learn how to use the Vonage Voice API to
  create a text-to-speech outbound phone call securely.
author: michael-crump
published: true
published_at: 2023-02-09T19:31:18.435Z
updated_at: 2023-02-09T19:31:18.488Z
category: tutorial
tags:
  - node
  - voice-api
  - text-to-speech
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
* Control the flow of inbound and outbound calls in JSON with [Nexmo Call Control Objects](https://developer.vonage.com/en/voice/voice-api/overview#ncco) (NCCO). (Note: Nexmo is now Vonage).
* Record and store inbound or outbound calls
* Create conference calls
* Send text-to-speech messages in 40 languages with different genders and accents.
* And more! 

In this tutorial, you will learn how to securely use the Vonage Voice APIs to create a text-to-speech outbound phone call using Node.js. 

T﻿he source code to the application can be found [here](https://github.com/Vonage-Community/blog-voice-nodejs-outbound-call).

## Prerequisites

Before you begin, make sure you have the following:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [OPTIONAL - Vonage CLI](https://www.npmjs.com/package/@vonage/cli) Once Node.js is installed, install the CLI by typing `npm install -g @vonage/cli`. This tool allows you to create and manage your Vonage applications from a command-line interface vs. the Vonage Developer Portal.

## Create a Voice-Enabled Vonage Application

To use the [Vonage Voice API](https://developer.vonage.com/voice/voice-api/overview), you'll have to create a [Vonage Application](https://developer.vonage.com/application/overview) from the developer portal.

You need a Vonage application because it contains the security and configuration information you need to interact with the Vonage Voice APIs. All requests to the Vonage Voice API require authentication. You must generate a private key with the Application API, which allows you to create JSON Web Tokens (JWT) to make the requests.

The application's associated public and private keys can be created through the [Vonage Developer Portal](https://developer.vonage.com/) or the [Vonage CLI](https://developer.vonage.com/application/vonage-cli). For the purpose of this tutorial, we'll use the Vonage Developer Dashboard. 

## The Vonage Developer Dashboard

After creating an account, log into the Vonage Developer Dashboard, look for the [Application section](https://dashboard.nexmo.com/applications), and from here, create a new application. Give your application a name, such as **OutboundCall**, and press **Generate public and private key**. This will prompt you to download your private key and populate the public key for you. Please keep this information safe, as anyone with access to it can use your account!

![Creating a Voice enabled application](/content/blog/how-to-make-an-outbound-text-to-speech-phone-call-with-node-js/outboundcall.png "OutboundCall.png")

Scroll down the page and ensure that the **Voice** capability is toggled on, and you can leave the **Answer**, **Event**, and **Fallback URLs** blank. 

![Turning on the Voice Capability](/content/blog/how-to-make-an-outbound-text-to-speech-phone-call-with-node-js/voicecapability.png "VoiceCapability.png")

Press **Generate new application** to continue. You can also link a number if you choose, but it won't be necessary for this tutorial. 

## Node.js project setup

Now that we have created our Vonage Voice Application inside  the developer dashboard and generated our public/private key pair let’s look at how we should configure our Node.js application. 

Begin by going to a command/terminal prompt, creating a working directory, and then initialize a Node.js project.

```bash
npm init -y
```
We need to install the Node Vonage Server SDK as well. 

```bash
npm install @vonage/server-sdk
```


Create an `index.js` file, and initialize the Vonage SDK and credentials with the following code: 

```javascript
const {
    Vonage
} = require('@vonage/server-sdk');
const credentials = {
    applicationId: 'VONAGE_APPLICATION_ID',
    privateKey: "./private.key",
};
const options = {};
const vonage = new Vonage(credentials);
```

Replace the **VONAGE_APPLICATION_ID** with the **Application ID** provided for the Vonage application you created in the developer dashboard. Ensure that the **privateKey** has the path to the private key downloaded earlier. Note: I copied my private.key file to the root of my application and can access it by specifying that the privateKey is equal to `./private.key`.

To call a number with the Voice API, we'll use the **vonage.voice.createOutboundCall** method of the Vonage Node.js library. This method accepts objects as parameters, with information about the recipient, sender, and text to be spoken. 

For the Voice API, we'll need to specify a recipient, sender, and the [NCCO](https://developer.vonage.com/en/voice/voice-api/overview#ncco) object we want to send. 

Finally, the content object accepts a type of text and a text message. The callback returns an error and response object, and we'll log messages about the success or failure of the operation.

```javascript
vonage.voice.createOutboundCall({
		to: [{
			type: 'phone',
			number: YOUR_NUMBER
		}],
		from: {
			type: 'phone',
			number: VONAGE_VIRTUAL_NUMBER
		},
		ncco: [{
			"action": "talk",
			"text": "You are listening to a test text-to-speech call made with the Vonage Voice API",
		}]
	})
	.then(resp => console.log(resp))
	.catch(err => console.error(err));
```

Replace **YOUR_NUMBER** with a phone that you can answer immediately. The **VONAGE_VIRTUAL_NUMBER** should be your virtual number, which you can find in your dashboard. You can customize the NCCO object with [optional params](https://docs.nexmo.com/voice/voice-api/ncco-reference) with various voices by language, gender, and even accent.

Please note that we could host the NCCO object on GitHub, for example, by declaring a **ANSWER_URL** with the following and removing the **ncco** with an **answer_url** as shown below:

```javascript
const ANSWER_URL = 'https://raw.githubusercontent.com/nexmo-community/ncco-examples/gh-pages/text-to-speech.json'

...

answer_url: [ANSWER_URL]
```

## Run the Application

Enter the following at your command/terminal prompt to run the application:

```javascript
node index.js
```

Your phone should ring, you will hear a voice reading the text we specified, and the call will be terminated. Please note that you may observe some latency depending on the phone carrier that you are connecting to.

## Wrap-up

Now that you have learned how to create an outbound call with the Vonage Voice API and Node.js, you can listen to the audio in [different languages](https://developer.vonage.com/en/voice/voice-api/concepts/asr) or [accents](https://developer.vonage.com/en/verify/templates#creating-a-custom-template) or [customize the spoken text](https://developer.vonage.com/en/voice/voice-api/concepts/customizing-tts) even further to find the right one for your business. 

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!