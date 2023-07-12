---
title: How to Send MMS Messages with Node.js
description: The Vonage Messages & Dispatch API now allows you to send MMS
  (Multimedia Messaging Service) from any US shortcode number you have in your
  dashboard to other numbers within the United States. In this article you will
  learn how to send an MMS using Node.js using the official vonage-node client
  library. Prerequisites Before you start, […]
thumbnail: /content/blog/how-to-send-mms-with-node-js-dr/send-mms-nodejs.png
author: amanda-cavallaro
published: true
published_at: 2023-06-30T11:39:24.631Z
updated_at: 2023-06-30T11:39:24.655Z
category: tutorial
tags:
  - mms
  - messages-api
  - javascript
comments: true
redirect: ""
canonical: ""
---
The Vonage Messages API now allows you to send MMS (Multimedia Messaging Service) from US shortcode numbers you have in your dashboard to other numbers within the United States. 

In this article, you will learn how to send an MMS using Node.js using the official [vonage-node-sdk](https://github.com/vonage/vonage-node-sdk) client library.

## Prerequisites

Before you start, make sure you have the following:

* A basic understanding of JavaScript & Node.js
* Node.js installed on your machine
* A fresh US SMS & MMS capable number. You can reference our [Numbers Guide](https://developer.vonage.com/en/numbers/guides/number-management) for details on how to buy a number if you don't have one.

**Note**: MMS messaging only works for A2P (Application to person) use cases. Therefore if you were to purchase two virtual phone numbers to send and receive the message to each other other.

<sign-up number></sign-up>

## Getting Started

Before we begin, let's get set up for success by taking care of a few things inside the Vonage dashboard.

Let's start by creating a new Messages Application. To do this, head to the [Application section](https://dashboard.nexmo.com/messages/applications) of your admin dashboard and click on *Create a new application*. 

![Your Applications page containing a purple button to create a new application](/content/blog/how-to-send-mms-messages-with-node-js/yourapplications.jpg "Your applications page")

Next, you will need to fill out the form with your application name and the required webhook URLs. 

![Messages capability showing inbound and status URLs using MockBin](/content/blog/how-to-send-mms-messages-with-node-js/messages.jpg "Message capability with added URLs ")

Every Messages & Dispatch application requires you to specify a `Status URL` and an `Inbound URL`. In larger, production-ready applications, you would have these pointing to a URL on your own server. For this tutorial there is no need for that, you'll just need a URL that can respond with a `200 OK` status, you can use a service such as [MockBin](http://mockbin.org) to provide what you need as I did in the example above.

![Generate Public and Private Key](/content/blog/how-to-send-mms-messages-with-node-js/keys.jpg "Generate Public and Private Key")

In order to authenticate API requests alongside your API key and API secret you will also need a public/private key pair, which can be automatically generated for you by clicking the *Generate public/private key pair* link.

This will set a public key in the form field, and also download a `private.key` file to your machine. You will need to put this file wherever you will write the code we put together.

Finish up by clicking *Create a new application.* You will then be asked to select a number to use with this application by clicking the *Link* button next to your number of choice. If you do not have any numbers on your dashboard you can purchase them there or via the command line.



Note down the *Application ID*, it will join your API key, API secret and Private Key in the code we will work on next.

## Sending MMS Using Node.js

Now that the admin is complete, let's create a Node.js script that will use the [Vonage Server SDK for Node.js](https://github.com/vonage/vonage-node-sdk) to send an MMS to a pre-set number when run.

Start by creating a new folder (and if you haven't already done so, put the `private.key` file generated earlier in it.

Run the following commands to set up a new Node.js application and to install the `vonage-node` client library:

```
npm init -y
npm install -g @vonage/cli
npm install @vonage/server-sdk
```

Then create a new file called `send-mms.js`, and open it inside your editor.

Initialise a new Vonage instance:

```
const { Vonage } = require('@vonage/server-sdk')
const { MMSImage } = require("@vonage/messages")

const vonage = new Vonage({
  apiKey: VONAGE_API_KEY, // Found in your Vonage Dashboard
  apiSecret: VONAGE_API_SECRET, // Found in your Vonage Dashboard
  applicationId: VONAGE_APPLICATION_ID, // Generated earlier
  privateKey: VONAGE_APPLICATION_PRIVATE_KEY_PATH // Generated earlier
})
```

Finally, add the code to send the MMS message using the Messages API:

```javascript
const image = new MMSImage({
  image: {
    url: "https://placekitten.com/200/30",
  },
  to: "TO_NUMBER",
  from: "FROM_NUMBER",
});

vonage.messages
  .send(image)
  .then((resp) => console.log(resp))
  .catch((err) => console.error(error));
```

The above code sends a single message (a picture of a cat) to any number you add into the first object. If for any reason there is a failure in sending the message, the issues will be logged to the console, otherwise, if everything works as it should then the `messageUUID` will be logged instead.

**N﻿ote**: 'https://placekitten.com/200/300', // This is a placeholder image you can use

With that, head to your terminal of choice and run:

```
node send-mms.js
```

## Where To Go From Here

Sometimes, for any number of reasons, an MMS message may not be delivered to a recipient so, as a next step from this project, you could experiment with our [Dispatch API](https://developer.vonage.com/en/dispatch/overview) and build a failover mechanism that checks whether the MMS has been delivered, and if not, your app would instead send a standard SMS message with a link to the image you wanted to include.