---
title: How to Receive an SMS Delivery Receipt with Node.js
description: How to receive SMS delivery receipts from mobile carriers with a
  webhook written with Node.js and Express.js
thumbnail: /content/blog/how-to-receive-an-sms-delivery-receipt-with-node-js/delievery-sms_node-js.png
author: michael-crump
published: true
published_at: 2023-03-28T17:59:49.579Z
updated_at: 2023-03-28T17:59:49.640Z
category: tutorial
tags:
  - sms-api
  - node
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

When you send a text message to a phone number with Vonage APIs, the HTTP response from the API can tell you if the message has been successfully **sent**. However, it doesn't tell you if the message is **delivered** to the recipient. Thankfully, there is a field in your [Vonage Developer Portal](https://developer.vonage.com/) that allows you to see when the status of the delivery changes in the form of a webhook.

In this tutorial, you will learn how to find out if the SMS messages sent from your virtual number have been delivered using Node.js.

If you would like to skip ahead and just run the app, you can find a fully working version on [GitHub](https://github.com/Vonage/vonage-node-code-snippets/blob/master/sms/dlr-express.js).

## Prerequisites

Before you begin, make sure you have the following:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. 

## Using ngrok to set up a local tunnel

ngrok is a cross-platform application that enables developers to expose a local development server to the Internet with minimal effort. We'll be using it to expose our webhook (created shortly) to the public Internet.

When a message gets delivered, the mobile phone carrier typically returns a **Delivery Receipt (DLR)** to Vonage, explaining the delivery status of your message. If you have set up a webhook endpoint, Vonage then forwards this delivery receipt to your endpoint.

We need to set up ngrok to create a local tunnel so that we can run our Node.js code later. 

Once you have ngrok setup and are logged in (again, the free account is acceptable), then run the following command:

```
ngrok http 5000
```

After ngrok runs, it will give you a **Forwarding** URL that we'll use as the base for our Webhook later in the article. Mine looks like the following:

![ngrok running successfully](/content/blog/how-to-receive-an-sms-delivery-receipt-with-node-js/ngrok.png "ngrok.png")

Note the Forwarding URL, as we will use it in the next step.

## Turning on Delivery receipts (DLR) inside the Vonage Developer Dashboard

To use this feature, you must create a [Vonage Developer Account](https://developer.vonage.com/) from the developer portal. If you don't have an account, then go ahead and create one, as we provide credits to get started. No credit card is required. 

Sign in to your Vonage account, and go to [Settings](https://dashboard.nexmo.com/settings). Scroll down to **SMS settings**, fill out the **Delivery receipts (DLR) webhooks** with the ngrok Forwarding URL and append **/receipt**, and press save.

![SMS Settings](/content/blog/how-to-receive-an-sms-delivery-receipt-with-node-js/smssettings.png "smssettings.png")

Every time you send a message from your virtual number, the delivery receipt webhook call will be made to that URL which will run the code on your local machine. Let’s write some code with Node.js and Express to handle it!

## Node.js project setup

Now that we have configured ngrok and added a **Delivery receipt webhook** let’s look at how we should configure our Node.js application. 

Begin by going to a command/terminal prompt, creating a working directory, and initializing a Node.js project.

```bash
npm init -y
```

We will handle the requests with [Express](https://expressjs.com/) and use [body-parser](https://www.npmjs.com/package/body-parser) to parse incoming request bodies. Install both of these with the following:

```javascript
npm install express body-parser --save
```

Create an `index.js` file, instantiate express, and listen to the server to port 5000, since we have configured ngrok to that port.

```javascript
const app = require('express')();
const bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
```

Let’s define the endpoint for the Receipt URL as `/receipt`.

```javascript
app
  .route('/receipt')
  .get(handleDeliveryReceipt)
  .post(handleDeliveryReceipt)
```

Then define the **handleDeliveryReceipt** function as the following:

```javascript
function handleDeliveryReceipt(request, response) {
  const params = Object.assign(request.query, request.body)
  console.log(params)
  response.status(204).send()
}

app.listen(process.env.PORT || 5000)
```

The application will then log the receipt to the console window. 

## Run the Application

Enter the following at your command/terminal prompt to run the application:

```javascript
node index.js
```

Try sending text messages from your Vonage virtual number to your mobile phone number to test.

If your message has been successfully sent to your mobile phone, you should get a receipt with the info, including **status**, **messageId**, **network-code**, etc. This information is shown through the terminal or command prompt. 

Below is a sample of what mine looks like: 

```text
{
  msisdn: '14259999999',
  to: '18339999999',
  'network-code': '310260',
  messageId: 'c0f9b0a4-f62a-481e-8698-d52231de9047',
  price: '0.00869000',
  status: 'delivered',
  scts: '9992241831',
  'err-code': '0',
  'api-key': '99999999',
  'message-timestamp': '2023-02-24 18:31:15'
}
```

Note: Not all networks and countries support delivery receipts. Please refer to the [documentation](https://developer.vonage.com/en/messaging/sms/code-snippets/delivery-receipts) for further information.

## Wrap-up

Now that you have learned how to find out if the SMS messages sent from your virtual number have been delivered using Node.js, you could extend this project to save the response to a cloud database or try out our [Messaging API](https://developer.vonage.com/en/messages/overview). Also, try sending a text message when your phone is off to see the various states of the **status** field.

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!