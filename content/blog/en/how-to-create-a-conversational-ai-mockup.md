---
title: How to Create a Conversational AI Mockup
description: A best practices guide to help people sketch out their chatbots by
  creating a mockup before building with AI Studio.
author: benjamin-aronov
published: true
published_at: 2023-03-07T15:48:22.516Z
updated_at: 2023-03-07T15:48:22.543Z
category: tutorial
tags:
  - ai-studio
  - lowcode
  - nocode
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
So you want to start building Conversational AI agents? That’s awesome! But where do you begin? Vonage’s AI Studio has created a super simple drag-and-drop solution that lets you get going fast. But even [AI Studio](https://studio.docs.ai.vonage.com/), with its [nodes](https://studio.docs.ai.vonage.com/voice/nodes), [parameters](https://studio.docs.ai.vonage.com/properties-1/parameters), [entities](https://studio.docs.ai.vonage.com/properties-1/entities), and more, can be overwhelming when you sit down to build. How can you know what should go where? With a mockup (flowchart), of course!

Maybe you think mockups are a waste of time and you can just start dragging and dropping. But I will argue that mockups actually save you time in the long run. Some important advantages of starting with a mockup:

* Visualization: A mockup provides a visual representation of the design, giving all your stakeholders an exact idea of how the final product will look and function. Visualization helps ensure that everyone is on the same page regarding the flow and design of your agent.
* Feedback: By creating a mockup, designers and other stakeholders can test the design by walking through the flow and imagining any unforeseen contingencies. By seeing the flow, designers can catch any oversights.
* Efficiency: A mockup can save time and resources by identifying potential issues and roadblocks early in the design process. This can prevent costly mistakes or redesigns later on. While the AI Studio is surely fast for building, being able to make changes in a picture without having to reconfigure nodes is even faster! For instance, when you set an [entity](https://studio.docs.ai.vonage.com/properties-1/entities) for a [parameter](https://studio.docs.ai.vonage.com/properties-1/parameters), it cannot be changed in the future, so a mockup can be useful to determine the correct entity (data type).

So now that I’ve convinced you of the importance of mockups, let’s see how to build a good one.

## High Fidelity Mockups

Any degree of flowchart which helps you visualize your agent will provide your team with greater direction. However, I believe there are a few key components of a mockup that really make it “high fidelity”.

* Node Representation - A mockup must account for all node types that may be used in an agent. Having a clear visual cue to which node is used in each specific case will save lots of time for the agent builder.
* Variable Accountability - How are variables handled? A mockup must account for required variables (called parameters in AI Studio) and how they will be used throughout the flow. If we can follow all instances of our variables, then we can easily determine which data type is required. 
* User Result Satisfaction - Are there clear user journeys that follow a logical and coherent flow in your mockup? Is it easy to follow the user choices as junctions in the flow? Is the result achieved by a user satisfactory? The user might not be happy with the result, but does the conversation at least satisfy their intent? I.e. the agent may not be able to help a user find their order but does the agent ask the correct questions and does the agent supply another way the user can find the resources they are looking for? 

## Mockup Tools

I personally have used Miro for my mockups, but there are many good options out there. 

* [Miro](http://miro.com) - Lots of shapes, colors, connectors, and text, Miro has everything I need.
* [Lucidchart](http://lucidchart.com) - Some of my colleagues who work as Conversational Designers in AI Studio prefer Lucidchart. It’s supposed to be pretty great!
* [Draw.io](http://draw.io) - a free, open-source builder that has everything you need to build agents.
* Pen and paper - You don’t need fancy software for mockups! You can use pen and paper or a whiteboard, just get drawing and get going,

## Start With a Key

As mentioned before, having a complete representation of all your different types of [conversation](https://studio.docs.ai.vonage.com/whatsapp/nodes/conversation) nodes is essential. When creating a key, think about all the different functionalities in your agent. Luckily for us, with AI Studio we already have a blueprint for a key. Here is a simple key of components I use:

![Example Key of Mockup Componenets](/content/blog/how-to-create-a-conversational-ai-mockup/screenshot-2023-03-07-at-13.04.49.png "Example Key of Mockup Componenets")

## Literally, Be Literal

When using a component, write exactly what will happen in the agent. If this is sending a message, write the message text. If it’s writing a condition, what will the condition check? If it’s classification, what variable will be classified? An example of this:

![Example of a Mockup Flow](/content/blog/how-to-create-a-conversational-ai-mockup/screenshot-2023-03-07-at-13.10.14.png "Example of a Mockup Flow")

\
In the above example, we can see how variables are also treated. The `$Topic` variable is set in the [Collect Input](https://studio.docs.ai.vonage.com/whatsapp/nodes/conversation/collect-input) node and then used in the Classification node.

## Ready to Build

After building out your mockup, you might end up with many different flows for your agent. There may still be some lingering questions like error handling, the exact implementation of integrations or webhooks, or if the text in certain nodes should be edited.  But in general, you will have a picture-perfect representation of your agent, so that all stakeholders can agree on the possible outcomes and flows. Below you can see an example of a mockup I made for an agent we used to help conference attendees last year at [apiDays Paris](https://developer.vonage.com/en/blog/closing-2022-with-apidays-and-devcity-paris#connecting-with-more-developers).

![Example of a complicated mockup](/content/blog/how-to-create-a-conversational-ai-mockup/apidays-flowchart.png "Example of a complicated mockup")

## Conclusion

Was this guide on mockups and flowcharts useful for you? Are you convinced that you should start your AI Studio building with a mockup? Please let me know on [Twitter](https://twitter.com/AronovBenjamin) or the [Vonage Community Slack](https://developer.vonage.com/en/community/slack) (we even have a channel for AI Studio). I’m really interested to see what you’re building with Low Code!