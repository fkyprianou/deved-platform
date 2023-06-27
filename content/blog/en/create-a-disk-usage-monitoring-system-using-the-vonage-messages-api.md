---
title: Create a Disk Usage Monitoring System Using the Vonage Messages API
description: Learn how to build a custom disk usage monitoring system using
  Python and the Vonage Messages API
thumbnail: /content/blog/create-a-disk-usage-monitoring-system-using-the-vonage-messages-api/disk-usage-monitoring-system.png
author: adrian-francis
published: true
published_at: 2023-07-18T15:51:29.041Z
updated_at: 2023-07-18T15:51:29.061Z
category: tutorial
tags:
  - python
  - messages-api
comments: true
spotlight: true
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Disk Usage is an important metric to track in any organization's IT infrastructure. Without being aware of disk usage at any given moment, an organization risks performance issues in their production running applications as well as financial overlay due to sudden costs incurred in upgrading existing infrastructure. 

Modern cloud infrastructure providers, e.g., AWS provide advanced metric systems with alarms configured, but they do come at an extra cost. Disk usage is a key metric, and for a fraction of the cost by cloud providers, you can develop your own disk usage monitoring system complete with alerts. Cool, right?

In this guide, we will be going over exactly how to build a custom disk usage monitoring system using the Python programming language and the Vonage Messages API. Note that you will be able to run this program on your servers and even your local development machine.

## Prerequisites:

* [A Vonage developer account](https://dashboard.nexmo.com)
* Python 3.6+

### Set up a Vonage Application Via the Vonage CLI

1. Install the latest version of the Vonage CLI on your development machine:
   `npm install --location=global @vonage/cli`
2. [Get your Vonage API_KEY and API_SECRET by visiting the settings page on the dashboard](https://dashboard.nexmo.com/settings)
3. Configure your Vonage `API_KEY` and `API_SECRET` from the last step:
   `vonage config:set --apiKey=API_KEY --apiSecret=API_SECRET`
4. Create a vonage messages app:
   Use the command `vonage apps:create` and follow the prompts in the Interactive CLI mode as below:

```powershell
✔ Application Name … <insert any appropriate name>
✔ Select App Capabilities › <select Messages>
✔ Create messages webhooks? › (y/N)
✔ Allow use of data for AI training? Read data collection disclosure - https://help.nexmo.com/hc/en-us/articles/4401914566036 … <select either y/N>
```

## Our Goals

We want to build a lightweight application that does the following:

* Get our disk usage statistics (either on the server or local development machine). We will rely on a Python library, `psutil` to give us our disk usage statistics.
* Setup an arbitrary threshold e.g. 50% of disk usage, that allows us to send notifications when this threshold is hit.
* Customize a message that alerts us when our disk usage is above our threshold. We rely on the Vonage API to deliver this message to us.

## Steps

1. Create a project directory locally: For example, `mkdir disk_usage_monitor && cd disk_usage_monitor`.
2. Within our project directory, create a virtual environment to keep our project dependencies separate from the global Python dependencies:
   `python3 -m venv .venv`.
3. Activate our virtual environment using the following command:
   `source .venv/bin/activate`.
4. Inside our project directory, create a file `requirements.txt` using the command `touch requirements.txt` and copy the following project dependencies to the file:

```editorconfig
vonage==3.5.1
python-dotenv==1.0.0
psutil==5.9.5
```

5. To install the dependencies to our virtual environment, run the following command:
   `pip3 install -r requirements.txt`
6. Create .env file that will house our environment variables using the following command `touch .env` and copy the following content: 

**Note:**
Change `YOUR_VONAGE_API_KEY`, `YOUR_VONAGE_API_SECRET`, `THE_SENDER_NUMBER` and `THE_RECIPIENT_NUMBER` with the information found in the [Vonage Dashboard](https://dashboard.nexmo.com). 

```editorconfig
# Vonage Credentials
VONAGE_API_KEY=YOUR_VONAGE_API_KEY
VONAGE_API_SECRET=YOUR_VONAGE_API_SECRET
VONAGE_SENDER=THE_SENDER_NUMBER
RECIPIENT=THE_RECIPIENT_NUMBER
```

7. Add a file called `monitor.py` using the command `touch monitor.py` to our project directory and paste in the following content:

**Note:**

* `psutil.disk_usage` from the `psutil` python library, returns a named tuple of the form: `sdiskusage(total=xxxxx, used=xxxxx, free=xxxx, percent=47.0)`.
  The percent value will be of most interest to us.

```python
import psutil
import vonage
from dotenv import dotenv_values

# load your environment variables
config = dotenv_values(".env")

client = vonage.Client(key=config["VONAGE_API_KEY"],  secret=config["VONAGE_API_SECRET"])

# get disk usage statistics for home directory
stats = psutil.disk_usage("/")


# device: give it an easily identifiable name e.g. server 1.89.200.4
device = "local dev machine"

# send out a message indicating the disk usage level
client.messages.send_message(
	{
    	"channel": "sms",
    	"message_type": "text",
    	"from": config["VONAGE_SENDER"],
    	"to": config["RECIPIENT"],
    	"text": f"Your Disk Usage is at {stats[-1]} % on {device}.",
	}
)
```

With that, you can run the `monitor.py` file using the following command: `python3 monitor.py`.

8. We would like to get notifications once a certain threshold is hit on our device. We can modify our `monitor.py` file to send us alerts once a given threshold is hit:

```python
# our previous code remains as is on this section
...
# the disk usage threshold. Adjust this to an acceptable threshold of your choice
threshold = 50

# if disk usage is above given threshold, send out an sms alert
if stats[-1] > threshold:
	client.messages.send_message(
    	{
        	"channel": "sms",
        	"message_type": "text",
        	"from": config["VONAGE_SENDER"],
        	"to": config["RECIPIENT"],
        	"text": f"Your Disk Usage is at {stats[-1]} % on {device}. Delete some files soon to create some disk space!",
    	}
	)
```

## Bonus

On Unix systems, we can set this program up as a cron job that runs daily (or depending on your preferred frequency) to perform regular disk usage checks:

Edit your crontab entry in a terminal session using the following command `sudo crontab -e` and add the following cron entry to run daily at 6:00 AM:

```powershell
0 6 * * * cd <project_directory> && .venv/bin/python3 monitor.py
```

On Windows, we can set up our program as a scheduled task to run daily using the following steps:

1. Open the task scheduler from the start up menu.
2. From the actions panel, select the create task option to create a new cron job.
3. On the create task window that appears on your screen, add a name for the task under the general tab and leave the other options as default.
4. Select the triggers tab and click on new to add a trigger.
5. On the begin task window, select daily from the side panel, and the start time as 06:00:00 AM for example. Leave the other options as default.
6. Create a new action from the actions panel and on the New Action panel, provide a path to the python executable, which is the result of running the following command in a command prompt: `where python`.
7. In the add arguments section, add the file name e.g `monitor.py`.
8. In the start in section, add the path to the project directory: `C:\Users\<user>\path\to\project-directory`.

## Conclusion

We have set up a fully-fledged disk usage monitoring system with an alert component that relies on the Vonage APIs to send out SMS alerts. 

If you enjoyed this or you have any further questions, you can shoot me a message on [Email](mailto:adriannduva@gmail.com) or view the full code sample on [GitHub](https://github.com/Vonage-Community/blog-messages_api-python-disk_usage_monitoring).

Vonage always welcomes community involvement. Please feel free to join Vonage on [GitHub](https://github.com/Vonage-community) and the [Vonage Community Slack](https://developer.vonage.com/community/slack).