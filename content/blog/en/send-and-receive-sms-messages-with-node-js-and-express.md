---
title: Send and Receive SMS Messages With Node.js and Express
description: An in-depth tutorial that demonstrates how to send SMS text
  messages and receive replies using the Vonage APIs, Node.js and the Express
  framework.
thumbnail: /content/blog/send-and-receive-sms-messages-with-node-js-and-express/node-js_express_sms.png
author: michael-crump
published: true
published_at: 2023-01-05T22:12:22.461Z
updated_at: 2023-01-05T22:12:22.541Z
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

The [Vonage Messages API](https://developer.vonage.com/messages/overview) allows you to send and receive messages over SMS, MMS, Facebook Messenger, Viber, and WhatsApp. So you can communicate with your customers on whichever channel they love most. In this article, we'll focus on how to send and receive SMS messages with [Node.js](https://nodejs.org/) and [Express](https://expressjs.com/).

To begin, we'll perform the most straightforward implementation of the API and send an SMS message with Node.js and the [Vonage Messages API](https://developer.vonage.com/messages/overview). Next, we'll build a Webhook that can receive SMS messages (sent from your phone) using Express and perform an action once the message has been received. Finally, we'll wrap up a couple of steps that you can take to extend the application. Keep in mind that the documentation for the Messages API can be found [here](https://developer.vonage.com/api/messages-olympus?theme=dark).

## Prerequisites

Before you begin, make sure you have the following:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. 
* [Vonage CLI](https://www.npmjs.com/package/@vonage/cli) - Once Node.js is installed, you can use `npm install -g @vonage/cli` to install it. This tool allows you to create and manage your Vonage applications.

## Send an SMS Message With the Messages API

The Vonage Messages API is a multi-channel API that can send a message via different channels, such as SMS, MMS, Facebook Messenger, Viber, and WhatsApp. To use it, we'll need to install the [Vonage Node.js SDK](https://github.com/Vonage/vonage-node-sdk).

```
npm install @vonage/server-sdk
```

While Vonage has two different APIs capable of sending and receiving SMS, you can only use one at a time because it will change the format of the webhooks you receive.

Ensure that the Messages API is set as the default under **API keys**, then **SMS settings** of [your account](https://dashboard.nexmo.com/settings).

![Set Messages API as the default API for sending SMS messages](/content/blog/send-and-receive-sms-messages-with-node-js-and-express/messages-as-default-sms.png "Set Messages API as the default API for sending SMS messages")

Toggle the setting to **Messages API**, and then press **Save changes**. 

## Run ngrok

ngrok is a cross-platform application that enables developers to expose a local development server to the Internet with minimal effort. We'll be using it to expose our service to the Internet. If you haven't used ngrok before, there is a [blog post](https://learn.vonage.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/) that explains it in more detail. Once you have ngrok setup and are logged in (again, the free account is acceptable), then run the following command:

```
ngrok http 3000
```

After ngrok runs, it will give you a **Forwarding** URL that we'll use as the base for our Webhooks later in the article. Mine looks like the following:

![ngrok running successfully](/content/blog/send-and-receive-sms-messages-with-node-js-and-express/ngrok.png "ngrok.png")

## Create a Messages-Enabled Vonage Application

To interact with the Messages API, we'll need to create a Vonage API application to authenticate our requests. Think of applications more like containers and metadata to group all your data on the Vonage platform. We'll [create one using the Vonage API Dashboard](https://dashboard.nexmo.com/applications/new). 

Provide a name (such as "Send and Receive SMS Messages") and click on **Generate public and private key**. You'll be prompted to save a keyfile to disk—the private key. It's usually a good call to keep it in your project folder, as you'll need it later. Applications work on a public / private key system, so when you create an application, a public key is generated and kept with Vonage, and a private key is generated, not kept with Vonage, and returned to you via the creation of the application. We'll use the private key to authenticate our library calls later on.

Next, you need to enable the **Messages** capability and provide an **Inbound URL** and a **Status URL**.

Use the ngrok URL you got in the previous step and fill in each field, appending `/webhooks/inbound` and `/webhooks/status`, for the **Inbound URL** and **Status URL**. 

When a message reaches the **Messages API**, the data about it is sent to the **Inbound URL**. When you send a message using the API, the data about the message status gets sent to the **Status URL**.

Finally, link one or more of your virtual numbers to this application. Any messages received on these numbers will be passed along to your **Inbound URL**.

![Create Messages enabled Vonage Application](/content/blog/send-and-receive-sms-messages-with-node-js-and-express/tutorial.gif "Create Messages enabled Vonage Application")

## Send the SMS Message

Create a new folder where you'd like to store the source code for the application that we are creating. Once complete, type `npm init` to initialize a new Node.js application. Finally, create an `index.js` file and initialize the Vonage node library installed earlier.

```javascript
const { Vonage } = require('@vonage/server-sdk');

const vonage = new Vonage({
  applicationId: VONAGE_APPLICATION_ID,
  privateKey: VONAGE_APPLICATION_PRIVATE_KEY_PATH
});
```

Replace the **VONAGE_APPLICATION_ID** with the **Application ID** provided for the Vonage application you created. Replace the **VONAGE_APPLICATION_PRIVATE_KEY_PATH** with the path to the private key that was automatically downloaded for you. Note: I copied my `private.key` file to the root of my application and can access it by specifying that the **privateKey** is equal to `'./private.key'`

To send an SMS message with the Messages API, we'll use the `vonage.messages.send` method of the Vonage Node.js library. This method accepts objects as parameters, with information about the recipient, sender, and content. They vary for the different channels, so you'll need to check the [API documentation](https://developer.nexmo.com/api/messages-olympus) for the other channels mentioned.

For SMS, we'll need to specify a recipient and sender. Finally, the `content` object accepts a `type` of `text` and a text message. The callback returns an error and response object, and we'll log messages about the success or failure of the operation.

```javascript
const text = "👋Hello from Vonage";

vonage.messages
  .send({
    text: text,
    message_type: "text",
    to: TO_NUMBER,
    from: FROM_NUMBER,
    channel: "sms",
  })
  .then((resp) => console.log(resp.message_uuid))
  .catch((err) => console.error(err));
```

Replace `TO_NUMBER` with the destination phone number as a string and add your assigned phone number to the `FROM_NUMBER` field, then run the code with:

```
node index.js
```

That's it; you've just sent an SMS message using the Vonage Messages API. You might notice that the Messages API is a bit more verbose, yet it still needs just one method to send an SMS message.

## Receive SMS Messages

When a Vonage number receives an SMS message, Vonage will pass that message along to a predetermined Webhook. You've already set up the webhook URL when you created the Messages enabled Vonage application: `YOUR_NGROK_URL/webhooks/inbound`

### Create a Web Server

We'll create our webserver using [Express](https://expressjs.com/) because it's one of the most popular and easy-to-use Node.js frameworks for this purpose. We'll also be looking at the request bodies for the inbound URL, so we'll need to install `express` from npm.

```
npm install express --save
```

Let's create a new file for this and call it `server.js`. This can live in the same folder that we created earlier. 

We'll create a basic `express` application that uses the JSON parser from `express` and sets the `urlencoded` option to `true`. Let's fill out the `server.js` file we created. We'll use port 3000 for the server to listen to, and we already have ngrok running on port 3000.

```javascript
const express = require("express");

const { json, urlencoded } = express;

const app = express();

app.use(json());

app.use(
  urlencoded({
    extended: true,
  })
);

app.listen(3000, () => {
  console.log("Server listening at http://localhost:3000");
});
```

### Create Webhook for the Inbound URL

We will create a POST request handler for `/webhooks/inbound` for the inbound URL, and we'll log the request body to the console. Because Vonage has a retry mechanism, it will keep resending the message if the URL doesn't respond with `200 OK`, so we'll send back a `200` status.

```javascript
app.post("/webhooks/inbound", (req, res) => {
  console.log(req.body);

  res.status(200).end();
});
```

You can run the code with the following:

```
node server.js
```

### Try It Out

Now send an SMS message from your phone to your Vonage number. You should see the message being logged in the terminal window where you ran the code. It looks similar to this:

```bash
{
 to: '18335787204',
 from: '19999999999',
 channel: 'sms',
 message_uuid: 'a580f869-e995-4d76-9b80-a7befe3186a3',
 timestamp: '2022-12-07T23:04:32Z',
 usage: { price: '0.0057', currency: 'EUR' },
 message_type: 'text',
 text: "Hello from Michael's phone!",
 sms: { num_messages: '1' }
}
```

## Wrap-up

Now that you have learned how to send and receive SMS messages with the Vonage Messages API and Node.js, you could extend this project to reply to incoming SMS messages or include more complex, interactive elements for your current or future SMS needs. You can also use Node.js to send messages on [WhatsApp](https://developer.vonage.com/en/blog/send-a-whatsapp-message-with-node-dr) and other platforms.

As always, if you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!