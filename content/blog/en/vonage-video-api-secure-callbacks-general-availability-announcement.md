---
title: Vonage Video API - Secure Callbacks General Availability Announcement
description: Learn How To Secure Your Video Webhooks With a Feature Called Secure Callbacks
thumbnail: /content/blog/vonage-video-api-secure-callbacks-general-availability-announcement/videoapi_secure-callbacks.png
author: michael-crump
published: true
published_at: 2023-07-19T00:05:00.719Z
updated_at: 2023-07-19T00:05:00.731Z
category: announcement
tags:
  - video-api
  - releases
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

The [Vonage Video API](https://www.vonage.com/communications-apis/video/features/) has a new feature called [Secure Callbacks](https://tokbox.com/developer/guides/secure-callbacks/) which is in General Availability today. The secure callbacks feature provides a way to verify a webhook callback request is coming from Vonage and that its payload has not been tampered with during transit. This will protect against unauthorized callbacks and adds additional security benefits for those industries with specific compliance requirements. 

The Vonage Video API uses callbacks to provide events and status updates to the application for Session Monitoring, Archiving/Recording, SIP, and Experience Composer. The secure callbacks are signed by a signature secret that the user configures through the Video Account Portal, which can be verified by the receiving application to validate that the callback originated from Vonage.

## Enabling Secure Callbacks

Head over to the [Video API Dashboard](https://tokbox.com/account/) and select a project that you’d like to implement this feature. Scroll down until you see **Project Settings** and the **Secured Callback** slider button, as shown below. 

![Project Settings](/content/blog/vonage-video-api-secure-callbacks-general-availability-announcement/projectsettings.png "Project Settings inside the Video API Dashboard")

Press the **Edit** button and toggle the **Secured Callback** slider button to on, and press **Save**. 

\> Note: For restricted inbound traffic, [Vonage IP addresses](https://tokbox.com/developer/guides/secure-callbacks/#callback-ip-address) must be added to your firewall's list of approved IP addresses.

Now that the feature has been enabled let’s discuss the various API callbacks and what each does before enabling one. 

## Vonage Video API Callbacks

The Vonage Video API provides several API callbacks that can be secured with this feature, including: 

1. **Session Monitoring** - Monitors session activity.
2. **Archiving Monitoring** - Monitors archive (or recordings) status. 
3. **SIP Call Monitoring** - Monitors SIP call activity.
4. **Experience Composer Monitoring** - Monitors [Experience Composer](https://tokbox.com/developer/guides/experience-composer/).

We’ll use **Session Monitoring** for this example, but the knowledge gained in this section applies to any of the other callbacks mentioned. 

## Session Monitoring

Developers can monitor certain activities of clients from within their app server. By registering for callbacks, your callback URL will receive HTTP POST requests when any client connects or publishes.

To enable it on your current video project, scroll down until you see **Session Monitoring** and click **Configure.** 

![Session Monitoring](/content/blog/vonage-video-api-secure-callbacks-general-availability-announcement/sessionmonitoring.png "Session Monitoring inside the Video API Dashboard")

You can configure the callback URL and toggle the **Signature Secret** to on. It provides a randomly generated signature by default, but you can specify your own. Click **Submit** to continue. The system will report that secure callbacks have been configured with the URL and signature secret.

## Validate the Request

There are two primary ways to validate secure callbacks:

1. **Verifying the request** - Secure Callbacks will include a JWT in the Authorization header. The secret used to sign the request corresponds to the signature secret associated with the `api_key` included in the JWT claims. You can identify your signature secret on the Dashboard. It's recommended that signature secrets be no less than 32 bits to ensure their security.
2. **Verifying the payload** - Once you have verified the authenticity of the request, you may optionally verify the request payload has not been tampered with by comparing an SHA-256 hash of the payload to the `payload_hash` field found in the JWT claims. The payload has been tampered with during transit if they do not match. You only need to verify the payload if you are using HTTP rather than HTTPS, as Transport Layer Security (TLS) prevents [MITM attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack). 

An example of what the **Signed** **JSON Web Token** **(JWT)** would look like:

```json
// header
{
  "alg": "HS256",
  "typ": "JWT",
}
// payload
{
  "iat": 1587494962,
  "jti": "c5ba8f24-1a14-4c10-bfdf-3fbe8ce511b5",
  "iss": "Vonage",
  "payload_hash" : "d6c0e74b5857df20e3b7e51b30c0c2a40ec73a77879b6f074ddc7a2317dd031b",
  "api_key": "a1b2c3d",
  "application_id": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab"
}
```

Notice that the `payload_hash` field is an SHA-256 hash of the request payload. It can be compared to the request payload to ensure it has not been tampered with during transit.

A [Node.js code sample can be found here](https://tokbox.com/developer/guides/secure-callbacks/). 

## Wrap-up

To recap, **[Secure Callbacks](https://tokbox.com/developer/guides/secure-callbacks)** is a security feature that ensures that each callback has not been tampered with in transit. This defends against intercept and later replay. It verifies that the request originates from Vonage and ensures compliance needs are met for many important industries. You can use it today as a generally available feature, free of charge, by enabling it within your project. Please visit [this page](https://tokbox.com/developer/guides/secure-callbacks/#known-limitations-considerations) for a list of limitations and considerations, as information is updated frequently.