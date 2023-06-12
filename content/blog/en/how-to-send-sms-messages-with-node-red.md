---
title: How to Send SMS Messages with Node-RED
description: The Vonage API allows you to send and receive SMS worldwide. In
  this article, you will learn how to send SMS messages with Node-RED.
author: michael-crump
published: true
published_at: 2023-06-05T20:24:37.155Z
updated_at: 2023-06-05T20:24:37.206Z
category: tutorial
tags:
  - node
  - node-red
  - javascript
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

[Node-RED](https://nodered.org/) is a low-code programming tool designed to quickly connect hardware devices, APIs, and online services with a browser-based editor. The editor makes it easy to wire together flows using the wide range of nodes in the palette and can be deployed with a single click. In this article, we'll combine Node-RED with Vonage's Communications API to send and receive SMS worldwide using a simple HTTP-based API. 

## Prerequisites

Before we begin, ensure the following has been completed:

* [Node.js](https://nodejs.org/en/download/) installed. Node.js is an open-source, cross-platform JavaScript runtime environment. 
* [Node-RED](https://nodered.org/docs/getting-started/installation) installed. This will provide a low-code interface where we'll use the flow to send an SMS message.
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. 
* [Vonage Developer Account](https://developer.vonage.com/en/) - If you don't have one, you can create one, and we'll give you free credit to play around with our APIs.

## Setting Up Your Node-RED Editor

First, you’ll need to [install](https://nodered.org/docs/getting-started/local) Node-RED. For this tutorial, we'll do this on our local machine but it could be done on a Raspberry Pi or Debian-based operating system or a number of other options. 

In my case, I prefer to install Node-RED using npm. 

```bash
sudo npm install -g --unsafe-perm node-red
```

Note: If you are using Windows, do not start the command with `sudo`.

Once installed, you can use the `node-red` command to start it in your terminal. You can use Ctrl-C or close the terminal window to stop Node-RED.

If you are on MacOS and get the error `zsh: command not found: node-red`, then you can [view](https://stackoverflow.com/questions/12743928/command-not-found-after-npm-install-in-zsh) these suggestions or run it inside a Docker container. 

```bash
node-red
```

```text
Welcome to Node-RED
===================

31 May 13:16:14 - [info] Node-RED version: v3.0.2
31 May 13:16:14 - [info] Node.js version: v20.2.0
31 May 13:16:14 - [info] Windows_NT 10.0.22000 x64 LE
31 May 13:16:16 - [info] Loading palette nodes
31 May 13:16:17 - [info] Settings file: 
...
```

You can then access the Node-RED editor by pointing your browser at <http://localhost:1880>.

Once you have your editor open, you'll need to install the Vonage packages. Click on the **Hamburger Icon** on the right-hand side, then **Manage palette**, then search for the `node-red-contrib-nexmo` package and click **Install** as shown below. 

![Install the Vonage Node-RED package](/content/blog/how-to-send-sms-messages-with-node-red/installnode.png "Install the Vonage Node-RED package")

Now you should see all of the Vonage nodes appear on the left side of your screen, among the default nodes.

![Listing the Vonage nodes for Node-RED](/content/blog/how-to-send-sms-messages-with-node-red/listnodes.gif "Listing the Vonage nodes for Node-RED")

## Sending an SMS with Node-RED

Scroll down to the `Send SMS` node and drag it into your workspace. 

![Adding the Send SMS Node to the canva](/content/blog/how-to-send-sms-messages-with-node-red/sendsmsnode.png "Adding the Send SMS Node to the canva")

Double-click it and fill in the parameters below. 

| Parameter  | Description                                                                                                                                               |     |     |     |     |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- | --- |
| API KEY    | Your API key, shown in your [account overview](https://dashboard.nexmo.com/)                                                                              |     |     |     |     |
| API SECRET | Your API secret, shown in your [account overview](https://dashboard.nexmo.com/)                                                                           |     |     |     |     |
| TO         | The number you are sending the SMS to.                                                                                                                    |     |     |     |     |
| FROM       | The number or text shown on a handset when it displays your message. Use the [Number](https://dashboard.nexmo.com/your-numbers) assigned to your account. |     |     |     |     |
| TEXT       | The content of your message.                                                                                                                              |     |     |     |     |

![Adding the Properties to the SMS Node](/content/blog/how-to-send-sms-messages-with-node-red/propertiesofnode.png "Adding the Properties to the SMS Node")

Next, add an `inject` node to the flow and connect it to the `sendsms` node by connecting them together as shown below. 

![Adding the inject and sendsms Node together](/content/blog/how-to-send-sms-messages-with-node-red/injectnode.png "Adding the inject and sendsms Node together")

The `inject` node can be used to manually trigger a flow by clicking the node’s button within the editor. It can also be used to automatically trigger flows at regular intervals. 

We'll use it to set off our flow since we hard-coded the parameters in the `sendsms` node.

Note that the `TO`, `FROM` and `TEXT` parameters have a `{}` sign beside them, this means that [Mustache templating](https://mustache.github.io/) is supported for those fields. You could take advantage of this to pass information dynamically rather than hard coding it as we did earlier. A simple example would be setting the `TEXT` parameter to: 

```
The timestamp is {{msg.payload}}.
```

To have additional insight into what's happening when you send an SMS, drag a `debug` node onto the canvas and connect it to the `sendsms` node as shown below. 
Double click on `debug` and set the `Output` to `complete msg object` as shown below.

![Adding the Debug Node](/content/blog/how-to-send-sms-messages-with-node-red/debugnode.png "Adding the Debug Node")

Press **Deploy** if you haven't already, then place a check by clicking on the blue square on the left side in the node called `timestamp`, and your SMS should be on its way!

![SMS Deployed Successfully](/content/blog/how-to-send-sms-messages-with-node-red/deploynode.gif "SMS Deployed Successfully")

When you make a successful request, it returns an array of message objects. When looking at the `status` field, if you see a 0, it indicates that your message has been sent successfully. There are other details like recipient number, message-id, remaining balance, price, and carrier network that are also available. 

You can at this response object in the **Debug** area in the right side of your Node-RED editor.

![Debug Area](/content/blog/how-to-send-sms-messages-with-node-red/debug.png "Debug Area")

The output is helpful in troubleshooting or debugging an SMS that it is sent. 

When the message gets delivered, the mobile phone carrier returns a [Delivery Receipt](https://developer.vonage.com/en/messaging/sms/guides/delivery-receipts) to Vonage. This will contain the delivery status.

## Receiving a Delivery Receipt from a Mobile Carrier

To find out the `status` of your outbound message, you'll need to set up a webhook endpoint that Vonage can forward the **Delivery Receipt** to.

First, connect a `http` input node to a `http response` node and a `debug` node, so that you can view your delivery receipt in the debug area. Your screen should look like the following: 

![Delivery Receipt](/content/blog/how-to-send-sms-messages-with-node-red/deliveryr.png "Delivery Receipt")

In the `http` input node, select `POST` as a `Method` and fill in the `URL` field with `/receipt`.

![Post Method](/content/blog/how-to-send-sms-messages-with-node-red/nodepostmethod.png "Post Method")

## Exposing Your Local Server to the Internet

Next you'll have to expose your local server to the internet, so that Vonage can access it. A convenient way to do this is by using a tunneling service like [ngrok](https://ngrok.com).

[Download](https://ngrok.com/download) and install **ngrok**, then run it in the terminal to start a tunnel on port `1880`.

```shell
ngrok http 1880
```

![Starting an ngrok tunnel in a terminal window](/content/blog/how-to-send-sms-messages-with-node-red/ngrok.png "Starting an ngrok tunnel in a terminal window")

Your local server now has a ngrok URL that can be used as your webhook endpoint.

### Setting Up the Endpoint with Vonage

The last step is letting the Vonage API know where it should forward the delivery receipts. You can do so under your [API settings](https://dashboard.nexmo.com/settings) in the **SMS Setting** section.

Ensure the **Webhook format** is set to **POST** then  set the default webhook URL for delivery receipts to `YOUR_NGROK_URL/receipt`, then `Save changes`.

![Delivery receipt endpoint URL in dashboard](/content/blog/how-to-send-sms-messages-with-node-red/smssettings.png "Delivery receipt endpoint URL in dashboard")

Now when you go back into your Node-RED editor and send another message, you'll see the delivery receipt appear in the debug area:

![Delivery receipt in debug sidebar](/content/blog/how-to-send-sms-messages-with-node-red/delivery-receipt-in-debug.png "Delivery receipt in debug sidebar")

The `status` and `err-code` parameters both indicate that the message has successfully been delivered. Learn more about delivery receipt status messages and error codes in the [docs](https://developer.nexmo.com/messaging/sms/guides/delivery-receipts).

## Wrap-up

Now that you have learned how to get started with Node-RED and Vonage Communications API, you could extend this project by using additional flows through the [Node-RED Library](https://flows.nodered.org/). You also might want to try implementing this solution with several recipe's from the [Node-RED Cookbook](https://cookbook.nodered.org/). 

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!