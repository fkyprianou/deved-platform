---
title: Vonage Verify V2 is now GA for 2FA Integrations
description: The new version of Verify adds new channels and more customisation
author: james-seconde
published: false
published_at: 2023-05-16T18:57:59.057Z
updated_at: 2023-05-16T18:57:59.081Z
category: announcement
tags:
  - verify-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
We're excited to announce that Verify, our API for Two-Factor Authentication (2FA), has just had Version 2 released for General Availability! This evolution of our 2FA solution has been designed to work better for developers, by using Webhooks for asyncronous integrations and offering more options and flexibility. Let's go through the differences between V1 and V2. You can also [check out a comprehensive guide to switching versions on this page in our documentation](https://developer.vonage.com/en/verify/verify-v2/guides/verify-migration-guide).

### Goodbye Polling, Hello Webhooks

Version 1 of Verify was designed to have a more synchronous flow - an example of this is after starting a request, the API needs to be polled if you need to know the status of the request before the user submits a PIN code (which effectively counts as the 'start' and 'end' of the request lifecycle).

The first iteration of Verify was built around a synchronous flow, with API polling required to check the status of a request after its initiation and prior to the submission of the user's PIN code.

In contrast, Verify Version 2 harnesses the power of webhooks. Initiating a request now provides you with a unique GUID. Your integration, assuming JWTs are your authorization method, will listen for incoming webhooks corresponding to the request GUID for updates. The associated endpoints with these webhooks are:

-   Initiate the request
-   Verify the user's PIN
-   Cancel a request

## Strengthened Fraud Protection

With the surge in fraudulent activity exploiting Communications APIs, we have integrated the [Verify Anti-Fraud System](https://developer.vonage.com/en/verify/verify-v2/guides/v2-anti-fraud) with Version 2. This system detects suspicious activity and triggers a Network Block. For added flexibility, users can also [customize this feature](https://developer.vonage.com/en/verify/verify-v2/guides/v2-anti-fraud) according to their requirements if necessary.

## Enhanced Delivery Methods

The most significant change we've made to the API is, naturally, adding new communication channel options. When you start a new request, the following existing methods can be used:

* SMS
* Voice Text to Speech (TTS)

You can now use these new methods:

- WhatsApp
- WhatsApp (Interactive Yes/No prompt)
- Email
- Silent Authentication

I make that a product expansion of... 200%!

## Enhanced Workflow Control

Related to the new channels, you now have the ability to control exactly how your request workflow is structured. Previously, you would send a `workflow_id` in the request, taken from a [predefined list in our developer portal](https://developer.vonage.com/en/verify/guides/workflows-and-events). Instead, for V2 you can include a custom payload for your workflow. For instance, if you want the attempted order of channels to be Silent Auth -> Email -> SMS, the request would look like this:

```json
{
   "brand": "ACME, Inc",
   "workflow": [
      {
         "channel": "silent_auth",
         "to": "44770090000X"
      },
	  {
         "channel": "email",
         "to": "alice@company.com",
         "from": "bob@company.com"
      },
      {
         "channel": "sms",
         "to": "44770090000X"
      }
   ]
}
```

## Custom PIN Generation

Developers also have the ability to send a custom code for channels that require it (i.e. all apart from WhatsApp Interactive and Silent Authentication). This code can be between 4 and 10 characters in length and is alphanumeric. Here's an example of the JSON payload sent when using your own generated PIN:

```json
{
   "brand": "ACME, Inc",
   "code" : "R4Fe4dR1Qz",
   "workflow": [
      {
         "channel": "sms",
         "to": "447700900000"
      }
   ]
}
```

## More REST, Please

Version 2 of Verify makes better use of HTTP response codes. Yes, yes, OK: so not REST (I just wanted to put it in the heading), but better use of the HTTP protocol. Here are some examples:

- When starting a request that has already been fired, you now get a [409](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409) response.
- Hitting the rate limit now gives you a standard [429](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)
- An invalid payload for either the request start or PIN submission endpoints gives you a [422](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/422)
- This one is quite a nice use case: if you submit an incorrect PIN too many times, you eventually get a [410](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/410) to indicate the request entity is now no longer available for any state changes.

## What Will YOU Build?

The new channels we've launched now give developers a wealth of options to integrate 2FA into their systems, side project,s and enterprise applications. The question is, what have you built? A passionate side-project turned startup with Laravel or Rails? A rollout across a Node microservice architected enterprise? We want to know! 