---
title: "Low Code Leverage: AI Studio + Airtable"
description: This tutorial will show you how to create a backend database with
  Airtable for a conversational AI agent in Vonage's AI Studio. All in Low Code!
author: benjamin-aronov
published: true
published_at: 2023-02-26T12:46:26.526Z
updated_at: 2023-02-26T12:46:26.559Z
category: tutorial
tags:
  - ai-studio
  - low-code
  - airtable
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
1. ![]()

## Introduction

A few months ago I was in Dubai for a hackathon. The hotel taxi driver was very friendly and wanted to make sure I made the most of my two-day trip. He began offering all kinds of services, like picking me up at the start and finish of each day of the event, doing different kinds of specialized tours of the city, or even doing a trip to Abu Dhabi.

He gave me his business card with a QR code to message him on WhatsApp. I scanned the code and it took me to a chat window with his business account. But that was it! None of the cool services he had just told me about were presented to users. So my business brain got to working  and I suggested to him, you know you can really help empower your customers with Vonage’s AI Studio!

![An example of a QR code Taxi Driver card. Credit to zazzle.com](/content/blog/low-code-leverage-ai-studio-airtable/driver_taxi_limo_cool_black_metal_qr_code_business_card-r815c59cf1b504530b3dd0614064be5a1_tcvul_736.webp "An example of a QR code Taxi Driver card. Credit to zazzle.com")

Last year Vonage launched AI Studio, a NoCode/LowCode platform that allows anyone to build Conversational AI agents, fast! Typical use cases for Conversational AI are chatbots for customer support or marketing. But chatbots can improve the user experience for so many more applications!

In this tutorial, I’ll show you how to build an AI Studio WhatsApp agent for this taxi service which uses Airtable as a backend database to store and access information. We’ll also use Postman to show how to trigger a real-time message, pulling our user information from Airtable and sending it to AI Studio.

Yalla, let’s go!

## Prerequisites

