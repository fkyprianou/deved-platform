---
title: Build a Farewell App Using Vonage AI Studio
description: Learn how to build a farewell app using Vonage AI Studio, allowing
  you to say goodbye to colleagues in a unique and memorable way.
author: diana-pham-1
published: true
published_at: 2023-08-02T21:29:18.666Z
updated_at: 2023-08-02T21:29:18.686Z
category: tutorial
tags:
  - "#ai-studio"
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## It’s Not a Goodbye, It’s a See You Later

Before I joined the awesome DevRel team at Vonage, I had the opportunity to work at another company. Saying goodbye to them at the end of my internship was a unique experience, and to show my appreciation, I decided to surprise them with something special.

I came up with an idea to build a special app using an SMS API and a low/no code tool. The app allowed my former coworkers to send their names to a virtual phone number and receive a personalized message I had written for each of them. It was a thoughtful way to say farewell and let them know how much I appreciated their help that summer.

Whether you're deciding to hop teams, companies, or just build this simple app for fun, I'd love to show you how you can recreate this project using Vonage's Messages API and AI Studio. Let's dive in and get started!

## Prerequisite

Before you begin, ensure you have a [Vonage Developer Account](https://developer.vonage.com/sign-up) - If you don't have one, you can create one, and we'll give you free credit to play around with our APIs. 

## Set up Vonage AI Studio

Once that's done, go to your API Dashboard and, on the left-hand side find **BUILD & MANAGE** > **AI Studio** > **Go to AI Studio** > **Create Agent**. You can select any agent type, but for our example we're going to build it with **WhatsApp**.

Fill in all the required details:

* **Region**: Where will your Agent be typically used - The USA or in Europe
* **Agent Name**: Give your Agent a unique name that is meaningful to you.
* **Language**: Select the language of your Agent.
* **Time Zone**: Choose the time zone where your Agent will operate.

For templates, we are going to **Start from Scratch**.

For event, this will be an **Inbound Call** because we are the ones texting our agent.

## Define the Conversational Flow

Now comes the fun part! With Vonage AI Studio's easy-to-use interface, you can design your chatbot's conversational flow. It's a drag-and-drop setup, meaning you don't need coding skills to get going. You can add different dialog nodes and define responses.

Our flow should look like this:

![Conversation Flow for Farewell App](/content/blog/build-a-farewell-app-using-vonage-ai-studio/conversation-flow.png "conversation-flow.png")

NODES > Conversation > Collect Input

The virtual agent will prompt a question to the user.

* Node name: `ASK_NAME`.
* Prompt: What's your name?

NODES > Conversation > Conditions
The user will need to type their name. Here you can create as many conditions as people you want to write notes to.

* Node name: `TYPE_NAME`

Create Condition (button)
I named each condition after the person. (e.g., `Mason`)

* Parameter: INPUT
* Operation: Contains
* Value: Their name

***Note***: My parameter is `KURTICE` because that was his full name. However, because we used the operation 'contains', if Kurtice texted 'kurt' (not case sensitive), it would still send him the correct message. However, if you had a 'Michael' and he typed 'mike', it would send him the generic message (explained in next paragraph).

There were some people on my team who I didn't talk to very much, so I just had a generic thank you message for them. As long as they text *a* name, they will receive a message. I labeled that condition as `Generic`.

* Parameter: INPUT
* Operation: Has value
* Value: N/A

NODES > Action > Send SMS

I named each node after the person in all caps (e.g., `MASON`)

* Text
* Agent says: {Write your message to the recipient}

NODES > Actions > End Conversation

## Wrap Up

Saying goodbye doesn't have to be ordinary. With a touch of creativity and the usage of Vonage's Messages API and AI Studio, you can make farewells heartfelt and unforgettable. So, go ahead and spread some joy with personalized messages that will make them smile one last time. I wish you all the best in your coding adventures!

## Stay in Touch

If you have questions or feedback, join us on the  [Vonage Developer Slack](https://developer.vonage.com/community/slack)  or send me a Tweet on [Twitter](https://twitter.com/dianasoyster). Thanks again for reading, and I will catch you on the next one!