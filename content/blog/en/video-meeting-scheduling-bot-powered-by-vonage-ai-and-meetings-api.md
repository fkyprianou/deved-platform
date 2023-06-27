---
title: Video Meeting Scheduling Bot Powered by Vonage AI and Meetings API
description: Learn how to enhance your user experience around video meetings
  with Meetings API and AI Studio, two nocode/lowcode APIs from Vonage. Just a
  little javascript to make custom video calls and chat bot experience.
thumbnail: /content/blog/video-meeting-scheduling-bot-powered-by-vonage-ai-and-meetings-api/meetings-scheduler.png
author: enrico-portolan
published: true
published_at: 2023-07-11T09:39:52.554Z
updated_at: 2023-07-11T09:39:52.579Z
category: tutorial
tags:
  - meetings-api
  - ai-studio
  - lowcode
comments: true
spotlight: true
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Remote work and virtual collaboration allow businesses to think of a whole new customer experience: video conferencing. As a web developer, I'm always looking for ways to improve user experience. And I especially like it if I can keep from adding too much extra work for myself.

That's why I'm thrilled to introduce my latest project, a smart scheduling bot that I've built using the Vonage Meetings API and Vonage AI. This bot provides a ton of value for clients, enhances user experience, and ultimately requires little coding from me!

In this blog post, I'll walk you through the making of this bot, the challenges I faced, and the solutions I discovered. 

