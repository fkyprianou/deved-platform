---
title: How to Play an Audio Stream into a Phone Call with Python
description: How to Play an Audio Stream into a Phone Call with Python
author: oleksii-borysenko
published: false
published_at: 2023-02-28T14:02:29.730Z
updated_at: 2023-02-21T14:02:29.741Z
category: tutorial
tags:
  - puthon
  - flask
  - voice-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
There are various reasons you might want to play an audio stream into a call. An everyday use case is where developers want to put the caller on hold to help keep them relaxed, and you can play the music that gets their stress levels down.

Some example scenarios include:

* Playing caller on-hold music.
* Conference call - play music into a conference call until you have a quorum.
* Pre Recorded message - useful where Vonage doesn't support your language in its Text-to-Speech engine.
* Voice mail - when you call and leave a message, the voice message can be played back into a later call.

Below, you will see how to implement scenarios 1 and 2. The other scenarios will be covered in future blog posts.

There are also two methods for playing an audio stream into a call:

* Using a Call Control Object (NCCO)
* Using the Voice API (VAPI)

You will use method 1 for scenario 1, and method 2 for scenario 2.

Call Control Objects (NCCOs) provide a convenient way to control an inbound call.
NCCOs consist of some JSON configuration that describes how to handle the Call. There is a detailed [reference guide on NCCOs](https://developer.vonage.com/en/voice/voice-api/ncco-reference) where the many actions that can be carried out are described.
There is only one action we are interested in our scenarios, `stream`. There are actually two ways you can use the `stream` action: synchronous and asynchronous. Asynchronous means the caller can interrupt the audio `stream` using the phone keypad.
There are some stream action options that are of interest:

**Option**		**Description**

| Syntax      | Description                                                                                                                                                                                                                                                                                                                                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `streamUrl` | An array containing a single URL to an MP3 or WAV (16-bit) audio file to stream to the Call or Conversation.                                                                                                                                                                                                                                                                                          |
| `level`     | Set the audio level of the stream in the range `-1 >=level<=1` with a precision of 0.1. The default value is 0.                                                                                                                                                                                                                                                                                       |
| `bargeIn`   | If set to `true`, this action is terminated when the user presses a button on the keypad. Use this feature to enable users to choose an option without having to listen to the whole message in your Interactive Voice Response (IVR) system. If you set `bargeIn` to `true` on one more Stream actions then the next action in the NCCO stack must be an input action. The default value is `false`. |
| `loop`      | The Number of times audio is repeated before the Call is closed. The default value is 1. Set to 0 to loop infinitely.                                                                                                                                                                                                                                                                                 |

An example NCCO for playing audio into a call:

```python
[
  {
    "action": "stream",
    "streamUrl": ["https://acme.com/music/relaxing_music.mp3"]
  }
]
```

The audio file formats supported are MP3 and 16-bit WAV.

We've already developed a starter Vonage application to receive a call and play music into your Call using two scenarios. With the starter application, you need to add your credentials to the .env file and deploy the app using Github Codespaces. Also, you can edit the starter application and experiment.
To get started:
Fork [this repository](https://github.com/obvonage/tutorial-audio-stream--python-sdk). Open it in Codespaces by clicking "Create codespace on main"

![](codespaces.png)

Alternatively, you can deploy the app with [Python](https://www.python.org/downloads/) and [ngrok](https://ngrok.com/).

## Create a new Vonage app

Sign in/Sign up for free [developer.vonage.com](https://developer.vonage.com/); to be able to use the Vonage Voice API, you'll have to create a Vonage Application from the developer portal.
All requests to the Vonage Voice API require authentication. We will use the API key and private.key
In the left menu [here](https://dashboard.nexmo.com/), click API Settings. Under the API keys tab you will find your API key and click button `Generate public and private key`. Paste key in `private.key` file in related Codespace.

![](settings.png)

This tutorial also uses a virtual phone number. 

We need to buy a virtual number for our app to accept phone calls:

* Search and buy virtual phone numbers using [Vonage Dashboard](https://dashboard.nexmo.com/buy-numbers).
* Select `Voice` feature from the dropdown menu.
  Link numbers using Vonage Dashboard, go to Applications, open related App (e.g. VoiceApp), and click the 'Link' button in the list of numbers.

![Link number with the application](/content/blog/how-to-play-an-audio-stream-into-a-phone-call-with-python/link-number-with-app.png)

## Streamed audio into your Call using an NCCO

In this scenario, users will call a Nexmo number, and music will be streamed into users' calls using an NCCO with a `stream` action. 
We will use the following `scenario-1.py` file:

```python
from dotenv import load_dotenv
from flask import Flask, request, jsonify

from os import environ as env

# Load environment variables from a .env file:
load_dotenv('.env')

# Load in configuration from environment variables:
VONAGE_APPLICATION_ID = env['VONAGE_APPLICATION_ID']
CONF_NAME = env['CONF_NAME']
STREAM_URL = env['STREAM_URL']

app = Flask(__name__)

ncco = [
   {
       "action": "stream",
       "streamUrl": [
           STREAM_URL
       ]
   }
]

@app.route("/webhooks/answer")
def answer_call():
   return jsonify(ncco)

@app.route("/webhooks/event", methods=['POST'])
def events():
   return ("200")

if __name__ == '__main__':
   app.run(host="localhost", port=9000)
```

**Try It Out**

You can run your code in Codespace, run the following commands:
Install the dependencies

```bash
pip install -r requirements.txt 
```

Start your app with the following command

```bash
python3 scenario-1.py
```

In the terminal, open the `Port` tab. Click on `Private` in the `Visibility` column, and change it to `Public`.

![Change port visibility](codespace-port-public.png)

The sequence of events in this scenario is as follows:

* Dial your Vonage Number.
* Vonage receives the Call.
* A callback is generated on the Answer webhook URL you specified.
* Your application receives the callback and responds with an NCCO.
* Music is played into your Call.

## Streamed audio into your Call using Voice API

In this scenario, you call your Vonage Number, and you are joined into a conference. You can then navigate to the /stream URL to initiate streaming into the conference. Music is then played into your conference using Voice API.

**Try It Out**

You can run your code in Codespace, run the following commands:
Install the dependencies

```bash
pip install -r requirements.txt 
```

Start your app with the following command

```bash
python3 scenario-2.py
```

In the terminal, open the `Port` tab. Click on `Private` in the `Visibility` column, and change it to `Public`.

![Change port visibility](/content/blog/how-to-play-an-audio-stream-into-a-phone-call-with-python/codespace-port-public.png)

The sequence of events in this scenario is as follows:

* Dial your Vonage Number.
* Vonage receives the Call.
* A callback is generated on the Answer webhook URL you specified.
* Your application receives the callback and responds with an NCCO.
* You are joined into a conference.
* Run the next command in the new terminal window `echo "https://${CODESPACE_NAME}-9000.preview.app.github.dev/stream"`
  Navigate to the link (in case of running on a local machine, `localhost:9000/stream`), and music will be played into your conference.

## Wrap-Up

Congrats! You can use the workflow and prepare code samples to stream related audio in call and related music when callers are on hold.

Show off your creations, or let us know how we can help! Join the conversation on our [Vonage Community Slack](https://developer.vonage.com/community/slack) or send us a message on [Twitter](https://twitter.com/VonageDev).