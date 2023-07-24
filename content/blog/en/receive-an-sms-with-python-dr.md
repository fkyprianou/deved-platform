---
title: How to Receive SMS Messages in Python with the Vonage API
description: Find out how to receive inbound SMS messages in Python using the
  Vonage API. Learn more in this tutorial from Vonage Developer.
thumbnail: /content/blog/receive-an-sms-with-python-dr/Receive-an-SMS-with-Python.png
author: marklewin
published: true
published_at: 2019-05-31T09:57:29.000Z
updated_at: ""
category: tutorial
tags:
  - python
  - sms-api
comments: false
redirect: ""
canonical: ""
---
The Vonage Messages API allows you to receive inbound SMS messages in a range of different ways. This includes the option of receiving messages in Python, one of the most popular and flexible programming languages.

This tutorial will show you how to receive SMS messages in Python using the Vonage Messages API. You can find the [source code for this tutorial](https://github.com/Nexmo/nexmo-python-code-snippets/blob/master/sms/receive-flask.py) on GitHub.

---

*Note:*
Vonage has two different APIs for receiving SMS. The SMS API is used strictly for SMS messaging, Meanwhile, the Messages API allows you to use other channels like WhatsApp, etc. How you configure your webhook is determined by which API you choose. In this tutorial, we’ll be using the Messages API to send messages to our Python application.

---

## Prerequisites

To build a Python application that receives SMS messages, you will need a [Vonage API account](https://developer.vonage.com/sign-up). 

In addition to your Vonage API account, you will also need to install the following tools:

* [Python](https://www.python.org/downloads/) is needed to create your application and use the Flask library. 
* [Flask](https://flask.palletsprojects.com/en/2.3.x/) will be used to write your webhook for inbound SMS messages.
* The [Vonage CLI](https://developer.vonage.com/en/tools) package can be used to send a test SMS to Python. It can also be used to purchase a virtual number.

Below, we’ll walk you through the installation and setup process for each of these tools.

### Python

Visit the official [Python downloads page](https://www.python.org/downloads/) to download a Python version that works for you.

### Flask

Before installing Flask, create your project directory and navigate into it as follows:

```python
mkdir receive_sms
cd receive_sms
```

You can install Flask with the pip package manager that comes installed with Python. Use the following terminal command to install Flask inside the newly created folder.

```python
pip install flask
```

### Vonage CLI


Install the Vonage CLI globally in your terminal by running this command:

```bash
npm install -g @vonage/cli
```
To purchase a phone number in a country, the United States for instance, use the following CLI commands:

```bash
vonage numbers:search US
vonage numbers:buy 15555555555 US
```
In the above commands, you can use the [Alpha-2 two-letter](https://www.iban.com/country-codes) code of the country you need to replace `US` which works for the United States.

## Instructions

Once you have successfully installed Python, Flask, and the Vonage CLI, you are ready to get started. Below, we’ll explain how to use these tools so that Python can receive text messages from the Vonage API.

To create a Python application that receives SMS messages, simply follow these 4 steps:

1. Create a webhook for inbound SMS
2. Make the webhook URL publicly accessible
3. Configure your Vonage API account
4. Send and receive an SMS message

### 1. Create a Webhook for Inbound SMS

To receive an SMS in Python, the Vonage API requires a webhook for inbound SMS messages. 

Upon receiving an SMS message on your Vonage account, Vonage verifies whether you have configured a webhook to which it can forward the message to your application.

You may configure the webhook to work for a single number or all the numbers in your account. To handle incoming messages, your webhook handler will be designed to accept a `POST` request. To facilitate this, you will create a Flask endpoint dedicated to handling the incoming requests.

Here’s how to create the inbound SMS webhook for Python.

Create an `sms-receive.py` file in your Flask project and add the following code:

```python
#!/usr/bin/env python3
from flask import Flask, request, jsonify
from pprint import pprint

app = Flask(__name__)

@app.route("/webhooks/inbound-message", methods=['POST'])
def inbound_message():
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

In the previously provided code snippet, you defined a webhook endpoint named `/webhooks/inbound-message`. This endpoint is responsible for receiving and parsing the data contained within inbound message payloads.

* The `request.get_json()` method is utilized to verify whether the request is in JSON format.

* In case the request is not in JSON format, the key/value pairs within the request body or URL query string are retrieved using `request.form` and `request.args`, respectively.

* The request data is parsed and stored in the `data` variable, and then printed to the terminal using `pprint`.

Finally, you indicate that the application be run and served in port `3000`.

[H3] 2. Make the Webhook URL Publicly Accessible
Your webhook endpoint will need to be accessible over the public internet, so Vonage APIs can request them.

Usually in production mode, you would need a server with a public domain to append the endpoint. However, for testing in development mode, we will use `ngrok` to mimic the server environment. We have a [tutorial](https://developer.vonage.com/en/blog/local-development-nexmo-ngrok-tunnel-dr?) that explains in-depth how to use `ngrok` when testing Vonage APIs.

You can download or install ngrok from the official [website](https://ngrok.com/download).

Next, launch ngrok on your terminal:

```bash
ngrok http 3000
```

Once ngrok is launched, it will generate a public URL for you similar to the following:

```bash
https://f9db-102-89-43-137.eu.ngrok.io -> http://localhost:80
```

---

*Note:*
* The URL changes whenever you restart the `ngrok` server if you are on the free plan.
* An ngrok server session lasts for 1 hour in the free plan after which you can launch the server again and receive a new public URL.

---

You can then append your endpoint to the newly generated ngrok URLs to form your webhook URLs like this:

* https://f9db-102-89-43-137.eu.ngrok.io/webhooks/inbound-message

Now, you will need this URL when creating your application in the next section of this tutorial.

### 3. Configure Your Vonage API Account
Now that the Vonage API can request your webhook, it’s time to configure your Vonage account. 

At this point, you will create an application in the account and assign your webhook for incoming messages. This way, any incoming messages will be sent to your Python application.

To do this, go to your dashboard [settings](https://dashboard.nexmo.com/settings) page. Then, go to the **SMS settings** section to select Messages API.

Next, go to the [applications](https://dashboard.nexmo.com/applications) page to create an application. Click the **+ Create application** button. Then, enter your desired name in the **Name** field. 

In the Capabilities section, select **Messages** and input your webhook **Inbound URL**. You can check out our [tutorial](LINKTOBLOGHERE) on how to set up the **Status URL** for accepting the delivery status of outbound SMS.

Click **Generate new Application** to complete the process. You will then be redirected to the newly-created application page. On this page, you can link your number or pay for a virtual number.

### 4. Send and Receive an SMS Message
Your Python application is now ready to receive SMS messages! Your ngrok server is running in port `3000`, and you have configured your application with a virtual number in your Vonage account.

Now, you can start the Flask application in another terminal window (make sure to keep the previous terminal window running with ngrok). Then, in Flask, initiate this command:

```bash
python receive-sms.py
```

By executing that command, your Flask application will be launched on your local host port `3000`. This port will be automatically detected by ngrok, enabling you to serve your application to the public internet using ngrok's public URL.

Now, use the following Vonage CLI command to send a test SMS to your Vonage number on your terminal. If you’ve followed the steps in the tutorial correctly, this SMS message will be received in your Python application.

```bash
 vonage sms --to=VONAGE_VIRTUAL_NUMBER --from=VONAGETEST --message='Testing webhooks!'
```
In the terminal window that is running your Python application, you should see that your webhook received the inbound SMS:

```bash
{'keyword': ['THIS'],
 'message-timestamp': ['2023-04-28 11:30:22'],
 'messageId': [''],
 'msisdn': ['VONAGETEST'],
 'text': ['Testing webhooks!'],
 'to': ['447700900001'],
 'type': ['text']}
127.0.0.1 - - [28/April/2023 12:35:25] "GET /webhooks/inbound-sms?msisdn=VONAGETEST&to=2340000001&me

Congratulations! You just created a Python application that receives SMS messages!

```
## Conclusion

In this tutorial, you've learned the process for receiving inbound text messages in Python. You’ve learned how to configure a webhook for SMS messages, how to set up your Vonage account to send messages to Python, and how to test your Python application. 

Now you can easily integrate the Vonage Messages API into Python-based applications and start receiving SMS messages in Python!

## Further Reading
Check out Vonage’s resources to learn more about the Messages API:

1. [Messages API Overview](https://developer.vonage.com/en/messages/overview)

2. [Messages API Reference](https://developer.vonage.com/en/api/messages-olympus)

3. [Receive SMS Delivery Receipts in Python](https://developer.vonage.com/en/blog/receiving-sms-delivery-receipts-with-python-dr)