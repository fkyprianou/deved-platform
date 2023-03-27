---
title: Using Vonage APIs with MongoDB Atlas - Part 1
description: MongoDB Atlas and its associated products are a great complement to Vonage APIs. What is MongoDB Atlas and what does it bring to the table?
thumbnail: 
author: christankersley
published: true
published_at: 2023-03-15T16:11:12.953Z
updated_at: 2023-03-15T16:11:12.974Z
category: tutorial
tags:
 - mongodb
 - node
 - verify-api
 - messages-api
 - meetings-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---

Developing applications is hard. Not only are there the base requirements for the application itself, but there are always common problems to solve, like how to authenticate users, how the database is managed, and where is all of this hosted. In 2023 we are almost spoiled with the number of services that can help solve these problems, but all of this needs to be brought together inside an application. One solution that has come along is MongoDB Atlas, a suite of products that are designed to help developers build their applications quickly and handle many common problems.

## What is MongoDB Atlas?

MongoDB Atlas is a hosted cloud database service that can be used by multiple cloud hosting providers. This means that not only can you host your database in multiple regions, but you can also host your database across AWS, Azure, and Google Cloud Platform for multi-cloud availability. Since the databases are hosted in the cloud, scaling up and down can be done on demand. For developers, it frees up a lot of administrative time handling more servers by allowing MongoDB to handle all the infrastructure while developers can focus on their applications.

