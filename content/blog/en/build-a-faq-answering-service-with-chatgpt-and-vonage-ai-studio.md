---
title: Build a FAQ Answering Service with OpenAI and Vonage AI Studio
description: Learn how to utilize OpenAI and Vonage AI Studio to build a FAQ
  Answering Service
author: michael-crump
published: true
published_at: 2023-02-15T18:46:42.627Z
updated_at: 2023-02-15T18:46:42.686Z
category: tutorial
tags:
  - ai-studio
  - voice-api
  - gpt
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
[ChatGPT](https://openai.com/blog/chatgpt/)is a state-of-the-art conversational language model developed by [OpenAI](https://openai.com/). It is based on [transformer ](https://en.wikipedia.org/wiki/Transformer_(machine_learning_model))architecture and has been trained on a diverse range of internet text, allowing it to generate human-like responses to various questions and prompts. It can also write computer programs, compose music, write poetry, and more! Below is a sample poem it wrote for Vonage’s Communication APIs.  

![Write a poem about Vonage](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/vonagepoem.png "vonagepoem.png")

As you can see, ChatGPT is very powerful. We’ll use the GPT API today to create a FAQ (Frequently Asked Questions) answering service with [Vonage AI Studio](https://www.vonage.com/communications-apis/ai-studio/?icmp=l3nav%7Cl3nav_gototheaistudiooverviewpage_novalue), a Low-Code / No-Code conversational AI platform that helps businesses handle complex customer interactions through voice and text.

What benefits does using GPT-3 as a base model bring in a FAQ answering service with  [Vonage AI Studio](https://www.vonage.com/communications-apis/ai-studio/?icmp=l3nav%7Cl3nav_gototheaistudiooverviewpage_novalue)? Since OpenAI uses [Large Language Models (LLM)](https://www.nvidia.com/en-us/deep-learning-ai/solutions/large-language-models/#:~:text=Codify%20Intelligence%20with%20Large%20Language,transforming%20domains%20through%20learned%20knowledge.) technology, it reduces the time spent training an Agent. Instead of manually creating intents and activity sets, developers can input a bulk of text and set fallbacks with custom responses. Additionally, it allows the opportunity to fine-tune the Agent to answer queries beyond the training data provided. 

For example, we might train our model today by inputting potential customer questions, creating intents for each question, addressing utterances, testing for input mismatch, and optimizing based on results. In contrast, we will train our data tomorrow by inputting a bulk of text instead of intents and creating fallbacks with custom responses.

## Prerequisites

Before you begin, ensure you have completed the following:

* A [Vonage Developer Account](https://developer.vonage.com/en/) - If you don't have one, you can create one, and we'll give you free credit to play around with our APIs.
* An [OpenAI Account](https://openai.com/api/) and an [API Key](https://platform.openai.com/account/api-keys) are free and needed to get a reply from the user's input. We’ll use GTP-3 as a base model for this example. You can learn more about this model and others by visiting their [documentation](https://platform.openai.com/docs/models/overview). 
          * Press “**Create new secret key”** and store the information somewhere safe, as we’ll use it later. See the example below.
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet.
* [Python ](https://www.python.org/)is installed - I’m currently using version 3.11.1 for this article.
* [Pip ](https://pypi.org/project/pip/)is installed - Double-check that you can run `pip` from your terminal or command prompt. 

![OpenAI API Key Dialog](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/apikeys.png "apikeys.png")

## Creating the Server

We’ll need to create a server application that AI Studio will call out to once the user types a question from their phone. 

To do so, we’ll use a short Python script to start a server on port 9000 using our OpenAI credentials, as shown below: 

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

openai_key = 'YOUR-OWN-OPENAI-API-KEY'
openai.api_key = openai_key

@app.route("/webhook", methods=["POST"])
def webhook():
    input = request.headers.get('input')
    model_engine = "text-davinci-003"
    completions = openai.Completion.create(engine=model_engine, prompt=input, max_tokens=1024, n=1, stop=None, temperature=0.5)
    # Get the response text
    response = completions.choices[0].text
    return {'res': response.strip()}

if __name__ == '__main__':
    app.run(port=9000, host='0.0.0.0', debug=True)  # run app in debug mode on port 5000
```

To use this code: 

* Copy all of the Python code provided to a file named `application.py.`
* Replace *YOUR-OWN-OPENAI-API-KEY* with the OpenAI API Key we created earlier. 
* Install any packages that the script is dependent upon

  * `sudo pip install openai` - For OpenAI libraries
  * `sudo pip install flask_login` - For our Website
  * `sudo pip install pillow` - For Imaging libraries

Once the application starts successfully, you’ll get a deployment URL. For example, mine is <http://10.0.0.190:9000/>, and for us to call that URL through AI Studio, we’ll need to use something like [ngrok](https://ngrok.com/). 

## Run ngrok

ngrok is a cross-platform application that enables developers to expose a local development server to the Internet with minimal effort. We'll be using it to expose our service to the Internet. If this is your first time using ngrok, a [blog post](https://developer.vonage.com/en/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/) explains it in more detail. Once you have ngrok setup and are logged in (again, the free account is acceptable), run the following command:

```text
ngrok http 9000
```

After ngrok runs, it will give you a Forwarding URL that we'll use as the base for our Webhooks later in the article. Mine looks like the following:

![Ngrok Running](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/ngrokrunning.png "ngrokrunning.png")

Now, if I wanted to access my local python server through AI Studio, I’d use the Forwarding address shown in the screenshot. 

## AI Studio

Navigate to the [Vonage AI Studio](https://studio.docs.ai.vonage.com/#_ga=2.52857896.1004057560.1652645262-2049185403.1651612958) home page and press the button to Create Agent. You will see an option of what type of Agent you would like to create.

![Agent Creation](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/agent-creation.png "agent-creation.png")

To begin, select the Telephony Agent and press **Next,** as we want to create a voice-call scenario that you can use on your mobile phone.

We will need to fill in some details here:

* **Region**: Where will your Agent be typically used - The USA or in Europe
* **Agent Name**: Give your Agent a unique name that is meaningful to you. 
* **Language**: Select the language of your Agent.
* **Voices**: Select any voice (I’m using Matthew).
* **Time Zone**: Choose the time zone where your Agent will operate.

Next is the option to choose a template, and you can view some of the [other templates available](https://studio.docs.ai.vonage.com/ai-studio/templates) for different Agent types. In this case, we will select **Start from Scratch** and press **Next**.

Finally, we have the option to **Select Event**. Events trigger your Agent, whether initiated by a user or the Agent itself. We will use an **Inbound call** and press **Create** for this example.

Next, you will see the main user interface of AI Studio! If you’d like a primer on what it is capable of, visit here. 

To keep things simple, we’ll use the following conversation flow: 

![Simple User Interface](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/simpledesignlayout.png "simpledesignlayout.png")

We added: 

* 2 Parameters to store the value that the user enters and the response that OpenAI responds with. 

  * Custom Parameters
  * Name: AGENT_RES and Entity is @sys.any 
  * Name: INPUT and Entity is @sys.any 
* Collect Input Node - That welcomes the user and collects their input. 

  * Parameter: Use the INPUT Parameter that we created in the previous step. 
  * Prompt Text: Welcome to the Vonage FAQ! Please enter your question, and we'll try our best to answer! 

    * When we run the Agent, we’ll use language anyone might say in a normal conversation, such as “**Where is the Vonage Headquarters?**" or **“What communication APIs does Vonage support?”**.
* Webhook Node - 

  * Request URL

    * Request Type: POST
    * URL - Should look like the following but substitute my ngrok URL with your own <https://db20-2601-600-9580-d650-cfa-951f-cdef-d1d5.ngrok.io/webhook> 
  * Headers:

    * HTTP Header - INPUT and Value is $INPUT
  * Response Mapping:

    * Object Path - res
    * Value is $AGENT_RES
* Speak Node - To output the results from OpenAI.

  * Text - Your answer is: $AGENT_RES
* End Call Node - To end the conversation once the conversation flow has completed. 

Let's run the application by pressing the **Tester** button at the top right-hand corner of the screen and see what type of output we get. To begin, we see the welcome message along with the first question. The user might input, "**Where is the Vonage Headquarters?**" The Virtual Agent responds with a message that came back from OpenAI, which says the following: 

![Sample Question #1](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/chatgpt-example1.png "chatgpt-example1.png")

Or maybe you wish to ask it a more open-ended question, such as “What communication APIs does Vonage offer? And you’ll get the following reply:

![Sample Question #2](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/chatgpt-example2.png "chatgpt-example2.png")

GPT API also supports multi-intent handling, which occurs when you have an input with more than one intent, and the system handles both. A great example of this would be asking, “Where is the Vonage headquarters, and when was the company founded?” You might be surprised to see the result! 

![Sample Question #3](/content/blog/build-a-faq-answering-service-with-openai-and-vonage-ai-studio/chatgpt-example3.png "chatgpt-example3.png")

## Wrap-up

We saw how you could implement OpenAI’s GPT API in Vonage’s AI Studio in less time that it takes to grab lunch! By automating a FAQ (in this example), you could save hours a typical employee might spend to collect the data and create a response. You also might want to try to publish the Agent and test it with your cellphone! 

Also, note that the current integration via webhooks is a short-term solution, and we plan to develop dedicated nodes to support LLM by the end of Q2 2023. 

So what are you waiting for? Give the GPT API and [Vonage AI Studio](https://studio.docs.ai.vonage.com/#_ga=2.52857896.1004057560.1652645262-2049185403.1651612958) a shot today! Also, if you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/en/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!