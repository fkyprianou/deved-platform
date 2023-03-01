---
title: Traffic Alert Through SMS Using Vonage and Google Maps API
description: Learn how you can create a traffic alert through SMS using Vonage
  and Google Maps API in Node.js.
thumbnail: /content/blog/traffic-alert-through-sms-using-vonage-and-google-maps-api/traffic-alert.png
author: pranav-shinde
published: true
published_at: 2023-03-02T10:10:54.060Z
updated_at: 2023-03-02T10:10:54.071Z
category: tutorial
tags:
  - javascript
  - node
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Overview

In this hustling and bustling world where life’s moving pieces are constantly going, users value retrieving information as conveniently as possible. This is not always easy, especially in countries like India, where the internet network coverage in remote areas is weaker.

Let’s say for example that you are about to leave the office to go home after a busy day and want to check if there is traffic on your route, but your internet connection is slow. What if you could send an SMS with your start and end locations and instantly get the traffic information? That would be a lifesaver!

In this article, I will show how you can create a traffic alert through SMS using Vonage and Google Maps API in Node.js.

## Prerequisites

Before you begin, ensure you have installed the following: 

* [Node.js](https://nodejs.org/en/download/). Node.js is an open-source, cross-platform JavaScript runtime environment.
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet.
* [Vonage CLI](https://www.npmjs.com/package/@vonage/cli) - Once Node.js is installed, you can use `npm install -g @vonage/cli` to install it. This tool allows you to create and manage your Vonage applications.

<sign-up></sign-up>

Once you have created an account, you can find your API Key and API Secret at the top of the [Vonage API Dashboard](https://dashboard.nexmo.com/settings).

![Vonage API dashboard](/content/blog/traffic-alert-through-sms-using-vonage-and-google-maps-api/settings-app.png)

Once you have received the **API Key** and **API Secret**, you can use it for the [SMS API](https://dashboard.nexmo.com/getting-started/sms).

### Google Maps API

> We will also require [Google Direction API](https://developers.google.com/maps/documentation/directions/overview). Create a Google account and sign in to access free credits to use the Google API and the [API Key](https://cloud.google.com/api-keys/docs/get-started-api-keys).

![GCP account and API Key](/content/blog/traffic-alert-through-sms-using-vonage-and-google-maps-api/gcp-account.png)

To get the Google Maps API key:

* Go to the **Google Maps Platform** > [Credentials page](https://console.cloud.google.com/project/_/google/maps-apis/credentials).
* On the Credentials page, click **Create credentials > API key**.
  The **API key** created dialog displays your newly created **API key**.
* Click Close. The new **API key** is listed on the Credentials page under **API keys**.

## Problem Breakdown

The implementation of this application can be broken down into three parts:

1. Receive and read the SMS message.
2. Extract the source and destination locations from the SMS text and get the traffic details from Google Maps API.
3. Process the traffic data and send back the SMS in a textual and human-readable format.

## Setup

As our application will be developed in Node.js, make sure you have [Nodejs](https://nodejs.org/en/) installed in the system.

Install the dependencies by running this command in your terminal:

```bash
npm install express body-parser dotenv
```

Create a single file named `App.js` which will run the application on port `3000` and handle all the processing inside it.

We have used the `dotenv` npm package to get the environment variables. Using this, we can store secrets safely.

```javascript
const app = require('express')()
const bodyParser = require('body-parser')

if (process.env.NODE_ENV !== "production") {
    require("dotenv").config();
}

app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: true }))

app.listen(process.env.PORT || 3000, "127.0.0.1", () => {
    console.log("Server Running on Port ", process.env.PORT || 3000);
});
```

To store and access the environment variables, create a `.env` file in your root directory and add the properties.

```
PORT = 3000
```

These properties can be accessed using `process.env.PORT`

### Receive SMS

The first step is to receive the SMS and read it. Vonage provides a Node.js SDK to use SMS clients within our Node application.

You can follow along with [this tutorial](https://www.google.com/url?q=https://developer.vonage.com/en/blog/send-and-receive-sms-messages-with-node-js-and-express&sa=D&source=docs&ust=1676867052865733&usg=AOvVaw0zn-dLVEG5l5LYpWOzuBT-)  on how you can receive the Message in a Node.js application

```bash
npm install @vonage/server-sdk
```

The next step is to configure the inbound API and add it to the Vonage console. This way, whenever an SMS is sent to our number, the inbound API will be called and we can read that SMS.

There is a detailed tutorial on how you can [set up the inbound API and read the SMS](https://developer.vonage.com/messaging/sms/code-snippets/receiving-an-sms/node).

The inbound API is what we expose to any third-party application to which they send the data.

Any `GET` or `POST` request that our inbound API `/webhooks/inbound-sms` receives will be passed to the `handleInboundSms` function.

```javascript
app.route("/webhooks/inbound-sms")
.get(handleInboundSms)
.post(handleInboundSms);

function handleInboundSms(request, response) {
    // reads the SMS body
    const params = Object.assign(request.query, request.body);

    // process the SMS text
    getTrafficDetailsAndSendSMS(params);

    // notify that we have received SMS
    response.status(204).send();
}
```

To make this inbound API work, we will have to set our application live. We can do that by using [ngrok](https://ngrok.com/).

To set the application live, first [setup ngrok](https://ngrok.com/docs/getting-started).

And then run the node app locally

```bash
node App.js
"Server Running on Port 3000"
```

Once the local app is running, we can map it and set it live using ngrok.

```bash
ngrok http 3000
```

Expected output:

![Ngrok CLI output](/content/blog/traffic-alert-through-sms-using-vonage-and-google-maps-api/ngrok-cli.png)

Once the app is live, we will receive a public URL, for example `https://58c8-103-179-3-99.in.ngrok.io`

Add this URL to the Vonage dashboard so that we can listen to the inbound API.

![Vonage SMS dashboard](/content/blog/traffic-alert-through-sms-using-vonage-and-google-maps-api/inbound-sms-configuration-vonage-dashboard.png)

Now that we are able to receive the SMS, let's process the text and get the traffic information from Google.

- - -

### Get the Traffic Information Between Two Routes Using Google Maps API

To extract the source and destination information from the SMS that we receive through the Inbound API, the SMS text needs to be in a certain format so that it can be parsed better. I have created a simple template that is easier to process.

For example, to receive information about traffic between Mumbai and Pune, let's create the following SMS:

```
Traffic between Mumbai and Pune
```

In the example above, `Mumbai` is the source, and `Pune` is the destination.

In the `getTrafficDetailsAndSendSMS(params);` function we can take the SMS body and then extract the source.

```javascript
function getTrafficDetailsAndSendSMS(params) {
    // get the source and destination by parsing the text
    const { origin, destination } = parseText(params);

    // get the traffic details and routes
    const routes = getTrafficDetails({ origin, destination });

    // send back the SMS
    sendSMS(params.msisdn, routes);
}
```

#### Parse Text

To find the routes, we will have to extract the source and destination from the SMS that we have received.

The `parseText(params)` method extracts the source and destination from the SMS text and returns that.

```javascript
function parseText({ text }) {
    let sampleText = "Traffic between mumbai and pune";
    if (text) {
   	 sampleText = text;
    }

    sampleText = sampleText.trim();
    const characters = sampleText.split(" ");
    const origin = characters[2];
    const destination = characters[4];

    return { origin, destination };
}
```

#### Get Traffic Details

Now this `source` and `destination` can be passed to the [Google Distance Matrix API](https://developers.google.com/maps/documentation/distance-matrix/distance-matrix#distance-matrix-advanced) that gets the traffic details.

To make an API call from our application we would require `axios`, so let's add that dependency.

```bash
npm install axios
```

To get the traffic information we have to pass the `departure_time` in the query parameter. We are passing `now` as its value to get the current traffic details.

Also, we have set the `mode` as `driving`. This can be made configurable and can be accepted through the SMS along with the routes to provide a better user experience.

```javascript
async function getTrafficDetails({ origins, destinations }) {
    try {
   	 const YOUR_API_KEY = process.env.GOOGLE_MAP_API_KEY;
   	 const mode = "driving";
   	 const departure_time = "now";

   	 var  config = {
   		 method:  "get",
   		 url:  `https://maps.googleapis.com/maps/api/directions/json?origins=${origins}&destinations=${destinations}&mode=${mode}&departure_time=${departure_time}&language=en-US&key=${YOUR_API_KEY}`,
   	 };

   	 let response = await axios(config);
   	 response = await response.data;

   	 const  routes = getRoutes(response);
   	 return  routes.join(" \n\n ");
    } catch (e) {
   	 console.error("There was some error while getting routes details", e);
    }
}
```

Following is the sample response from the Google Maps API.

```json
const response = {
  	routes: [
    	{
      	bounds: {
        	northeast: { lat: 41.8781139, lng: -87.6297872 },
        	southwest: { lat: 34.0523525, lng: -118.2435717 },
      	},
      	copyrights: "Map data ©2022 Google, INEGI",
      	legs: [
        	{
          	distance: { text: "579 km", value: 932311 },
          	duration: { text: "8 hours 48 mins", value: 31653 },
          	duration_in_traffic: { text: "8 hours 55 mins", value: 932311 },
          	end_address: "Panvel",
          	end_location: { lat: 37.0842449, lng: -94.513284 },
          	start_address: "Mumbai",
          	start_location: { lat: 41.8781139, lng: -87.6297872 },
          	steps: [],
          	traffic_speed_entry: [],
          	via_waypoint: [],
        	},
        	{
          	distance: { text: "217 km", value: 349512 },
          	duration: { text: "3 hours 17 mins", value: 11799 },
          	duration_in_traffic: { text: "3 hours 40 mins", value: 932311 },
          	end_address: "Alephata",
          	end_location: { lat: 35.4675612, lng: -97.5164077 },
          	start_address: "Panvel",
          	start_location: { lat: 37.0842449, lng: -94.513284 },
          	steps: [],
          	traffic_speed_entry: [],
          	via_waypoint: [],
        	},
        	{
          	distance: { text: "1,328 km", value: 2137682 },
          	duration: { text: "19 hours 28 mins", value: 70097 },
          	duration_in_traffic: { text: "20 hours 22 mins", value: 932311 },
          	end_address: "Pune",
          	end_location: { lat: 34.0523525, lng: -118.2435717 },
          	start_address: "Alephata",
          	start_location: { lat: 35.4675612, lng: -97.5164077 },
          	steps: [],
          	traffic_speed_entry: [],
          	via_waypoint: [],
        	},
      	],
      	summary: "I-55 S and I-44",
      	warnings: [],
      	waypoint_order: [0, 1],
    	},
  	],
  	status: "OK",
	};
```

[Check the complete response structure](https://github.com/pra9shinde/google-vonage-traffic-app/blob/main/package.json)

We have processed the response that we received from the Google Direction API and formed a readable string from it that will show the different routes we can take and how much time and distance it may take to reach the destination. 

To form the string, we took all the paths for the given routes, their distance, and the time to reach the destination normally and with traffic.

We will convert all the routes to a human-readable string and then return them together.

To process the response, we will be using helper functions.

```javascript
const  getRoutes = ({ routes }) => {

// calculate the overall distance of all the routes

return  routes.reduce((a, b, l) => {
const { legs } = b;

 

let  via = "";
let  normalTime = 0;
let  timeInTraffic = 0;

 

// for each route, calculate the distance path wise, from one to another

const  distance = legs.reduce((x, y, i) => {

const { distance, duration, duration_in_traffic, steps, start_address, end_address } = y;

normalTime += getTimeInMinutes(duration.text);

timeInTraffic += getTimeInMinutes(duration_in_traffic.text);

 

if (i !== legs.length - 1) {

via = via ? via + " -> " + end_address : end_address;

}

const  string = `From ${start_address} to ${end_address} it takes ${duration.text} normally and ${duration_in_traffic.text} in traffic to cover a distance of ${distance.text}`;

x.push(string);
return  x;
}, []);

// for the final string
const  finalString = `Route${l + 1} via ${via} will take ${processTime(
normalTime
)} normally and ${processTime(
timeInTraffic
)} in traffic. Path via breakdown is: ${distance.join(" AND ")}`;

// push the string
a.push(finalString);
return  a;
}, []);
};

// helper function extract to hours and minutes from text
// and return time in minutes

const  getTimeInMinutes = (timeText) => {

timeText = timeText.split(" ");

let  hrs = timeText[0];

let  mins = timeText[2];

return  parseInt(hrs) * 60 + parseInt(mins);

};

 

// helper function to get hours and mins from the time

const  processTime = (time) => {

const  hrs = Math.floor(time / 60);

const  mins = time % 60;

return  `${hrs} hours ${mins} mins`;

};
```

This method will return us the array of routes with distance via path in normal time and in traffic and we can return them in a textual format.

### Send Back the SMS with Traffic Details

In the `getTrafficDetails()` function we are processing a traffic API response and using this response to generate a text that contains the traffic details. The output text looks like this:

```
"Route1 via Panvel -> Alephata will take 31 hours 33 mins normally and 32 hours 57 mins in traffic. Path via breakdown is:  From Mumbai to Panvel it takes 8 hours 48 mins normally and 8 hours 55 mins in traffic to cover a distance of 579 km AND From Panvel to Alephata it takes 3 hours 17 mins normally and 3 hours 40 mins in traffic to cover a distance of 217 km AND From Alephata to Pune it takes 19 hours 28 mins normally and 20 hours 22 mins in traffic to cover a distance of 1,328 km"

"Route2 via Eastern Express Highway -> Lonavala will take 11 hours 33 mins normally and 12 hours 57 mins in traffic. Path via breakdown is:  From Mumbai to Eastern Express Highway it takes 7 hours 48 mins normally and 7 hours 55 mins in traffic to cover a distance of 579 km AND From Eastern Express Highway to Lonavala it takes 2 hours 17 mins normally and 2 hours 40 mins in traffic to cover a distance of 200 km AND From Lonavala to Pune it takes 1 hours 28 mins normally and 2 hours 22 mins in traffic to cover a distance of 1,28 km"
```

This generated text contains the time it takes to reach the destination from the source location during normal and peak hours. We can send this information back to the same phone number from which we received the input.

We can make use of the Vonage SMS API to send the information back. When we receive the SMS we also receive the sender's mobile number to which we can send the SMS back.

```javascript
async function sendSMS(msisdn, routes) {
    try {
   	 const data = qs.stringify({
   		 from: "Vonage APIs",
   		 text: routes,
   		 to: msisdn,
   		 api_key: process.env.VONAGE_API_KEY,
   		 api_secret: process.env.VONAGE_API_SECRET,
   	 });

   	 const config = {
   		 method: "post",
   		 url: "https://rest.nexmo.com/sms/json",
   		 headers: {
   		 "Content-Type": "application/x-www-form-urlencoded",
   		 },
   		 data: data,
   	 };

   	 let response = await axios(config);
   	 response = await response.data;
    } catch (e) {
   	 console.error("There was some error while sending sms", e);
    }
}
```

You can find the source code of this application [here](https://github.com/pra9shinde/google-vonage-traffic-app)

## Conclusion

Now that we have created the instant traffic alert through SMS notification, you can reference this article to create a different alert system with SMS, or via WhatsApp using [Vonage Messages API](https://www.vonage.com/communications-apis/messages/). 

Engagement from the community is always welcome. Join Vonage on [GitHub](https://github.com/Vonage) for code samples and the [Community Slack](https://developer.vonage.com/community/slack) for questions or feedback. Send Vonage developers a [tweet](https://twitter.com/vonagedev) and let them know about something cool you built using Vonage APIs.

You can also reach out to me on my blog [learnersbucket.com](https://learnersbucket.com) where I write web development articles.