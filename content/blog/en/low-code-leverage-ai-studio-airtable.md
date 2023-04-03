---
title: "Low Code Leverage: AI Studio with Airtable"
description: This tutorial will show you how to create a backend database with
  Airtable for a Conversational AI agent in Vonage's AI Studio. All in Low Code!
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
## Introduction

A few months ago I was in Dubai for a hackathon. The hotel taxi driver was very friendly and wanted to make sure I made the most of my two-day trip. He began offering all kinds of services, like picking me up at the start and finish of each day of the event, doing different kinds of specialized tours of the city, or even doing a trip to Abu Dhabi.

He gave me his business card with a QR code to message him on WhatsApp. I scanned the code and it took me to a chat window with his business account. But that was it! None of the cool services he had just told me about were presented to users. So my business brain got to working  and I suggested to him, you know you can really help empower your customers with Vonage’s AI Studio!

![An example of a QR code Taxi Driver card. Credit to zazzle.com](/content/blog/low-code-leverage-ai-studio-with-airtable/image-1-6-.png "An example of a QR code Taxi Driver card. Credit to zazzle.com")

Last year Vonage launched [AI Studio](https://studio.docs.ai.vonage.com/), a NoCode/LowCode platform that allows anyone to build Conversational AI agents, fast! Typical use cases for Conversational AI are chatbots for customer support or marketing. But chatbots can improve the user experience for so many more applications!

In this tutorial, I’ll show you how to build an AI Studio WhatsApp agent for this taxi service which uses Airtable as a backend database to store and access information.

Yalla, let’s go!

## Prerequisites

* Vonage Developer Account. To use AI Studio you'll need a Vonage Developer Account. Details to get started just below.
* Airtable Account - sign up [here﻿](https://airtable.com/signup). We'll be using Airtable as our backend database.
* Postman Account - sign up [here](https://identity.getpostman.com/signup). We'll be using Postman to send requests in the advanced section of this tutorial.
* Optional: Miro Account - sign up [here](https://miro.com/signup/). I use Miro to mockup my AI Studio agents. More information about this below.

<sign-up></sign-up>

## Set-Up

### Start With a Mockup

Before building any agent in AI Studio, I highly recommend mapping out your agent's flow using a visual tool. You can read more about creating a high-fidelity Conversational AI mockup **here**.

In our app we'll let users look up taxi prices to popular locations and also add their contact info to subscribe to future taxi deals. The mockup for those flow look like this:

![Example of mockup for inbound flows of AI Taxi Demo](blob:https://learn.vonage.com/2809d326-8c4f-43db-b91c-e952f27293a5 "Example of mockup for inbound flows of AI Taxi Demo")

N﻿ow that we have a clear picture of what we want to build, let's get building!

### Create an Airtable DB

B﻿efore we can get started with AI Studio, we'll need to setup our Airtable database. Airtable is a cool tool that is like Excel with superpowers! The [documentation](https://airtable.com/developers/web/api/introduction) is fantastic and a great place to get started.

O﻿nce you've gotten comfortable, you'll want to build a new base. We'll call it "AI Taxi Demo". In this base we'll have two tables: Destinations and Customers.

F﻿or Destinations, we'll keep it simple. There will be just two fields: Name and Price. 

![Destinations table with fields Name and Price](/content/blog/low-code-leverage-ai-studio-with-airtable/destinations-table.png "Destinations table with fields Name and Price")

S﻿imilarly for the Customers tables we will just use two fields: Name and Phone_Number:

![Customers table with fields Name and Phone_Number](/content/blog/low-code-leverage-ai-studio-with-airtable/customers-table.png "Customers table with fields Name and Phone_Number")

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

N﻿ow we have the search term that we want to send to Airtable and receive back our stored price info. But how can we pass it to Airtable? The great thing about Airtable is their built-in API for each base and helpful documentation. From our AI Taxi Demo base, click on **help** in the top right corner. Then in the sidebar, at the very bottom, you will find **API Documentation**. The really cool thing is that Airtable generates the required request for you! Here you will find the prebuilt curl request, it will look something like this:

![Example of CURL request to Airtable API](blob:https://learn.vonage.com/ff6ec57a-1835-427d-aa17-5b8a28aaf48b)

N﻿ow we'll need to add this to our AI Studio agent. So we'll add a **Webhook node** that allows us to make REST API requests. Learn more about Webhook nodes [here](https://studio.docs.ai.vonage.com/whatsapp/nodes/integrations/webhook).

I﻿n the node, we'll add our endpoint in the Request URL:

`https://api.airtable.com/v0/app4AtCxYJu9tagah/Destinations`

Y﻿ou'll also need to add your Airtable Personal Access Token in the Header parameters under the Headers tab. You can learn how to generate and use your Personal Access Token [here](https://airtable.com/developers/web/guides/personal-access-tokens). Make sure to give it scopes: `data.records:read` and  `data.records:write`. This token can only be seen once, so you should save it somewhere safe that you can copy/paste it later.

N﻿ow your webhook node should look like this:

![Example of Lookup Destination Webhook Node](/content/blog/low-code-leverage-ai-studio-with-airtable/group-2-28-.png "Example of Lookup Destination Webhook Node")

We can test that our webhook is working by clicking the `test request` button in the top right. You'll see that it returns all our Destinations in the table. You can see under the response tab what data will be returned:

![Example of Destinations Lookup Response](/content/blog/low-code-leverage-ai-studio-with-airtable/destinations-response.png "Example of Destinations Lookup Response")

B﻿ut we don't want all the of the destinations, we want to be able to search by our user input. Luckily for us, Airtable has some cool search features. For instance, I've used the `filterByFormula` to create a global table search. We'll use `filterByFormula` here to search against the Destination column. And now our webhook node looks like this:

![Example of Webhook Node with Query Parameters](/content/blog/low-code-leverage-ai-studio-with-airtable/lookup-with-filterbyformula.png "Example of Webhook Node with Query Parameters")

> You must click on the Query parameters tab and fill out the parameter and value, writing directly into the URL path will not save.

N﻿ow we can run the test again and see that the request returns an object which inside it has something called “records”, which itself contains an array of record objects. 

![Example of Filtered Response](/content/blog/low-code-leverage-ai-studio-with-airtable/filtered-response.png "Example of Filtered Response")

AI Studio allows us to handle API responses with the [Response Mapping ](https://studio.docs.ai.vonage.com/whatsapp/nodes/integrations/webhook#response-mapping)feature. We need to map the returned object to a parameter that we will then be able to use inside AI Studio. We can do so like this:

![Example of Response Mapping](/content/blog/low-code-leverage-ai-studio-with-airtable/response-mapping-.png "Example of Response Mapping")

A﻿nd now that we have mapped our response data, we can run the test and see some values returned to our parameters! 

![Example of Response Mapping Test Results](/content/blog/low-code-leverage-ai-studio-with-airtable/response-mapping-test-results.png "Example of Response Mapping Test Results")

W﻿e did it! We've connected our AI Studio agent to our Airtable data and now we can use this information in our agent. One last step is to use our data now in our agent and make a nice message to our user:

![Example of Send Price Node](/content/blog/low-code-leverage-ai-studio-with-airtable/send-price-node.png "Example of Send Price Node")

I﻿f we open our [tester](https://studio.docs.ai.vonage.com/ai-studio/chatbot-tester), we can now see the full user journey to ask for a taxi price.

![Testing Our Completed Flow](/content/blog/low-code-leverage-ai-studio-with-airtable/leverage-low-code-gif-1.gif "Testing Our Completed Flow")

## Sending Data To Airtable

Now that we’ve connected with our Airtable DB and begun receiving information, sending information will be a piece of cake!

We’ll start again from our Topic Classification node. We’ll add a new Intent called “Sign Up For Promotions” and add some user expressions. Something like this:

![Example of Adding User Expressions To Training Set](/content/blog/low-code-leverage-ai-studio-with-airtable/training-set-user-expressions-1.png "Example of Adding User Expressions To Training Set")

Now we are ready to connect to our webhook. So let’s take a look at the Airtable documentation again. It shows us exactly how to format our request. We find in the generated request something like this:

![Example of Airtable POST Request](/content/blog/low-code-leverage-ai-studio-with-airtable/example-airtable-post-request.png "Example of Airtable POST Request")

So now just need to open our new webhook node. First we change our request type from GET to POST. We can see that we again need to pass our personal access token in the Authorization key in the header. But now we have a second field to pass as well, we add Content-Type to tell that we are sending json with the request. And finally we’ll pass through our data in the body, with a object that has records of customers.

![Example of POST request webhook node](/content/blog/low-code-leverage-ai-studio-with-airtable/group-3-20-.png "Example of POST request webhook node")

First let’s pass through a test customer. Let’s say, Miss Piggy.

![Example of sending Miss Piggy customer to Airtable](/content/blog/low-code-leverage-ai-studio-with-airtable/miss-piggy-example.png "Example of sending Miss Piggy customer to Airtable")

And now if we hit Test Request, we should see a new entry in Airtable.

![New entry in Airtable; Miss Piggy](/content/blog/low-code-leverage-ai-studio-with-airtable/saved-test-customer.png "New entry in Airtable; Miss Piggy")

But we don’t want to add Miss Piggy in our database! We want to add our actual user. Now you could ask for the user’s name and number here and if you require more information, you’ll probably need to use some collect input nodes here. But for this app, we just care about the user name and number. We can use AI Studio’s built-in system parameters for this! So we update Miss Piggy’s name and phone number with the system parameters `$PROFILE_NAME`and `$SENDER_PHONE_NUMBER`﻿. So our request body now looks like this:

![Example of POST request with dynamic variables](/content/blog/low-code-leverage-ai-studio-with-airtable/post-request-with-variables.png "Example of POST request with dynamic variables")

Lastly, let’s add a thank you message with a send message node. And now we can test this flow:

![Testing Our Completed Flow 2](/content/blog/low-code-leverage-ai-studio-with-airtable/leverage-low-code-gif-2.gif "Testing Our Completed Flow 2")

## Conclusion

Now you've seen how to implement an AI Studio with an easy-to-use Airtable database. What will you build with your new skills?  Please let me know on [Twitter](https://twitter.com/AronovBenjamin) or the [Vonage Community Slack](https://developer.vonage.com/en/community/slack) (we even have a channel for AI Studio). I’m really interested to see what you’re building with Low Code!