* Vonage Developer Account. To use AI Studio you'll need a Vonage Developer Account. Details to get started just below.
* Airtable Account - sign up here﻿We'll be using Airtable as our backend database.
* Postman Account - sign up[here](https://identity.getpostman.com/signup). We'll be using Postman to send requests in the advanced section of this tutorial.
* Optional: Miro Account - sign up [here](https://miro.com/signup/). I use Miro to mockup my AI Studio agents. More information about this below.

<sign-up></sign-up>

## Set-Up

### Start With a Mockup

Before building any agent in AI Studio, I highly recommend mapping out your agent's flow using a visual tool. I use Miro for this but there are lots of tools online, or you can even just use a pen and paper. 

The idea is to have a high-fidelity mockup that you can basically just copy over to AI Studio, focusing on the more technical aspects of implementation without worrying about the flow. 
I use the following set of components, this way I can just copy/paste.

﻿First, create a small key of components so you can copy/paste and just add the text for each specific step. Here is what I use:

![A sample key of components for a chatbot mockup](/content/blog/low-code-leverage-ai-studio-airtable/screenshot-2023-02-26-at-15.19.20.png "A sample key of components for a chatbot mockup")

T﻿he cool thing is now you can mockup your bot super fast. You'll need to figure out what outcomes or user journeys you'll want to create for your users. For instance, in our app we'll let users look up taxi prices to popular locations and also add their contact info to subscribe to taxi deals. The mockup for those flow look like this:

![Example of mockup for inbound flows of AI Taxi Demo](/content/blog/low-code-leverage-ai-studio-airtable/screenshot-2023-02-26-at-15.28.40.png "Example of mockup for inbound flows of AI Taxi Demo")

We'll have a separate flow to send the promotional alerts. So our mockup for this:

![Example of mockup for outbound flow of AI Taxi Demo](/content/blog/low-code-leverage-ai-studio-airtable/screenshot-2023-02-26-at-15.28.45.png "Example of mockup for outbound flow of AI Taxi Demo")

N﻿ow that we have a clear picture of what we want to build, let's get building!

### Create an Airtable DB

B﻿efore we can get started with AI Studio, we'll need to setup our Airtable database. Airtable is a cool tool that is like Excel with superpowers! The [documentation](https://airtable.com/developers/web/api/introduction) is fantastic, it's a great place to get started.

O﻿nce you've gotten comfortable, you'll want to build a new base. We'll call it "AI Taxi Demo". In this base we'll have two tables; Destinations and Customers.

F﻿or Destinations, we'll keep it simple. There will be just two fields: Name and Price. 

![Destinations table with fields Name and Price](/content/blog/low-code-leverage-ai-studio-airtable/destinations-table.png "Destinations table with fields Name and Price")

S﻿imilarly for the Customers tables we will just use two fields: Name and Phone_Number:

![Customers table with fields Name and Phone_Number](/content/blog/low-code-leverage-ai-studio-airtable/group-1-40-.png "Customers table with fields Name and Phone_Number")

W﻿e can add some dummy data for these tables so that our API calls return values. Phone numbers should be entered with only digits, no plus sign (+).

## Create a New Agent

N﻿ow that we've got our database set up, we can start to build out our AI Studio agent which will interact with the user and then pass data to and from Airtable. Follow the instructions found in the AI Studio documentation [here](https://studio.docs.ai.vonage.com/ai-studio/create-a-new-agent). There are three important options for our agent, select:

* **T﻿ype:** WhatsApp
* **Template:** S﻿tart From Scratch
* **E﻿vent:** Inbound 

## Retrieving Data From Airtable

N﻿ow let's get started building out our flows. The great thing is that we've basically done all the work already! We just need to turn our vertical flowchart into a horizontal agent.

O﻿ur steps:

1. **S﻿end Message Node**: send a welcome message, "Welcome to AI Taxi"
2. **Collect Input Node:** ask the user, "How can I help you?". The user's response will be stored in a parameter called TOPIC.

> Parameters are like variables, which can be accessed throughout the agent. You can read more about parameters [here](https://studio.docs.ai.vonage.com/properties-1/parameters). Just like variables in code have data types, parameters have [entities](https://studio.docs.ai.vonage.com/properties-1/entities). For simplicity, all parameters will be of type @sys.any.

3. **C﻿lassification Node:** classify the `TOPIC` parameter. We will have two intents: `Retrieve Prices` and `Sign Up For Promotions`. Here we could've used a simple condition since in this example we only have two intents. However, the beauty and power of AI Studio is the Classification feature. We want to allow our users the freedom to enter any request they desire and write it naturally. You can see how powerful and easy to use the Classification node with suggested User Expressions is; [here](https://studio.docs.ai.vonage.com/whatsapp/nodes/conversation/classification).
4. **Collect Input Node:** ask the user, "Where would you like to go?". The user's response will be stored in a parameter called DESTINATION.

N﻿ow we have the search term that we want to send to Airtable and receive back our stored price info. But how can we pass it to Airtable? The great thing about Airtable is their built-in API for each base and helpful documentation. From our AI Taxi Demo base, click on \*help\* in the top right corner. Then in the sidebar, at the very bottom, you will find \*API Documentation\*. The really cool thing is that Airtable generates the required request for you! Here you will find the prebuilt curl request, it will look something like this:

![Example of CURL request to Airtable API](/content/blog/low-code-leverage-ai-studio-airtable/screenshot-2023-02-19-at-14.11.58.png "Example of CURL request to Airtable API")

N﻿ow we'll need to add this to our AI Studio agent. So we'll add a **Webhook node** that allows us to make REST API requests. Learn more about Webhook nodes [here](https://studio.docs.ai.vonage.com/whatsapp/nodes/integrations/webhook).

I﻿n the node, we'll add our endpoint in the Request URL:

`https://api.airtable.com/v0/app4AtCxYJu9tagah/Destinations`

Y﻿ou'll also need to add your Airtable Personal Access Token in the Header parameters under the Headers tab. You can learn how to generate and use your Personal Access Token [here](https://airtable.com/developers/web/guides/personal-access-tokens). Make sure to give it scopes: `data.records:read`and`data.records:write`. This token can only be seen once, so you should save it somewhere safe that you can copy/paste it later.

N﻿ow your webhook node should look like this:

![Example of Lookup Destination Webhook Node](/content/blog/low-code-leverage-ai-studio-airtable/group-2-28-.png "Example of Lookup Destination Webhook Node")

\
We can test that our webhook is working by clicking the `test request` button in the top right. You'll see that it returns all our Destinations in the table. You can see under the response tab what data will be returned:

![Example of Destinations Lookup Response](/content/blog/low-code-leverage-ai-studio-airtable/destinations-response.png "Example of Destinations Lookup Response")

B﻿ut we don't want all the of the destinations, we want to be able to search by our user input. Luckily for us, Airtable has some cool search features. For instance, I've used the `filterByFormula` to create a global table search. We'll use `filterByFormula` here to search against the Destination column. And now our webhook node looks like this:

![Example of Webhook Node with Query Parameters](/content/blog/low-code-leverage-ai-studio-airtable/lookup-with-filterbyformula.png "Example of Webhook Node with Query Parameters")

> \
> You must click on the Query parameters tab and fill out the parameter and value, writing directly into the URL path will not save.

N﻿ow we can run the test again and see that the request returns an object which inside it has something called “records”, which itself contains an array of record objects. 

![Example of Filtered Response](/content/blog/low-code-leverage-ai-studio-airtable/filtered-response.png "Example of Filtered Response")

AI Studio allows us to handle API responses with the [Response Mapping ](https://studio.docs.ai.vonage.com/whatsapp/nodes/integrations/webhook#response-mapping)feature. We need to map the returned object to a parameter that we will then be able to use inside AI Studio. We can do so like this:

![Example of Response Mapping](/content/blog/low-code-leverage-ai-studio-airtable/response-mapping-.png "Example of Response Mapping")

A﻿nd now that we have mapped our response data, we can run the test and see some values returned to our parameters! 

![Example of Response Mapping Test Results](/content/blog/low-code-leverage-ai-studio-airtable/response-mapping-test-results.png "Example of Response Mapping Test Results")

W﻿e did it! We've connected our AI Studio agent to our Airtable data and now we can use this information in our agent. One last step is to use our data now in our agent and make a nice message to our user:

![Example of Send Price Node](/content/blog/low-code-leverage-ai-studio-airtable/send-price-node.png "Example of Send Price Node")



## T﻿riggering an Outbound Event From Airtable

So now that we’ve proven we can send info to and from Airtable, let’s do something a bit more exciting. Let’s take all our customers in our database and send them a message. Notice here the user isn’t initiating the conversation, but rather we’re working with an outbound event. 

### Postman Setup

We’re going to be using Postman to do this trigger action. But when you are building your application you just need to be able to retrieve your contacts and then make the POST request to AI Studio.

First, we’ll need to [create a workspace](https://learning.postman.com/docs/getting-started/creating-your-first-workspace/) on Postman. A workspace allows you to create a [collection](https://learning.postman.com/docs/getting-started/creating-the-first-collection/), with which we can save pieces of information to variables. These [Postman variables](https://learning.postman.com/docs/sending-requests/variables/) allow us to pass data from our first GET request to our second POST request.

### Retrieving Our Contacts From Airtable in Postman

Just as we did earlier, we’ll do a GET request to our Airtable DB and pass our Access Token through in the headers. Your Postman should look something like this:

![Example of Postman GET Request](/content/blog/low-code-leverage-ai-studio-airtable/postman-retrieve-contacts-example.png "Example of Postman GET Request")

\
You can hit send and you should get a response with all the customers in your table. Now we can add the bit of Postman logic to store our customers in variables.



Under the Tests tab, here we’ll add a bit of Javascript:

```javascript
var jsonData = JSON.parse(responseBody);
var bodyData = jsonData.records;
pm.variables.set("retrieved_records", bodyData);
```

This will now let us access our GET response data under the key “retrieved_records”. You can see this by adding the line `console.log(pm.variables);` and opening the [console](https://blog.postman.com/the-postman-console/). Where you should see `retrieved_records` key: 

![Example of Postman Console Logging](/content/blog/low-code-leverage-ai-studio-airtable/postman-console-.png "Example of Postman Console Logging")



### Sending Each Contact To AI Studio With Postman

Now we can use the information stored in our `pm.variables` to iterate and send a POST request to AI Studio for each contact. 

First, we’ll add a new request to our collection. Let’s call it “Trigger Promotional Message” and here we’ll want to change it to a POST request. But where do we want to send our request?\
\
The cool thing is that all outbound WhatsApp agents are triggered by the same endpoint. You just need to pass your X-Vgai-Key and the proper parameters. Read about it [here](<https://studio.docs.ai.vonage.com/whatsapp/get-started/triggering-an-outbound-whatsapp-virtual-agent>)So with our X-Vgai-Key added in the headers, we need to provide the proper parameters to AI Studio. In the body, we’ll pass the following raw JS:

```javascript
{
   "components": [
       {
           "type": "header",
           "parameters": [
               {
                   "type": "text",
                   "text": {{currentName}}
               }
           ]
       }
   ],
   "namespace": "18b6e75b_1bf3_4d19_8c00_a59e16d4fc14",
   "template": "hellodemo",
   "locale": "en",
   "to": {{currentNumber}},
   "agent_id": "639198e1e7db284039394ee6",
   "channel": "whatsapp",
   "status_url": "string"
}

```

But how will we get the `currentName` and `currentNumber` to pass in our POST request? We’ll need to make use of Postman’s pre-request script to iterate through our `retrieved_records` and access each contact’s information to make a POST request. Like so:

```javascript
const records = pm.variables.get('retrieved_Records');
pm.variables.set('currentRecord', records.shift());
const currentRecord = pm.variables.get('currentRecord');
pm.variables.set('currentName',  JSON.stringify(currentRecord.fields.FIRST_NAME));
pm.variables.set('currentNumber',  JSON.stringify(currentRecord.fields.PHONE_NUMBER));


const currentName = pm.variables.get("currentName");


if (records.length > 0){
   console.log(currentName);
   console.log("records length: "+records.length);
   postman.setNextRequest('Trigger Bot');
} else {
postman.setNextRequest(null);
}

```