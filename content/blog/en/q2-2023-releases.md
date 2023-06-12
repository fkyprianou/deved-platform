---
title: Q2 2023 Releases
description: All Vonage releases from April to June 2023
author: diana-pham-1
published: true
published_at: 2023-06-12T21:35:45.175Z
updated_at: 2023-06-12T21:35:45.189Z
category: community
tags:
  - releases
  - video-api
  - verify-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
# Receiving Inbound SMS Notifications With Python (Using Messages API)

## Introduction
The Messages API allows you to receive inbound SMS messages.

This tutorial will guide you on how to receive SMS messages with your Vonage virtual number in a Python application. You can check out the [sample code](https://github.com/Nexmo/nexmo-python-code-snippets/blob/master/messages/inbound-message.py) on GitHub.

Vonage has two different APIs for receiving SMS: the SMS API used strictly for SMS messaging and the Messages API which allows you to use other channels like WhatsApp, etc. The way you configure your webhook is determined by which one you choose to use.. In this tutorial, we’ll be using the Messages API.

The inbound SMS messages will be received via a publicly accessible [webhook](https://developer.vonage.com/en/getting-started/concepts/webhooks). A later section in this tutorial covers the process of setting up a webhook and connecting it with your Vonage account.


# Steps

1. Setup and Installations

2. Create The Webhook For Inbound SMS

3. Make Webhook URL Publicly Accessible

4. Configure Vonage Accouunte


5. Try It out!



## Setup and Installations

1. Python

Python is needed in order to use the Flask library. Visit the official [Python downloads page](https://www.python.org/downloads/?) to download a Python version that works for you.

2. Flask

You will use [Flask](https://flask.palletsprojects.com/en/2.3.x/) to write your webhook. Before installing Flask, create your project directory and navigate into it as follows:
```python
mkdir receive_sms
cd receive_sms
```

You can install Flask with the pip package manager that comes installed with Python. Use the following terminal command to install Flask inside the newly created folder.

```python
pip install flask
```

3. Vonage CLI

The [Vonage CLI](https://github.com/Vonage/vonage-cli) package can be used to purchase a Vonage virtual number instead of purchasing on your dashboard UI. It can also be used to send a test SMS.

Install the Vonage CLI globally in your terminal by running this command:

```bash
npm install -g @vonage/cli
```
To purchase a phone number in a country, the United States for instance, use the following CLI commands::

```bash
vonage numbers:search US
vonage numbers:buy 15555555555 US
```
In the above commands, you can use the [Alpha-2 two-letter](https://www.iban.com/country-codes) code of the country you need to replace `US` which works for the United States.

Once you have successfully installed the above packages, you will then need to create the webhook.



## Create The Webhook For Inbound SMS
Upon receiving an SMS message on your Vonage account, Vonage verifies whether you have configured a webhook to which it can forward the message to your application.

You may configure the webhook to work for a single number or all the numbers in your account. To handle incoming messages, your webhook handler will be designed to accept a `POST` request. To facilitate this, you will create a Flask endpoint dedicated to handling the incoming requests.


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
    app.run(host="" port=3000)
```

In the previously provided code snippet, you defined a webhook endpoint named `/webhooks/inbound-message`. This endpoint is responsible for receiving and parsing the data contained within inbound message payloads.

- The `request.get_json()` method is utilized to verify whether the request is in JSON format.

- In case the request is not in JSON format, the key/value pairs within the request body or URL query string are retrieved using `request.form` and `request.args`, respectively.

- The request data is parsed and stored in the `data` variable, and then printed to the terminal using `pprint`.

Finally, you indicate that the application be run and served in port `3000`.




## Make Webhook URL Publicly Accessible
Your webhook endpoint will need to be accessible over the public internet, so Vonage APIs can request them. Usually in production mode, you would need a server with a public domain to append the endpoint. However, for testing in development mode, we will use `ngrok` to mimic the server environment. We have a [tutorial](https://developer.vonage.com/en/blog/local-development-nexmo-ngrok-tunnel-dr?) that explains in-depth how to use `ngrok` when testing Vonage APIs.

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

## Configure Vonage Account
Go to your dashboard [settings](https://dashboard.nexmo.com/settings) page. Then, go to the **SMS settings** section to select Messages API.

Next, go to the [applications](https://dashboard.nexmo.com/applications) page to create an application.
Click the **+ Create application** button.

Supply your desired name in the **Name** field. 

In the Capabilities section, select **Messages** and input your webhook **Inbound URL**. You can check out our [tutorial](LINKTOBLOGHERE) on how to set up the **Status URL** for accepting the delivery status of outbound SMS.

Click **Generate new Application** to complete the process. You will then be redirected to the newly-created application page. On this page, you can link your number or pay for a virtual number.

## Try It Out!
Your application is now ready! Your ngrok server is running in port `3000`, and you have configured your application with a virtual number in your Vonage account.

Now, you can start the Flask application in another terminal window to keep the previous terminal window running with ngrok.

```bash
python receive-sms.py
```

By executing that command, your Flask application will be launched on your local host port `3000`. This port will be automatically detected by ngrok, enabling you to serve your application to the public internet using ngrok's public URL.

Now, use the following Vonage CLI command to send a test SMS to your Vonage number on your terminal.

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
```



# Conclusion

Throughout this tutorial, you've discovered the process of receiving inbound messages using Python. We've walked through the necessary steps of configuring your webhook in Vonage, an essential part of this journey. You've also been introduced to ngrok, a useful tool that simplifies running local HTTP servers and testing your web applications. Now, equipped with the knowledge gained from this tutorial, you can readily integrate the Vonage Messages API into your Python-based applications and start receiving message notifications.


## Further Reading
Check out Vonage’s resources to learn more about the Messages API:

1. [Messages API Overview](https://developer.vonage.com/en/messages/overview)

2. [Messages API Reference](https://developer.vonage.com/en/api/messages-olympus)

3. [Concatenation and Encoding](https://developer.vonage.com/en/messaging/sms/guides/concatenation-and-encoding)


