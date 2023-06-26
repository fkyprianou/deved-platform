---
title: Cloud Runtime Marketplace Open Beta Now Available
description: The Cloud Runtime Marketplace provides you with guides and tools to
  facilitate application build, deployment, and production.
author: michael-crump
published: true
published_at: 2023-06-16T18:17:24.535Z
updated_at: 2023-06-16T18:17:24.623Z
category: release
tags:
  - cloud-runtime
  - marketplace
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Today, we are pleased to announce the Beta availability of the Vonage [Cloud Runtime Marketplace](https://developer.vonage.com/cloud-runtime), Vonage Cloud Runtime is our cloud-native, serverless development platform that Vonage Solutions and Services teams use to quickly and effectively deploy API interactions on behalf of our customers. 

Now this platform is available directly to you, our developers, via our Cloud Runtime Marketplace, where we provide you with many easy-to-use guides and tools to facilitate application build, deployment, and production.  And since Vonage securely hosts the serverless platform on which you build, you are eliminating the need to maintain infrastructure to use Vonage APIs.

When you access the Vonage Cloud Runtime Marketplace, you can quickly browse communication solutions built with Vonage products and deploy them with a simple point-and-click process.

Key features of the Cloud Runtime Marketplace include:

* Availability of prebuilt solutions for common communication workflows, such as voice and messaging notifications, IVRs, and chatbots 
* Numerous code samples and a familiar, browser-based coding environment powered by Visual Studio Code
* A library of additional developer guides and tutorials to further facilitate your building of communications solutions 
* Offers persistence, distributed scheduling, queue support, and object storage.

Gone are the days of downloading a sample, configuring local dependencies, setting up a web tunnel, and more!  Accessing and building on Vonage Cloud Runtime via the Marketplace is now in open Beta, accessible to everyone.  During this Beta period - before General Availability - there will be no fee to access Vonage Cloud Runtime, so you can start building richer customer engagement applications now!  

And your feedback during this period will help us ensure we provide you with the tools to help you create winning experiences for your customers easily and effectively.

Now let's take a look at how you can quickly get started with Vonage Cloud Runtime:

## Getting Started

Point your browser to <URL> see the landing page. 

![CRM Landing Page](/content/blog/announcing-cloud-runtime-marketplace/vcc-landing-page.png "vcc-landing-page.png")

You can begin by browsing through the list of available samples for our products or using the search functionality of what you'd like to build. 

Once you click on the sample you'd like to use, you will see an **Overview**, **Get Code**, and **Deploy Code**. Let's use the "Interactive Voice Response (IVR)" sample for this tutorial.

![Navigate to the IVR Sample](/content/blog/announcing-cloud-runtime-marketplace/ivr-sample.png "ivr-sample.png")

You'll see three sections once you click on a sample: 

**Overview** - A short description of what the sample aims to achieve. 

**Get Code** - Allows you to use the Visual Studio Code browser version to try the code, make changes, and instantly deploy it. 

**Deploy Code** - Allows you to deploy the application and see it in action.

You'll also note that you can see when the sample was last created or updated on the upper right-hand side of the screen. If there are multiple versions, you can click on **Releases** to see them, and we have a handy **See documentation** link to learn more about the sample straight from the author. 

## Signing In

You must **Sign In** to your Vonage account to try a sample. If you don't have one, click on **Sign Up**. Once you have an account and have signed in, you'll notice a few additional options, as shown below. 

![CRM Signed In](/content/blog/announcing-cloud-runtime-marketplace/vcc-signed-in.png "vcc-signed-in.png")

Let's examine the options starting from the top left to right. 

**Workspaces** - Shows the current code workspaces that you have created.

**Deployed Products** -  Allows you to manage installed product instances.

**API Key** - Your assigned API Key that you'll be using with future deployments. If you have multiple API Keys, select from the drop-down you'd like to use.  

**Sign Out** - Signs out of the currently logged in user. 

## Let's Try a Sample

Ensure you are still on the **Interactive Voice Response (IVR)** sample and select **Deploy Code**. Give it an **Instance name** and select the **Region** closest to you and click **Deploy Code**. 

![Deploy option for IVR Sample](/content/blog/announcing-cloud-runtime-marketplace/ivr-deploy.png "ivr-deploy.png")

You'll notice a popup that denotes that this sample requires a number. 

![Set up your project deployment](/content/blog/announcing-cloud-runtime-marketplace/ivr-req-number.png "ivr-req-number.png")

Within the deployment option, you can **Assign a number** that you already have or **Buy a New One**.

![Add phone number](/content/blog/announcing-cloud-runtime-marketplace/ivr-buy-number.png "ivr-buy-number.png")

Once you've selected a number, it begins deploying. 

![Deploying App](/content/blog/announcing-cloud-runtime-marketplace/deployed-app.png "deployed-app.png")

If you look under **Deploy Code** now, you'll see the **Deployed Instances**. If you press the **Launch** button then your app will run in the web browser and provide you with a number to call to test. Go ahead and call the number and you should hear an Interactive Voice Menu. 

![Deployed Instances](/content/blog/announcing-cloud-runtime-marketplace/deployed-instances.png "deployed-instances.png")

You can press the **Launch** button to view the site or click on **View Logs** in order to troubleshoot any issues that you may have. 

![Deployment Logs](/content/blog/announcing-cloud-runtime-marketplace/ivr-logs.png "ivr-logs.png")

## Modify a Sample

Now that we have successfully deployed a sample let's try to change the code and instantly deploy it to see the changes live. Click on **Get Code** and then **Create a new development environment**. 

![Create a development environment](/content/blog/announcing-cloud-runtime-marketplace/ivr-get-code.png "ivr-get-code.png")

Provide a **Workspace Name** along with a **Vonage Number**. If you don't know what that is, then you can click **Assign a number** and available numbers in your Vonage account will be listed. 

![IVR Workspace](/content/blog/announcing-cloud-runtime-marketplace/ivr-workspace.png "ivr-workspace.png")

You'll now be presented with the project in Visual Studio Code inside your browser! How cool is that?

Begin by reviewing the **Readme.md** which contains important sample information. If you navigate to one of the source files (such as index.js), you can modify the code in-browser! 

![IVR sample running in VS Code](/content/blog/announcing-cloud-runtime-marketplace/ivr-vs-code-sample.png "ivr-vs-code-sample.png")

Go ahead and make some changes to the file. On Line 33, you could edit the text to say something different when they call. 

## Running the Sample

To start debugging the project, open the **Run and Debug** menu on the left side. Then start the debugger by clicking the **Play** button.

![Starting the debugger in the online workspace](/content/blog/announcing-cloud-runtime-marketplace/debug.png "debug.png")

In the bottom panel, open the **Ports** tab, then choose the link for port 3000.

![Opening the project link in the online workspace](/content/blog/announcing-cloud-runtime-marketplace/cc.png "cc.png")

This will open a new window and provide the number you linked to your application earlier. 

Call the number, and you should be greeted with **Hello. Please enter a digit.** (if you didn't change it earlier). After you enter a digit, you should hear the number being read back to you. 

If you would like to deploy your project, open the "Terminal" tab from the same menu you selected **Ports** and type the `neru deploy` command. 

![IVR Workspace](/content/blog/announcing-cloud-runtime-marketplace/ivr-neru-deploy.png "ivr-neru-deploy.png")

At the bottom of the output, you'll see an **Instance Host Address**. That is where the code has been deployed. You can use the URL to access your deployed code anywhere. 

## Wrap-up

We're excited to see all the new and interesting ways customers use this tool. Feel free to consult our [documentation](https://developer.vonage.com/en/documentation) for further information. Also, if you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!