> If you would like to skip ahead and get right to deploying it, you can find all the code for the app on [GitHub](https://github.com/Vonage-Community/vonage-meetings-bot-scheduler).

## Project Setup

1. Vonage Account
2. Node and npm
3. ngrok for Webhook Testing

### V﻿onage Account

<sign-up></sign-up>

### Node and npm

The NodeJS server structure is the following:

1. `index.js` file where the server code is defined. In this file we will define the routes and call the Meeting API endpoint. 
2. An `.env` file where we set up the credentials to authenticate with Vonage.

### ngrok for Webhook Testing

If you don’t have it, install ngrok on your local machine. Learn how [here](https://developer.vonage.com/en/getting-started/tools/ngrok#see-also). Then run the command to start the tunnel, specifying the port number of the local server you want to expose (`ngrok http 3000`). Ngrok will generate a unique URL that allows external access to your local server. We will need it on the next step when setting up webhooks on Meetings API

## Use Meetings API

Vonage Meetings API allows you to integrate real-time, high-quality interactive video meetings into your web apps. Vonage has two main products when it comes to video capabilities: Video API and Meetings API. The Meetings API is ideal for those who want plug-and-play meetings with limited customization; it takes only a few lines of code to generate a meeting and embed it into your application. The Video API is ideal for those who want more customization and flexibility, but it will take more development effort. Please have a look at the [Meetings API Documentation](https://developer.vonage.com/en/meetings/technical-details) for more details. 

For this project, I choose to use the Meetings API because it allows me to create meeting links with only one API call, without the need to host any application.
The first step is to create a [Vonage application](https://developer.vonage.com/en/getting-started/concepts/glossary#vonage-application)in the dashboard, enable the Meetings API functionality, set the webhooks (I will get back on these later on in the post) and click create.

![Meetings API Webhooks in Dashboard](/content/blog/video-meeting-scheduling-bot-powered-by-vonage-ai-and-meetings-api/enrico-meetings-api-dashboard.png "meetings-api-webhooks-in-dashboard.png")

### Creating Endpoints for Meeting Creation:

One of the core functionalities we will implement is the ability to create meetings programmatically. Using the Vonage Meetings API, we can define endpoints on our Node.js server that allow us to generate meetings with specific parameters. These parameters include the type of meeting (such as instant or long term), expiration time, UI language, recording options, and display name. By providing this flexibility, we can tailor the meeting experience to meet the unique requirements of our application and users. We will collect the parameters during the conversation with the bot. The following code goes in your `index.js` file.

Let’s explore the code:

```javascript
const { type, expires_at, recording_options, ui_settings, display_name } =
    req.body;
  const jwt = generateJwt();
  const url = "https://api-eu.vonage.com/beta/meetings/rooms";
  const toSend = {
    type,
     recording_options: { auto_record: Boolean(recording_options) },
    ui_settings,
    display_name,
  };
  
  if (type === "long_term") {
    toSend.expires_at = new Date(expires_at).toISOString();
  }

  const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${jwt}`,
    },
    body: JSON.stringify(toSend),
  };
  return fetch(url, options)
    .then((response) => response.json())
    .then((data) => {
      console.log("Meeting Created", data);
      // check if links are present in the response
      if (!data._links) {
        return res.status(500).json({ error: "Failed to create meeting" });
      }
      // return the guest_url and host_url
      res.status(200).json({
        guest_url: data._links.guest_url.href,
        host_url: data._links.host_url.href,
      });
    })
    .catch((error) => {
      res.status(500).json({ error: "Failed to create meeting" });
    });
```

The code extracts the required data from the request body. As said in the paragraph above, these parameters will be sent by the bot. After it, we need to authenticate the request. We do that by generating the JWT and passing the required parameters `(ApplicationId, privateKey)`

If the meeting type is `'instant'`, the `expires_at` property is not required so I removed it from the body. The response data will contain host and guest URLs.

Don’t forget to set up your `.env` file. The required parameters are: 

```js

```

The applicationId is the Id of the application and the private key is the key generated when you create the application in the Vonage dashboard.

## Listening to Webhooks for Recording, Rooms, and Sessions:

To enhance our meeting capabilities, we will also implement webhook listeners in our Node.js server. Webhooks allow us to receive real-time notifications when specific events occur, such as the start or end of a recording, the creation of new rooms, or updates to session details. By leveraging these webhooks, we can take proactive actions within our application, such as triggering post-meeting actions or updating UI elements accordingly.

## Create the Bot in Vonage AI

Vonage AI is a conversational AI platform built to handle complex interactions between businesses and customers, lowering operational costs and significantly improving service levels. It currently offers the following channels: telephony for a voice based bot, WhatApp and SMS for text based bot and HTTP agents. 
I choose to use Whatsapp bot because it’s one of the most popular messaging platforms and offers different types of messages to interact with the user such as buttons, list messages and many others.

For more details on Vonage AI capabilities, you can find the documentation [here](https://studio.docs.ai.vonage.com/)

### Collect Input for Meeting Type, Recording Option, and UI Language

To initiate the meeting creation process, our WhatsApp bot will prompt the user to provide specific information. The first block we’ll utilize is the "Collect Input" block, which enables us to gather data from the user. Within this block, we can ask the user to specify the meeting type (one-time or long term), recording options (auto_recording or not), and preferred UI language. By capturing this information, we can tailor the meeting experience based on the user's preferences.

For the meeting type, we use WhatsApp reply buttons so the user doesn’t need to type the message but only click the desired type of meeting:

Same logic for the recording options, we will use a Collect Input Node with Reply Buttons.
A different approach is needed for the UI language options, since we have more than three options (we have eight available options). For the UI language options, we will use the List message, where you can add up to 10 options. This type of message shows a list of options to the user. Please see the below screenshot:

### Collect Expiration Date

One input we need to gather during our conversation is the expiration date for long term meetings.  Using another "Collect Input" block, we prompt the user to enter an expiration date for their meeting. 
To proceed, let's dig into the concept of entities within Vonage AI. An entity serves as a pre-defined database comprising a collection of values and their synonyms. These values are essential for Vonage AI to extract and validate specific data from user inputs expressed in natural language. When exploring the entity list, available in the documentation, we'll come across familiar categories such as contacts, dates, email addresses, phone numbers, and more.

Now, the critical aspect at this stage is to select the appropriate entity type for our purpose, which in this case is "@sys.date". By designating the entity as a date type, Vonage AI will intelligently parse the user's input as a date, facilitating its utilization in subsequent steps, particularly within the Webhook Node.

### Webhook Integration with the Node.js API

Once we have collected all the necessary information from the user, we need to connect our WhatsApp bot with the Node.js server we created earlier. To achieve this, we utilize the webhook block provided by Vonage AI. This block allows us to make HTTP requests to external endpoints, enabling seamless integration between different systems. By configuring the webhook block to call the relevant endpoints of our Node.js API, we can pass the collected meeting details and trigger the creation of the meeting.

Is it possible to add a video of the bot?

<youtube id="V3yu6MH7fLs"></youtube>

## Conclusion

In conclusion, the utilization of the Meeting API and Vonage AI in the meeting scheduler exemplifies the ease and user-friendliness of creating self-service, customizable video conferencing experiences for businesses and their customers. By leveraging the Meeting API, developers can effortlessly manage meeting creation, recording options, UI language settings, and more. Combining this functionality with Vonage AI opens up exciting possibilities, allowing users to interact with the bot through popular messaging platforms like WhatsApp. 

You can find the code repo at https://github.com/enricop89/vonage-meetings-bot-scheduler

Did you enjoy this tutorial? Did you get stuck? Reach out on Twitter or the Vonage Community Slack (we even have a channel for AI Studio). We’re excited to see what you’re building with Low Code!