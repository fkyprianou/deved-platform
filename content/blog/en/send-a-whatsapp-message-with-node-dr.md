---
title: How to Send a WhatsApp Message with Node.js
description: Find out how to send a WhatsApp message with Node.js in 5 simple
  steps. Learn more in this tutorial from Vonage Developer.
thumbnail: /content/blog/how-to-send-a-whatsapp-message-with-node-js/node-js_whatsapp.png
author: garann-means
published: true
published_at: 2020-04-15T12:06:53.000Z
updated_at: 2020-11-05T14:14:30.543Z
category: tutorial
tags:
  - messages-api
  - node
  - whatsapp
comments: true
redirect: ""
canonical: ""
---
# How to Send a WhatsApp Message with Node.js

Traditionally, businesses contact customers by phone or SMS. But sometimes you really want to communicate with your customers via WhatsApp. 

Connecting with customers via WhatsApp is easy thanks to the [Vonage Messages API](https://developer.vonage.com/en/messages/overview). With Vonage, you can easily automate WhatsApp messages, including sending WhatsApp messages via Node.js. 

In this tutorial, we’ll show you what you need  if you want to use Node for sending WhatsApp messages. 

After that, we’ll walk you through the steps to send and receive WhatsApp messages with a Node script.

## Prerequisites

Using the Vonage API, you won’t need any additional third-party tools to test WhatsApp messaging with Node. 

The [Vonage dashboard](https://developer.vonage.com/en/account/guides/dashboard-management) now even allows you to test out WhatsApp messaging without a WhatsApp business account. You can use the cURL command in the [Messages API Sandbox](https://developer.vonage.com/en/messages/concepts/messages-api-sandbox) to verify you're able to send a message. Once you've tried that, you might like to see how you can incorporate WhatsApp messaging into your Node.js application.

You can make use of the sandbox to test out your code without doing a lot of set up. Node already includes everything we need to make a POST request, and the Messages API does the rest. 

To send a WhatsApp message with Node.js, you will need the following:

* A [Vonage API account](https://developer.vonage.com/sign-up). Access your Vonage API Dashboard to locate your API Key and API Secret, which can be found at the top of the page.
* A Vonage phone number. To purchase one, go to Numbers > Buy Numbers and search for one that meets your needs.
* [Node.js](https://nodejs.org/) runtime environment
* An allowlisted [WhatsApp](https://www.whatsapp.com/) number to test with

## Instructions

Below, we’ll walk you through the step-by-step instructions for using Node.js and the Vonage API to send WhatsApp messages.

To send a WhatsApp message via Node.js, simply follow these 5 steps:

1. Set up a WhatsApp sandbox
2. Create your WhatsApp data
3. Create a POST Request in Node
4. Make the Node Request
5. Test Your Code by Sending a Message to WhatsApp

### **1. Set up a WhatsApp Sandbox**

Start by navigating to the [Messages API Sandbox](https://developer.vonage.com/en/messages/concepts/messages-api-sandbox) in the Vonage Dashboard. Then, set up a sandbox for testing WhatsApp.

The quickest way to do this is with a QR code. Take your device with WhatsApp installed and center the camera on the QR code supplied. This will generate a custom message to allowlist your number for testing.

You'll see several additional options for joining the allowlist, including manually creating a message to the number supplied with a unique passphrase. If none of the others will work for your setup, that one should.

![Sending a WhatsApp message to join the allow](/content/blog/how-to-send-a-whatsapp-message-with-node-js/whatsapp-whitelisting.jpeg "Sending a WhatsApp message to join the allow")

### **2. Create Your WhatsApp Data**

To make a call to the sandbox API you'll need to supply a unique username, password, and to and from WhatsApp numbers. 

The username and password will be the two halves of the `u` argument in the cURL command from your dashboard. Copy the masked value, which will look like `12ab3456:123AbcdefghIJklM`. The part before the colon is your username and the part after is your password. They're the same as your Vonage API key and secret, which are also available from "[Getting Started](https://dashboard.nexmo.com/getting-started-guide) in the dashboard.

You can also copy the data for your request straight from the cURL command and stringify it. This tells the API to send a text message from the sandbox number to your allowlisted number. You can change the text of the message to whatever you like, including generating it programmatically:

```javascript
var user = '12ab3456';
var password = '123AbcdefghIJklM';
var from_number = '14151234567';
var to_number = '441234567890';

const data = JSON.stringify({
  "from": { "type": "whatsapp", "number": from_number },
  "to": { "type": "whatsapp", "number": to_number },
  "message": {
    "content": {
      "type": "text",
      "text": "Hi! Your lucky number is " + Math.floor(Math.random() * 100)
    }
  }
});
```

### **3. Create a POST Request in Node**

Now that you’ve set up a sandbox and created WhatsApp data, the next step is to create a POST request in Node.

The code to send a POST request with [Node's `https` module](https://nodejs.org/api/https.html) might seem complicated, but it's just listing the ordinary parts of an HTTP request. First you'll require the module. Then, you'll construct an object with the request options. 

Most of the options, like the `hostname`, `port`, and `method`, would be the same for any request to the sandbox API. All you'll need to provide besides those are your authorization variables:

```javascript
const https = require('https');

const options = {
  hostname: 'messages-sandbox.nexmo.com',
  port: 443,
  path: '/v0.1/messages',
  method: 'POST',
  authorization: {
    username: user,
    password: password
  },
  headers: {
    'Content-Type': 'application/json'
  }
};
```

### **4. Make the Node Request**

With your request options defined in Node, you can go ahead and make the request using the `https` module's `request` function. 

When you make the request, you'll pass in your options and a callback, which gets any response to your request. The response contains an HTTP status code and you can listen for any additional data sent back, which should be a message UUID if everything works correctly. 

You can also create an error listener on the request itself.

Once you've created your Node request, you can write your data object to the request. This tells the API to send the WhatsApp message. After this, you can close the request and end your code:

```javascript
const req = https.request(options, (res) => {
  console.log(`statusCode: ${res.statusCode}`)

  res.on('data', (d) => {
    process.stdout.write(d)
  })
});

req.on('error', (e) => {
  console.error(e);
});

req.write(data);
req.end();
```

### **5. Test Your Code from the WhatsApp Sandbox**

Now you can save your code in a file called app.js. After that, you can test sending a message from your WhatsApp sandbox to your allowlisted WhatsApp number.

From the directory where the file is saved, run:

```shell
> node app.js
```


If you’ve followed the instructions in this post, your Node file should successfully send a message from the WhatsApp sandbox. At this point, the test message should appear in WhatsApp on your allowlisted device.



### Troubleshooting

If your message was sent successfully and you received a 202 status but the message never showed up, your permissions may have expired. An easy fix is to send your allowlisting passphrase again.

To understand more about WhatsApp policies and charges, check out our guide to [WhatsApp messaging for developers](https://developer.vonage.com/en/messages/concepts/whatsapp).

You can find the [code for this example on Github](https://github.com/nexmo-community/send-whatsapp-with-node/).