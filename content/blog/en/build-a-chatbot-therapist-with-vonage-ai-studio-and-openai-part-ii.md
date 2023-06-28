---
title: Build a ChatBot Therapist with Vonage AI Studio and OpenAI (Part II)
description: This blog covers steps in natural language processing and offers a
  step-by-step tutorial on how to build your own.
author: diana-pham-1
published: true
published_at: 2023-06-28T18:34:51.954Z
updated_at: 2023-06-28T18:34:51.974Z
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
Welcome back to the final part of our two-part series where we delve into the world of chatbot therapists! But before we dive into the building our chatbot with [Vonage AI Studio](), let's lay some groundwork by exploring Natural Language Processing (NLP), the powerful engine that drives [OpenAI](https://openai.com/)'s API.

## Unveiling the Magic of Natural Language Processing with OpenAI

What powers our innovative chatbot therapists? The answer lies in Natural Language Processing (NLP). When I say, "I am talking about ChatGPT," OpenAI doesn't just see a string of words. It performs several intricate steps to understand the meaning and context of the sentence.

### Segmenting and Tokenizing 

First, OpenAI breaks down my sentence into smaller parts or 'tokens.' This process is called segmenting. Tokenization is the process of chunking up our prompt to find things it can use as tokens. So "I am talking about ChatGPT" becomes "I," "am," "talking," "about," "ChatGPT." This way, each word can be analyzed individually.

### Stop Words

But not all words are equally informative. Stop words are commonly used words like 'is,' 'an,' 'the,' etc., that may not contain significant meaning in the context of NLP. Therefore, they are often filtered out. In my sentence, "about" might be treated as a stop word.

### Stemming and Lemmatization

Stemming and lemmatization are methods of reducing words to their base form. For instance, "talking" might be stemmed to "talk." Lemmatization is changing words to their dictionary forms, so "am" becomes "be." The goal is to identify the root of a word, which can be particularly useful when dealing with different forms of the same verb or dealing with plurals.


### Part-of-Speech Tagging

Part-of-speech tagging classifies words into their grammatical categories. This helps OpenAI determine how words function within a sentence. For example, in our sentence, "I" is a pronoun, "am" is a verb, "talking" is also a verb, "about" is a preposition, and "ChatGPT" is a noun.

### Named Entity Recognition (NER)

Finally, NER helps OpenAI identify and categorize real-world objects or 'entities' from text. "ChatGPT," for instance, would be identified as an entity related to artificial intelligence.

## Let's Build Our Bot!

Now that we've scratched the surface of how OpenAI understands language, let's get our hands dirty and create our very own chatbot therapist.

However, it's crucial to emphasize that this bot is purely for educational purposes and does NOT serve as a substitute for professional therapy, and any information or guidance provided should not be considered as professional advice. If you require therapeutic assistance, we strongly encourage you to seek the help of a trained professional for any therapeutic concerns to ensure your well-being and receive appropriate support.

### Prerequisites
Before you begin, ensure you have the following:

