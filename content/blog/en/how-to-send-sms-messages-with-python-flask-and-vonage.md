---
title: How to Send SMS Messages With Python, Flask and Vonage
description: Quick Guide on How to Create a Flask App for Sending SMS Using
  Vonage Python SDK
thumbnail: /content/blog/how-to-send-sms-messages-with-python-flask-and-vonage/python-flash_sms.png
author: oleksii-borysenko
published: true
published_at: 2023-02-16T17:59:43.584Z
updated_at: 2023-02-16T17:56:56.257Z
category: tutorial
tags:
  - puthon
  - flask
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

How often do you receive an SMS? We live in a world where much communication goes through smartphones and applications. SMS uses a lot of collaboration between people for verification, marketing campaigns, and essential messages such as government agencies or reports about stormy weather. Compared to messages in messaging apps, you can be sure that SMS will be delivered to specific phone numbers, not only to applications associated with corresponding phone numbers. In addition, many online services use SMS for verification and 2-factor authentication.

In this tutorial, we will send an SMS using the terminal and a Flask application. I've prepared a small code sample which will be an excellent starting point for creating your application

## Vonage API Account

Sign in/Sign up for free [developer.vonage.com](https://developer.vonage.com/); to be able to use the [Vonage Messages API](https://developer.vonage.com/en/messages/overview), you'll have to create a [Vonage Application](https://developer.vonage.com/application/overview) from the developer portal.

Open [API Settings](https://dashboard.nexmo.com/settings). Under the API keys tab, you will find your API key and Account secret (API secret). We will use these credentials later.

![Vonage Application Settings](/content/blog/how-to-send-sms-messages-with-python-flask-and-vonage/settings.png)

**Set up Python**

First, make sure that your laptop/server has [Python 3](https://www.python.org/downloads/) installed. We will then use venv to create an isolated environment with only the necessary packages.

## Install the Vonage Python SDK

Install virtualenv via pip

```bash
pip install virtualenv
```

Create the virtual environment

```bash
python3 -m venv venv
```

Activate your virtual environment

```bash
source venv/bin/activate
```

Install the [Vonage Python SDK](https://github.com/Vonage/vonage-python-sdk)

```bash
pip install vonage
```

Run REPL, the Python language shell

```bash
python
```

Import the [Vonage Python SDK](https://github.com/Vonage/vonage-python-sdk)

```bash
import vonage
```

Create a Vonage Client object that can be re-used and knows your Vonage API key and its secret.

```bash
client = vonage.Client(key='YOUR-VONAGE-API-KEY', secret='YOUR-VONAGE-API-SECRET')
```

Use `send_message method` to send an SMS. You can replace "Vonage APIs" with related Sender ID (SMS sender name)

```bash
client.sms.send_message({"from": "Vonage APIs", "to": "**6**84**7*", "text": "A text message sent using the Vonage SMS API"})
```

Expected output

```bash
{'messages': [{'to': '**6**84**7*', 'message-id': '59a00000-1c17-40c9-40c9-5ebc****4196', 'status': '0', 'remaining-balance': '**.12347500', 'message-price': '0.03430000', 'network': '2**0*'}], 'message-count': '1'}
```

We receive a dictionary where we can parse the following information:
Was the message sent successfully? The "status': '0" means that all is good
How many messages were your SMS divided into
How much does it cost you to send the message? It depends on the number of symbols in SMS and receivers country

Check your phone notification, and you should receive an SMS message. If not, check the contents of the response.

![SMS screenshot](/content/blog/how-to-send-sms-messages-with-python-flask-and-vonage/sms-screenshot.png)

You can monitor and troubleshoot SMS delivery [on the Vonage API Dashboard](https://dashboard.nexmo.com/sms/delivery/by-day)

## Python Flask Application for Sending the SMS

Let's create a tiny web application using Flask to send an SMS. Our Flask app should contain a form for a phone number and an SMS message. When users press "Send SMS," it will post to a second view that will send the SMS using the Vonage SMS API.
Create a `server.py` file and paste related code there

```python
from dotenv import load_dotenv
from flask import Flask, flash, redirect, render_template, request, url_for
import vonage

from os import environ as env

# Load environment variables from a .env file:
load_dotenv('.env')

# Load in configuration from environment variables:
VONAGE_API_KEY = env['VONAGE_API_KEY']
VONAGE_API_SECRET = env['VONAGE_API_SECRET']
VONAGE_NUMBER = env['VONAGE_NUMBER']

# Create a new Vonage Client object:
client = vonage.Client(
   key=VONAGE_API_KEY, secret=VONAGE_API_SECRET
)

# Initialize Flask:
app = Flask(__name__)
app.config['SECRET_KEY'] = env['FLASK_SECRET_KEY']

@app.route('/')
def index():
   """ A view that renders the Send SMS form. """
   return render_template('index.html')

@app.route('/send_sms', methods=['POST'])
def send_sms():
   """ A POST endpoint that sends an SMS. """
   # Extract the form values:
   to_number = request.form['to_number']
   message = request.form['message']
   # Send the SMS message:
   result = client.sms.send_message({
       'from': VONAGE_NUMBER,
       'to': to_number,
       'text': message,
   })
   # Redirect the user back to the form:
   return redirect(url_for('index'))
```

Let's now create that template at `templates/index.html`
The following HTML includes the Bootstrap CSS framework and then renders a form with two fields: `to_number` for taking the destination phone number and `message`, so the user can enter their SMS message.

```html
<!doctype html>
<html lang="en">
   <head>
       <!-- Required meta tags -->
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  
       <!-- Bootstrap CSS -->
       <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.3.1/dist/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
  
       <title>Send an SMS</title>
     </head>
<body>
<script src="https://code.jquery.com/jquery-3.5.0.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/popper.js@1.14.7/dist/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@4.3.1/dist/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
<div class="container">
   <h1>Send an SMS</h1>
   <form action="/send_sms" method="POST">
       <div class="form-group"><label for="destination">Phone Number</label>
           <input id="to_number" class="form-control" name="to_number" type="tel" placeholder="Phone Number" /></div>
       <div class="form-group"><label for="message">Message</label>
           <textarea id="message" class="form-control" name="message" placeholder="Your message goes here"></textarea></div>
       <button type="submit" class="btn btn-primary">Send SMS</button>
   </form>
</div>
</body>
</html>
```

In this tutorial, we will use the python-dotenv library to load a `.env` file for me
Before starting your server, you'll need to provide a configuration in a `.env` file. Start with the following and fill in your details:

```bash
# Do not use this in production:
FLASK_DEBUG=true

# Replace the following with any random value you like:
FLASK_SECRET_KEY=RANDOM-STRING_CHANGE-THIS-Ea359

# Sender ID (or SMS sender name), VONAGE_BRAND_NAME
VONAGE_NUMBER=Vonage APIs

# Get the following from https://dashboard.nexmo.com/settings
VONAGE_API_KEY=paste-your-api-key
VONAGE_API_SECRET=paste-your-api-secret
```

Create `requirements.txt` and put related dependencies as in [this file](https://github.com/Vonage-Community/tutorial-sms-flask-python-sdk/blob/main/requirements.txt) 

Start your app with the following command

```bash
FLASK_APP=server.py flask run
```
Open [http://localhost:5000/](http://localhost:5000/) in your browser and navigate to the Usage chapter.

## Deploy from code source
Alternatively, you can deploy an application using a prepared code sample.
To deploy the application, you need to clone the repo using [git](https://git-scm.com/downloads) and update `.env` with your credentials


Clone the source code

```bash
git clone https://github.com/obvonage/tutorial-sms-flask-python-sdk
```

Go to the project folder

```bash
cd tutorial-sms-flask-python-sdk
```

Install the dependencies

```bash
pip install -r requirements.txt
```

Start your app with the following command

```bash
FLASK_APP=server.py flask run
```

Expected output

```bash
* Serving Flask app 'server.py'
* Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
* Running on http://127.0.0.1:5000
Press CTRL+C to quit
* Restarting with stat
* Debugger is active!
* Debugger PIN: 648-303-150
```

Next, open [http://localhost:5000/](http://localhost:5000/) in your browser and you should see the following web page


## Usage

![Flask App for Sending SMS Using Vonage Python SDK](/content/blog/how-to-send-sms-messages-with-python-flask-and-vonage/flask-app-for-sending-sms-using-vonage-python-sdk.png)

Let's send an SMS using this interface. Ensure the number is in international format without the '+' at the start. Hit "Send SMS" and check your phone! You can find the source code of this application [here](https://github.com/Vonage-Community/tutorial-sms-flask-python-sdk).

## Wrap-up

Congratulations! You've now built a Python Flask application to send an SMS. You can also [try out](https://developer.vonage.com/en/messages/overview) inbound and outbound interaction using Viber Business Messages, Facebook Messenger, and WhatsApp.

Let us know how we can help! Join the conversation on our [Vonage Community Slack](https://developer.vonage.com/community/slack) or send us a message on [Twitter](https://twitter.com/VonageDev).


