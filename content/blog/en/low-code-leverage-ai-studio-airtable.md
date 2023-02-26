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
* ![]()

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