- A [Vonage Developer Account](https://developer.vonage.com/sign-up) - If you don't have one, you can create one, and we'll give you free credit to play around with our APIs.
- An [OpenAI Account](https://openai.com/product) and an [API Key](https://platform.openai.com/account/api-keys) are free. However, using the OpenAI API is no longer free, so you will need to pay for usage in order to get a reply. * Click “Create new secret key” and store the information somewhere safe since we'll need it later. See example below.
- [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet.
- [Python](https://www.python.org/) - I’m currently using version 3.11.1 for this article.
- [Pip](https://pypi.org/project/pip/) is installed - Double-check that you can run pip from your terminal or command prompt.

### Step 1: Create the Server and Integrate OpenAI API
We’ll need to create a server application that AI Studio will call out to once the user types to the "therapist." To do so, we’ll use a short Python script to start a server on port 9000 using our OpenAI credentials. Please note that I'm using Python, but you can create the same using your language of choice.

```python
from datetime import datetime, timedelta
import os
from pickle import TRUE
from urllib import response
import openai
from PIL import Image
from flask import Flask, request, jsonify, render_template, Response, redirect
from flask_login import LoginManager, UserMixin, login_required, login_user

application = Flask(__name__)

app = application

openai_key = 'YOUR_API_KEY'
openai.api_key = 'OPENAI_KEY'

@app.route("/webhook", methods=["POST"])
def webhook():
    input = request.headers.get('input')

    # You must give OpenAI a prompt to ask as a mental health therapist
    prompt = f"As an empathetic and understanding mental health therapist, how would you respond to the following message: '{input}. Try to include open-ended questions to encourage further conversation."
    model_engine = "text-davinci-003"
    completions = openai.Completion.create(engine=model_engine, prompt=prompt, max_tokens=1024, n=1,
                                               stop=None,
                                               temperature=0.4)
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user",
         "content": input},
    ]

    # Get the response text
    response = completions.choices[0].text
    return {'res': response.strip()}
if __name__ == '__main__':
    app.run(port=9000, host='0.0.0.0', debug=True)  # run app in debug mode on port 5000 (edited)

```

We made a few tweaks that could potentially improve the model's responses:

**Prompt**
- Ensure Empathy: You might want to include language in your prompt that emphasizes empathetic and supportive responses.
- Open-ended Questions: Encourage the model to ask open-ended questions to invite further discussion.

**Temperature**
Choosing the right temperature parameter depends on the desired behavior of your chatbot. However, since a mental health therapist would likely be providing consistent, focused, and precise responses, a lower temperature value might be suitable. A value between 0.2 and 0.5 could be a good starting point.

Here's a general guideline:

A low temperature value (around 0.2 - 0.5) will make the model's output more deterministic and consistent, which is helpful when you want the chatbot to stick to facts or be precise.

A high temperature value (around 0.7 - 1.0) will make the model's output more diverse and creative. This can lead to more engaging and conversational outputs but might also cause less predictability and precision in responses.

Remember, this is just a starting point. It's crucial to continuously monitor and adjust these parameters based on the actual performance of the chatbot in production, to ensure the best possible user experience. It's also important to ensure your bot responds in a manner that is safe and appropriate, given the sensitivity of mental health discussions.

Once your application is up and running, you'll get a deployment URL. For example, mine is http://10.0.0.190:9000/. Now, to give AI Studio the ability to call this URL, we'll use [ngrok](https://ngrok.com/).

### Step 2: Run ngrok

ngrok is a cross-platform application that allows developers to easily expose a local development server to the Internet. We'll be using it to expose our service to the Internet. If this is your first time using ngrok, a [blog](https://developer.vonage.com/en/blog/using-ngrok-in-rails-in-2022#join-the-conversation) post explains it in more detail. Once you have ngrok setup and are logged in, run the following command in your terminal:

```
ngrok http 9000
```

After ngrok runs, it will give you a Forwarding URL that we'll use as the base for our Webhooks later in the article. Mine looks like this:

[IMAGE]

Now, if I wanted to access my local python server through AI Studio, I’d use the Forwarding address shown in the screenshot.

### Step 3: Set up Vonage AI Studio

To kick things off, you need to create an account with [Vonage AI Studio](https://dashboard.nexmo.com/studio). Once that's done, click **Create Agent**. You can select any agent type, but for our example we're going to build it with **Telephony**.

Fill in all the required details:
- **Region**: Where will your Agent be typically used - The USA or in Europe
- **Agent Name**: Give your Agent a unique name that is meaningful to you.
- **Language**: Select the language of your Agent.
- **Voices**: Select any voice (I’m using Matthew).
- **Time Zone**: Choose the time zone where your Agent will operate.

For templates, we are going to **Start from Scratch**.

For event, this will be an **Inbound Call** because we are the ones calling our agent.

### Step 4: Define the Conversational Flow

Now comes the fun part! With Vonage AI Studio's easy-to-use interface, you can design your chatbot's conversational flow. It's a drag-and-drop setup, meaning you don't need coding skills to get going. You can add different dialog nodes and define responses.

Ours should look like this:

[IMAGE]

-   2 Parameters to store the value that the user enters and the response that OpenAI responds with.

    -   Custom Parameters
    -   Name: AGENT_RES and Entity is @sys.any
    -   Name: INPUT and Entity is @sys.any

-   **Collect Input Node** - That welcomes the user and collects their input.
    
    -   Parameter: Use the INPUT Parameter that we created in the previous step.
    -   Prompt Text: Hello there! I'm your digital companion, here to listen and chat about anything on your mind. Remember, this is a safe space where you can freely express your feelings. What would you like to talk about today?
        -   When we run the Agent, we’ll talk to it just as we would to a normal person. For example, **"I've been having a lot of trouble sleeping and it's starting to affect my mood."**

-   **Webhook Node** -
    
    -   Request URL
        -   Request Type: POST
        -   URL - Should look like the following but substitute my ngrok URL with your own  [https://db20-2601-600-9580-d650-cfa-951f-cdef-d1d5.ngrok.io/webhook](https://db20-2601-600-9580-d650-cfa-951f-cdef-d1d5.ngrok.io/webhook)

        ---
        **NOTE**
        Make sure you copy the Forwarding URL from your terminal where you ran ngrok and add /webhook to the end of it.
        ---

    -   Headers:
        -   HTTP Header - INPUT and Value is $INPUT

    -   Response Mapping:
        -   Object Path - res
        -   Value is $AGENT_RES

-   **Speak Node** - To output the results from OpenAI.
    -   Text - Your answer is: $AGENT_RES

-   **End Call Node** - To end the conversation once the conversation flow has completed.

### Step 6: Run the Application

Let's run the application by pressing the **Tester** button at the top right-hand corner of the screen and see what type of output we get. To begin, we see the welcome message.

Here are some examples of things you can say:
    1. I've been feeling really overwhelmed with work lately.
    2. I'm going through a breakup and I don't know how to cope.
    3. I've been feeling a lot of anxiety about the future recently.
    4. I don't have anyone to talk to about my feelings.
    5. I'm feeling down and I don't really know why.

Below is an example conversation I had with our chatbot therapist. Keep in mind that in our conversation flow, we put an End Call Node right after we receive our output in the Speak Node. Normally, a therapist wouldn't just hang up on you after responding to your message–that'd be a horrible therapist! If we wanted to, we can modify our flow to continue back-and-forth conversations.



And voilà, you've built your very own chatbot therapist! Remember, the power of AI holds enormous potential, but it's up to us to use it responsibly. With these tools at your disposal, you're well on your way to making mental health care more accessible and innovative.

## Stay in Touch

If you have questions or feedback, join us on the  [Vonage Developer Slack](https://developer.vonage.com/community/slack)  or send me a Tweet on [Twitter](https://twitter.com/dianasoyster). Thanks again for reading, and I will catch you on the next one!