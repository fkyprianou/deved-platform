---
title: Getting Started with Proactive Connect
description: Create outreach campaigns at a large scale using any Vonage
  Communication API such as Messages, Voice, and AI Studio.
thumbnail: /content/blog/getting-started-with-the-campaign-manager-api-and-ui/bulk-api.png
author: michael-crump
published: true
published_at: 2022-12-14T19:23:10.732Z
updated_at: 2022-12-14T19:23:10.825Z
category: tutorial
tags:
  - messages
  - bulk
  - campaign
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Today, we are pleased to announce a new tool called Proactive Connect that makes it easy to create outreach campaigns at a large scale using multiple Vonage Communications APIs, including SMS, Messages, Voice, and with AI Studio. This new API allows you to schedule and send personalized messages, across multiple channels, to different large-scale contact groups and offers the ability to receive and act on customer responses in a friendly Vonage Dashboard user interface. Proactive Connect enters Beta today, and you can try it now from your [Vonage Developer Portal](https://developer.vonage.com/). This blog explains how to set up campaigns through the user interface, where all the operations shown are also available programmatically through the API.

## Terminology

Once you log into your [Vonage Developer Portal](https://developer.vonage.com/), look under **Build & Manage**, and you will see **Proactive Connect**. Under **Configurations**, you'll see the following entries: Lists, Actions, Jobs, and Runs. Let's discuss what each of these means before moving forward. 

* **List**: A List contains all the targets to run the Job. They can be directly uploaded at the creation time or imported from an external resource, like a file or CRM.
* **Action**: An Action is typically used to send an SMS, WhatsApp Message, or make a phone call. Actions can also be used as reactions to trigger an API call upon receiving a response from the target user.
* **Job**: A Job iterates through the list and applies a predefined set of Actions to each element.
* **Runs**: The Runs show all of the Jobs that have run, are running, or are scheduled to run, and find out how your campaigns are currently performing.

## Building our Lists

We'll begin by creating a new List. Go ahead and click on **Lists** under **Configuration**, and you'll see the following screen:

![Lists](/content/blog/getting-started-with-proactive-connect/lists.png "lists.png")

Press **Create a new List**, and you'll be prompted to enter a **Name**, **Description**, and **Tags** for your list. We'll use **My Lists** for the **Name**, and for the **Description**, we'll use **This is my lists** and press **Next**.

> We could have added **Tags** to find this list, which is typically helpful when managing a large number of Lists.

It asks for a **Data Source**. You can select either **Manual** or **Salesforce**. If you select **Manual**, you'll need to provide a CSV file, or if you select **Salesforce**, you'll be asked to provide your Integration ID **and an option to provide the** Salesforce Object Query Language (SOQL)\*\*. For this demo, we'll use a CSV File that looks like this:

```text
firstName,lastName,Number,Location
Michael,Crump,14259999999,USA
Diana,Pham,14259999998,USA
Daniela,Facchinetti,14259999997,UK
Ben,Aronov,14259999996,Israel
```

Select **Data Source** and change it to **Manual**. Copy the text listed above and save it as a CSV file. Drag and drop the CSV file to the designer. Next, you'll assign a **Label name** with the label defined in the CSV File. For example, mine looks like the following:

![Map Labels to names defined in the CSV](/content/blog/getting-started-with-proactive-connect/csvlabels.png "csvlabels.png")

Finally, press the **Save** button. 

You now have a **Lists** which defines whom to send the campaign to, the number of entries, and when it was last modified. 

![Configured List](/content/blog/getting-started-with-proactive-connect/configuredlist.png "configuredlist.png")

## Creating a new Action

Now that we have a **List**, we need to perform an action to send messages to the group using one of our communication APIs. Again, this could be via SMS, MMS, Facebook Messenger, Viber, WhatsApp, or voice. Select the **Actions** under **Configurations** and press **Create a new Action**. You'll see several pre-configured Actions that you can use or start from scratch.

![Pre-configured actions](/content/blog/getting-started-with-proactive-connect/preconfigured-actions.png "preconfigured-actions.png")

This is where we can start seeing the power of **Proactive Connect**. Suppose we select the **SMS** pre-configured Action. In that case, it will automatically populate the fields, such as the **parameters** we'd like to use in the request, along with the **Command** used for the API call and **Response** settings that come back after a successful or unsuccessful call. 

The image below shows the command and headers to send an SMS message via the Vonage APIs. 

![SMS actions](/content/blog/getting-started-with-proactive-connect/smsaction.png "smsaction.png")

We will leave all the settings as default, so go ahead and press the **Save** button. 

## Creating a Job to run the Action

Select the **Jobs** under **Configurations** and press **Create a new Job**. A Job combines your Contact **List** and your **Actions** and allows you to apply segmentation and specify unique content to be sent to each. You can also use the Job to exclude specific groups from your campaign.

You'll be prompted to enter your list's **Name**, **Description**, and **Tags**. We'll use **My Job** for the **Name**, and for the **Description**, we'll use **This is my new Job**. We'll also need to **Include a List**, so I'll select **My Lists** from the drop-down and press **Next**.

We'll now be asked to **Create a Segment** to increase the success of your campaign by sending relevant content to your customers. The segments will be executed based on the order set here from left to right.

Let's create a segment that only sends a message to customers in the USA.

We'll provide the following information to this form:

* Name: US Customers
* Description: This message will go out to US-based customers.
* Condition: item.Location == "USA"
* Recipient: {{item.Number}}
* Template Message: Hello {{firstName}}, please visit vonage.com/usa to get your discount code. 

Note the information that we provided for the **Condition**. This matches the **Location** column from the CSV file to match only entries containing **USA**. In this case, there are only two entries. The **Recipient** field also uses the same data source and retrieves the **Number** provided in the CSV File. Finally, we provide a **Template Message** that uses the **FirstName** of the customer to send location-specific instructions. 

The only remaining thing to do is specify which **Action** we want to use for that **Segment**. I'll click on the drop-down and select **vg-send-sms**, which we defined earlier. By default, it will automatically populate the **Action parameters** as shown below. 

![configured actions](/content/blog/getting-started-with-proactive-connect/configure-actions.png "configure-actions.png")

Go ahead and press **Next**, and you'll reach the final step in creating a new job. Here you'll be allowed to define how you'll handle the responses from each of your chosen segments. This is optional, and for now, we'll press **Save** to continue. 

Keep in mind that if we wanted to create another **Segment** that targeted individuals living in the **UK**, we could press the **Create a Segment** button again and modify our **Condition** to match that location, along with a customized **Template Message** for that locale. We could also specify an **Action** that uses something besides the SMS API. For example, we could instead use Facebook Messenger, WhatsApp, etc.

## Scheduling a Run

The final step in creating a campaign is to create a **Run**. This is a scheduled job that will run at the interval specified. Select the **Runs** under **Configurations** and press **Schedule a Run**. 

You'll be prompted to enter your list's **Name**, **Description**, and **Job**. We'll use **My Run** for the **Name**, and for the **Description**, we'll use **This is my new Run**. We'll also need to select a **Job**, so I'll select **My New Job** from the dropdown.

Next, we need to **Schedule** when the **Run** will occur. Please note that the displayed start and end dates are in UTC. Once complete, press **Schedule** as shown below. 

![Create a Run](/content/blog/getting-started-with-proactive-connect/create-a-run.png "create-a-run.png")

If we now go to the **Runs** section and click on **Overview** and **Show More**, we can get detailed information about the run results.

![Run Results](/content/blog/getting-started-with-proactive-connect/run-results.png "run-results.png")

## Wrap-up

Now that you have successfully built your first campaign with sample data, you can begin incorporating your live production data. But before we go, check out the documentation for the product. If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack), and we will get back to you. Thanks again for reading, and I will catch you on the next one!