MongoDB Atlas allows you to set up MongoDB clusters. If you are not familiar with MongoDB, it is a [document-based](https://en.wikipedia.org/wiki/Document-oriented_database) [NoSQL Database](https://en.wikipedia.org/wiki/NoSQL) system. Unlike a traditional Relational Database Management System (RDMS), MongoDB stores information as JSON-like documents that can be searched. It has limited relational capabilities and focuses more on lightly-structured data rather than tabular row/column architectures. Documents are grouped together as "collections," which replace the standard table architecture. Documents in a collection can share a common schema, like an RDMS table, but can also change their structure.

```json Sample MongoDB Document
{
 "_id": ObjectId("6413733ba623c618c2fab2d9"),
 "name": "Hamburger",
 "price": 995
}
```

NoSQL databases, as the name suggests, do not use SQL to query for information. MongoDB uses a JSON-like query syntax to search for documents that match the criteria. For example, instead of using something like `SELECT * FROM users WHERE admin = true`, you would use the following syntax:

```
db.users.find({
 admin: {
 $eq: true
 }
})
```

Many developers prefer using a NoSQL database for the freedom that the schemaless document design provides. There are no major migrations as new "columns" can be added to documents by just adding them to new or existing documents. You can define a schema if you want, but it largely only helps the database engine filter through data in larger data sets. 

MongoDB Atlas itself also brings a few additional features that developers can use to build their applications on top of the robust NoSQL database that MongoDB brings. This is part of their "App Service" layer that adds user authentication, a serverless function runtime, an associated API gateway and router, automatic GraphQL and HTTPS data access, and a device data syncing service called MongoDB Realm.

What all of this means is that a developer can start developing their application right away without having to piece together a bunch of disparate services and can focus on the business problems that the application solves, not shave the proverbial yak on how to do user authentication or how to deploy code. Atlas and its App Services can do a lot of that heavy lifting for a developer.

## What do we plan to do?

In the next series of articles, we are going to walk through building an application that utilizes MongoDB Atlas and a suite of Vonage communications APIs. The demo application will take the form of a simple restaurant website and an associated backend. We will show:

* How to integrate Vonage Verify with a user login
* How to use Vonage Messages to send an order confirmation
* How to use Vonage Meetings for issue resolution
* How to use In-App messaging to pass notifications back to "admin" users

We will also help developers set up:

* A MongoDB Atlas cluster and associated App Service
* Having a front-end app talk back to MongoDB Atlas via Realm
* User Authentication with MongoDB Users

While we will be breaking down how all of this works over time, feel free to take a quick glance at the source code for the application at [the source code on GitHub](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo).

## Prerequisites

* [MongoDB Account](https://www.mongodb.com/cloud/atlas/register)
* [Vonage Developer Account](https://developer.vonage.com/sign-up)
* [Realm CLI](https://www.mongodb.com/docs/atlas/app-services/cli/) - A command line application that makes it easier to manage App Services in MongoDB Realm
* Node.js 16+ - Node.js is an open-source, cross-platform JavaScript runtime environment.

## Set up a Vonage Application

Our Messages, Verify, and In-App Messaging APIs are all backed by a Vonage Application, which is a set of configuration data that can be grouped together. Once you have signed into your developer account, go to the Applications page and [create a new application](https://dashboard.nexmo.com/applications/new). Give your application a name like **MongoDB Demo**, and then click **Generate Public and Private key**. This will create the authentication keys we will use in the SDKs. 

![Creating a Vonage application](/content/blog/using-vonage-with-mongodb-atlas-part-1/0001-new-app-name.png "Name and Secret Keys")

Now scroll down and we can turn a few different capabilities. We will need **Messages**, **RTC (In-app voice & messaging)**, and the **Meetings API**. Toggle each of those capabilities on. **Messages** and **RTC** needs a few callback URLs that we will not utilize for the moment, so just enter `https://example.com` for those handfuls of URLs that are required. **Meetings** can stay blank. Once that's all done, click on "Generate new application".

![Messages API Capability](/content/blog/using-vonage-with-mongodb-atlas-part-1/0002-messages-api.png "Messages API Capability")

![RTC API Capability](/content/blog/using-vonage-with-mongodb-atlas-part-1/0003-rtc-api.png "RTC API Capability")

![Meetings API Capability](/content/blog/using-vonage-with-mongodb-atlas-part-1/0004-meetings-api.png "Meetings API Capability")

Since we are using the **Messages API**, we will need to link a telephone number to our application. This will be used for outbound SMS later on. Developer Accounts should have a number already where available, so just click the "Link" button to attach it to this application.

## Set up MongoDB Atlas

Now that we have the Vonage side set up, let's set up the database in MongoDB Atlas. When you first log into your account, it will prompt you to deploy your database. As this is a hosted plan, we will need to set up some hosting information. Thankfully the MongoDB Atlas system has a very generous free tier. Just select the **M0** free tier to host our database. This is more than powerful enough for us to play around for our demo. The only other thing you will need to do is add a **Name** for the database cluster. For the purposes of this demo, I have just named it _VonageDemo_. If you want you can change the hosting provider or Region, but for now you can leave them at the default of "AWS" and "N. Virgina (us-east-1)". Click **Create** to move on.

![Database Cluster Settings](/content/blog/using-vonage-with-mongodb-atlas-part-1/0005-deploy-your-database.png "Database Cluster Settings")

As we will be accessing the MongoDB cluster over the internet, we will need to set up some authentication. We can use **Username and Password** auth for our demo as it is the easiest to get up and running with. It will pre-fill a username and password for you, feel free to change these. Just make sure you note down the password for later, as we will need that to authenticate to MongoDB. When you are done, click **Create User**.

![MongoDB Cluster Authentication](/content/blog/using-vonage-with-mongodb-atlas-part-1/0006-authentication.png "MongoDB Cluster Authentication")

For security reasons, MongoDB Atlas restricts who can talk to your cluster. For our demo, you can just select **My Local Environment**. The server component of the demo will connect directly to the cluster so we will need to allow it access to the cluster. By default, it adds your public IP address to the list. This is fine for running the demo locally, but if you are going to deploy this to a public server you will need to add that server's IP address. If you are hosting the server on another machine, please check with your hosting provider to find out your public IP address. If you are going to host the demo in a cloud provider like AWS or Google Cloud Platform, you can select **Cloud Environment** and provide the appropriate details. Click **Finish and Close** to finish up.

![MongoDB Security Settings](/content/blog/using-vonage-with-mongodb-atlas-part-1/0007-ip-access-list.png "MongoDB Security Settings")

Your MongoDB Atlas cluster is now all set up! You can administer the cluster through the browser, including viewing the stored documents. The dashboard also has instructions for connecting through their VSCode plugin if you want to access the database directly in your IDE. 

![MongoDB Dashboard](/content/blog/using-vonage-with-mongodb-atlas-part-1/0008-mongodb-finished.png "MongoDB Dashboard")

## Next Steps

In the next part, we will add some sample data to our MongoDB cluster, and walk through setting up the demo application to run. In the meantime, feel free to poke around the MongoDB Atlas dashboard as well as the Vonage Dashboard to see all the different services both companies offer.