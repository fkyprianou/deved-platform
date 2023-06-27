---
title: Build a ChatBot Therapist with Vonage AI Studio and OpenAI (Part I)
description: This blog covers the history of chatbot therapists and introduces
  how to build your own.
author: diana-pham-1
published: true
published_at: 2023-06-19T19:51:49.759Z
updated_at: 2023-06-19T19:51:49.781Z
category: tutorial
tags:
  - ai-studio
  - chatbot
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Hotdogs or Healthcare?

The mental health care system in the U.S. is, to put it mildly, not great. The challenge is that getting access to therapy is like finding a needle in a haystack. A striking statistic reveals that six out of ten psychologists report they are not accepting new patients, showing a concerning gap in access to service. There are only about 30 psychologists for every 100,000 people. To put this into perspective, Ohio State stadium, with a capacity almost equivalent to this population. That's like saying for everyone jam-packed in the stadium, there are likely more people selling you popcorn and hot dogs than there are psychologists available to help treat you. So what do we do? Well, one creative solution is chatbot therapists. Chatbot therapists are conversational AI-powered bots designed to help mental health patients. They are interactive software platforms designed to simulate human conversation, offering therapeutic interactions, predominantly through text-based channels.

## History of Chatbot Therapists

Now, chatbot therapists have a bit of history. ELIZA was one of the first built to show superficial computer-to-human interactions. ALICE came along to up the game as an improved version. However, it was Misuku that really showed us how a chatbot could lend an 'ear.' These innovative technologies have each significantly evolved the conversation (pun intended) around digital mental health solutions.

### Eliza

ELIZA, created in 1966, was the brainchild of Joseph Weizenbaum. She was the first real attempt to outsmart the Turing test, aiming to show just how similar computer-to-human interactions could seem. ELIZA used different scripts (like acting as a "doctor") and pattern matching to keep the conversation going. This involved searching sentences for keywords (like "hamster") and assigning values to these words to craft a response. For example, with words like "hamster" she may follow up asking about your pets. This is now known as the 'ELIZA Effect,' where we unconsciously believe that a computer's behaviors are just like our own.

As expected, ELIZA had her shortcomings. She didn't really understand the words she was processing; she simply reflected the user's input back as a question. Plus, her memory was short-lived and she lacked understanding of the broader world (like asking, "is your hamster still dead?" because she doesn't know that once something is dead, they stay dead).

Here's an example of a conversation with ELIZA:

![Conversation with ELIZA](/content/blog/build-a-chatbot-therapist-with-vonage-ai-studio-and-openai-part-i/eliza-convo.png "eliza-convo.png")

*https://www.codecademy.com/article/history-of-chatbots*

### ALICE

Fast forward to 2001, and along comes ALICE (Artificial Linguistic Internet Computer Entity). Richard Wallace had been tinkering on ALICE since 1995, and eventually wrote her in Java in 1998. ALICE built upon what ELIZA had started and even won the Loebner Prize three times for being the most human-like bot. Her use of simple patterns and templates to represent inputs and outputs made ALICE easier to build, a method called input partitioning.

### Mitsuku (now Kuki)

Then, there's Mitsuku, the creation of Steve Worswick. Her goal was to be a 24/7 virtual friend. Mitsuku won the AI Loebner Prize five times, showing off her ability to reason with specific objects. She's built on a supervised learning model, which means developers can fine-tune the rules to make her interactions feel more human. However, she still had her issues as seen in the conversation below.

![Conversation with Mitsuku](/content/blog/build-a-chatbot-therapist-with-vonage-ai-studio-and-openai-part-i/mitsuku-convo.png "mitsuku-convo.png")

### ChatGPT

Now, OpenAI's ChatGPT is stepping up, not just chatting but offering a little therapy.

Despite these developments, ethical concerns remain surrounding reliance on chatbot therapists. The current technology is far from perfect, and there is still the potential for misuse, with prompts potentially steering the bot to produce undesirable or even harmful responses.

On the bright side, some pretty cool results came from using ChatGPT as a therapist. Take Michelle Huang, who fed her teenage diary entries from age 7 to 18 into ChatGPT, basically creating a digital version of her old self. It was like she was talking to her younger self and got some real closure about her past. 

## Build-A-Bot

And here's where it gets exciting–we're now at a point where we can use tools like [Vonage AI Studio](https://studio.docs.ai.vonage.com/) and [OpenAI](https://openai.com/) to create our own chatbot therapists! This could be a game-changer for making mental health care more approachable and cutting-edge, as long as we remember to use it cautiously.

Vonage AI Studio is like a handy toolbox for creating intricate conversational flows, and the best part is you don't need to be a coder to use it. It's a low-code or even no-code solution that works using a simple drag-and-drop method. We will pair AI Studio with OpenAI's API for its ability to comprehend and create natural language.

Stay tuned for Part 2, where we'll dig into how we can use tools like Vonage AI Studio and the OpenAI API to build our own chatbot therapists and why it matters in the landscape of mental health care.

## References and Resources

* [Vonage AI Studio Documentation](https://studio.docs.ai.vonage.com/)
* [OpenAI Documentation](https://platform.openai.com/docs/)
* [Investing in a Diverse Mental Health Workforce Is Critical in This Moment](https://healthcity.bmc.org/policy-and-industry/investing-diverse-mental-health-workforce-critical-moment#:~:text=According%20to%20a%202020%20report,even%20lower%20in%20rural%20communities.)
* [The ELIZA Effect](https://99percentinvisible.org/episode/the-eliza-effect/)
* [ALICE chatbot: Trials and outputs](https://www.researchgate.net/publication/289684788_ALICE_chatbot_Trials_and_outputs)
* [Try Kuki (Mitsuku)](https://www.kuki.ai/)
* [History of Chatbots](https://www.codecademy.com/article/history-of-chatbots)
* [Twitter Thread by Michelle Huang](https://twitter.com/michellehuang42/status/1597005489413713921?lang=en)

## Stay in Touch

If you have questions or feedback, join us on the  [Vonage Developer Slack](https://developer.vonage.com/community/slack)  or send me a Tweet on [Twitter](https://twitter.com/dianasoyster). Thanks again for reading, and I will catch you on the next one!