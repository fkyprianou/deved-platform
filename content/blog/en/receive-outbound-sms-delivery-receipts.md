---
title: Receive Outbound SMS Delivery Receipts
description: Learn how to ensure that your SMS messages are being delivered with
  Messages API and the message status feature. This tutorial shows how to build
  a Python Flask app to receive delivery receipts.
author: benjamin-aronov
published: true
published_at: 2023-06-19T10:36:55.460Z
updated_at: 2023-06-19T10:36:55.479Z
category: tutorial
tags:
  - messages-api
  - python
  - sms
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

SMS has many benefits for your users to communicate: reliability, no need for an internet connection, no need to download an application, and many more. But one downside is that you don't know if a sent SMS message was delivered or not if you don't have access to the destination phone number.

However, Vonage allows you to receive the confirmation of delivery via webhooks. In this tutorial, you will learn how to set up webhooks to receive message delivery receipts and how to configure your account to use webhooks. 
There are two APIs for SMS applications in Vonage. The [SMS API](https://developer.vonage.com/en/messaging/sms/overview) can be used for SMS messaging while the [Messages API](https://developer.vonage.com/en/messages/overview) allows you to use both SMS and non-SMS channels like WhatsApp, Facebook Messenger, Viber, and MMS. This tutorial uses the Messages API.

In this tutorial, you will see how to receive delivery receipts via a publicly accessible [webhook](https://developer.vonage.com/en/getting-started/concepts/webhooks) upon sending SMS messages.

## Steps in the Tutorial

1. Setup and Installations
2. Create the Webhook for Delivery Receipts
3. Create Publicly-Accessible Webhook URL
4. Configure Your Vonage Account
5. Try It Out!

## Setup and Installations

1. [Python](https://www.python.org/downloads/?)
Flask is a web framework that requires Python to run. Get the appropriate version for your OS on the [Python downloads page](https://www.python.org/downloads/?).

2. [Flask](https://flask.palletsprojects.com/en/2.3.x/installation/)
Before installing Flask, create your project directory and navigate into it as follows:
```python
mkdir delivery_receipt
cd delivery_receipt
```

Install Flask with the `pip` package manager. `pip` comes installed with Python. You will then use Flask to power your application. Use the following terminal command to install Flask inside the newly created folder.

```python
pip install flask
```

3. You can use the [Vonage CLI](https://github.com/Vonage/vonage-cli) to purchase a Vonage virtual number. Alternatively, you may buy a number from the Vonage [dashboard](https://dashboard.nexmo.com/buy-numbers) in your browser. You can also use the CLI to send a test SMS.
Install the Vonage CLI globally in your terminal with the `npm` package manager:

```bash
npm install -g @vonage/cli
```

If you have successfully installed the above packages, get onto the next section, creating the webhook.

## Create the Webhook for Delivery Receipts
When you send messages out through your (Vonage application)[https://developer.vonage.com/en/getting-started/concepts/glossary#vonage-application], the delivery receipt is forwarded to your Python application if you have set a webhook URL in your Vonage dashboard. We’ll setup the Vonage part later on, but right now we’ll configure our Flask application to accept this POST request.

You will accept a `POST` request in your webhook handler to receive message status updates. Messages API webhook requests are `POST` requests. You will create a Flask endpoint for the request.

Create a `receipt.py` file in your Flask project and add the following code:

```python
#!/usr/bin/env python3
from flask import Flask, request, jsonify
from pprint import pprint

app = Flask(__name__)

@app.route("/webhooks/message-status", methods=['POST'])
def message_status():
  if request.is_json:
    data = request.get_json()
    pprint(data)
  else:
      data = dict(request.form) or dict(request.args)
      pprint(data)

  return "200"

if __name__ == '__main__':
  app.run(host="", port=3000)
```

In the code above, you created the webhook endpoint, `/webhooks/message-status`, which accepts the status code of SMS via webhooks. The endpoint also parses the incoming request and then:

1. Uses the `request.get_json()` function to check if the request is in JSON format. JSON encoded requests have their `mimetype` set to `application/json` or `application/*+json`.
2. If the request is not in JSON format, then `request.form` and `request.args` retrieve the key/value pairs in the request body or URL query string respectively.
3. Parses the request data to the `data` variable and prints it to the terminal using `pprint`.
4. The last command runs the application’s server on port `3000`.

## Create A Publicly-Accessible Webhook URL
Now, you will create a publicly-accessible URL to which Vonage APIs can send the webhook requests. [ngrok](https://developer.vonage.com/en/getting-started/tools/ngrok) is a lightweight and fast engine that mimics a web server. Therefore, you won't have to do DNS settings, SSL configurations, and other things that you would need in a real production scenario. ngrok is a useful tool for speedy testing in a development environment. You can check out this [tutorial](https://developer.vonage.com/en/blog/local-development-nexmo-ngrok-tunnel-dr) to see more information on how to use `ngrok`.

First, install ngrok from the official [website](https://ngrok.com/download).

Next, launch ngrok on your terminal:

```bash
ngrok http 3000
```
You will get a public-facing URL similar to the following:

```bash
https://f9db-102-89-43-137.eu.ngrok.io -> http://localhost:3000
```

You can make web requests to the URL that `ngrok` generated for you. It is served on port `3000` of your local machine.

Note the following points:
* The URL changes whenever you restart the `ngrok` server if you are on the free plan.
* An ngrok server session lasts for 1 hour in the free plan after which you can launch the server again and you get a new public URL.

You need to append your endpoint to the newly generated ngrok URL to form your webhook URL like this:

* https://f9db-102-89-43-137.eu.ngrok.io/webhooks/message-status

The above URL is your full webhook URL.

## Configure Your Vonage Account
Visit the [settings](https://dashboard.nexmo.com/settings) page on your dashboard and go to the **SMS settings** section to select Messages API. You can maintain the other defaults such as the version of the Messages API at `1.0`.


Next, go to the [applications](https://dashboard.nexmo.com/applications) page to create a Vonage application.
Click the **+ Create application** button.

Supply your desired name in the **Name** field. 

In the Capabilities section, select **Messages** and input your webhook **Status URL**. You will need to enter your URL for both Inboud and Status URL. You can check out our [tutorial]() on how to set up the **Inbound URL** for accepting inbound SMS messages.

Your dashboard should like this with your ngrok urls:


Click **Generate new Application** to complete the creation process. This automatically leads you to the application page

On the application page, you can then link your number or buy a virtual number if you are using a paid Vonage account.



## Try It Out!
By the end of the above section, your ngrok server should be running in port `3000`, Ensure you have also configured your application with a virtual number in your Vonage account.

Now, you can start the Flask application in another terminal window to keep the previous terminal window running with ngrok.  Use the following command to launch the application in the specified port `3000`. ngrok gets the application from the port and serves it over the internet through its public URL.

```bash
python receipt.py
```

To test our code, we’ll need to send an SMS from our Vonage number through the Messages API. So first we’ll create a new file in our project called send-sms.sh

In that file we’ll add the following code:

curl -X POST https://api.nexmo.com/v1/messages \
  -H 'Authorization: Bearer '$JWT\
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d $'{
          "message_type": "text",
          "text": "Vonage Delivery Receipt Testing",
          "to": "'$TO_NUMBER'",
          "from": "'$FROM_NUMBER'",
          "channel": "sms"
}'

Replace $TO_NUMBER with your personal number, $FROM_NUMBER with your Vonage virtual number, and $JWT with a JWT you create for the application. You can use our online JWT generator tool.

– Ensure to add your country code to `YOUR_PERSONAL_NUMBER` such as 23400000001

Then open a new tab in your terminal and run:
` bash send-sms.sh`

You should see a response with the message_id, like this: 
{"message_uuid":"6cdbd8ed-7217-4bbf-ba3f-da9cdcaeaf2b"}

Then if you look in your ngrok tab you’ll see a successful POST request, like this: 
`POST /webhooks/message-status  200 OK                                                                                                                                                                                                          `

And lastly you’ll see in your application’s tab, you should see the status of the delivery receipt in the terminal like the following:


{'channel': 'sms',
 'destination': {'network_code': '42507'},
 'from': '972523083947',
 'message_uuid': '30af1182-e2c9-4986-bf60-0129a14bbc79',
 'sms': {'count_total': '1'},
 'status': 'submitted',
 'timestamp': '2023-06-19T10:23:16Z',
 'to': '972532208231',
 'usage': {'currency': 'EUR', 'price': '0.093'}}

Notice that the status is `submitted`. In a few seconds you should get a second response that looks like:


{'channel': 'sms',
 'destination': {'network_code': '42507'},
 'from': '972523083947',
 'message_uuid': '30af1182-e2c9-4986-bf60-0129a14bbc79',
 'status': 'delivered',
 'timestamp': '2023-06-19T10:23:23Z',
 'to': '972532208231'}




## Conclusion
Getting the true status of messages sent is crucial when building applications that involve sending out messages. It enables you to consider messages not delivered and enables the application to take appropriate steps when such situations occur.

This article shows how to get delivery receipts of SMS messages sent via Vonage's Messages API. You saw a Python-based application that implements the functionality and how it could be deployed and tested with a lightweight server, Ngrok. Therefore, you can go ahead and include the delivery receipts feature in your Python applications.

## Further Reading
You can go through some of our resources to learn more about the Messages API:

1. Messages API Overview https://developer.vonage.com/en/messages/overview

2. [Messages API Reference](https://developer.vonage.com/en/api/messages-olympus)

3. [Delivery Receipts Guide](https://developer.vonage.com/en/messaging/sms/guides/delivery-receipts)

3. [Delivery Receipts Restrictions](https://api.support.vonage.com/hc/en-us/articles/204014863-What-will-I-receive-if-a-network-country-does-not-support-Delivery-Receipts-?)

Did you enjoy this tutorial? Did you get stuck? Reach out on Twitter or the Vonage Community Slack. We’re excited to see what you’re building!
