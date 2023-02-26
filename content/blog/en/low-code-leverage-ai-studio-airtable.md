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
## Introduction

A few months ago I was in Dubai for a hackathon. The hotel taxi driver was very friendly and wanted to make sure I made the most of my two-day trip. He began offering all kinds of services, like picking me up at the start and finish of each day of the event, doing different kinds of specialized tours of the city, or even doing a trip to Abu Dhabi.

He gave me his business card with a QR code to message him on WhatsApp. I scanned the code and it took me to a chat window with his business account. But that was it! None of the cool services he had just told me about were presented to users. So my business brain got to working. I suggested to him, you know you can really help empower your customers with Vonage’s AI Studio! 

![](/content/blog/low-code-leverage-ai-studio-airtable/driver_taxi_limo_cool_black_metal_qr_code_business_card-r815c59cf1b504530b3dd0614064be5a1_tcvul_736.webp)

Last year Vonage launched AI Studio, a NoCode/LowCode platform that allows anyone to build Conversational AI agents, fast! Typical usecases for Conversational AI are chatbots for customer support or marketing. But chatbots can improve the user experience for so many more applications! 

In this tutorial, I’ll show you how to build an AI Studio WhatsApp agent for this taxi service which uses Airtable as a backend database to store and access information. We’ll also use Postman to show how to trigger a real-time message, pulling our user information from Airtable and sending it to AI Studio.

Yalla, let’s go!

## Prerequisites

* Vonage Developer Account. To use AI Studio you'll need a Vonage Developer Account. Details to get started just below.
* Airtable Account - sign up here﻿We'll be using Airtable as our backend database.
* Postman Account - sign up[here](https://identity.getpostman.com/signup). We'll be using Postman to send requests in the advanced section of this tutorial.
* Optional: Miro Account - sign up [here](https://miro.com/signup/). I use Miro to mockup my AI Studio agents. More information about this below.