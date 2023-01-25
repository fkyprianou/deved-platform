---
title: Introducing Vonage Silent Authentication
description: Want to speed up your users' authentication flow using GSM? Vonage does that.
thumbnail: /content/blog/introducing-vonage-silent-authentication/silent-authentication.png
author: james-seconde
published: true
published_at: 2022-12-16T07:26:48.108Z
updated_at: 2022-12-16T07:26:49.237Z
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
Vonage is excited to bring Silent Authentication into our Verify API! In this article, we'll walk through what it is, what it does, and how to use it. So, to get started, we'll answer the following:

## **What is Silent Authentication?**

Silent Authentication is a recent development in personal authentication workflows that uses your mobile phone's [Subscriber Identity Module](https://ec.europa.eu/eurostat/cros/content/Glossary%3ASubscriber_Identity_Module_%28SIM%29_en) (SIM) as your identity, checked against the carrier's records to make sure your number is active and genuine. Once a request has been verified, a consumer vendor/client can then hit the same endpoint each time a user wants to authenticate, which will be valid until the request expires, or is cancelled by the client. What this effectively translates to is: *using* *your mobile phone as your password*.

## **What are the advantages of using Silent Authentication?**

[One-Time Passwords](https://workos.com/blog/guide-to-one-time-passwords-otps) (OTP) are used extensively for two-factor authentication across the world. The de-facto main use of our [Verify API](https://developer.vonage.com/api/verify.v2) is to use an OTP sent to a customer's SMS. Let's be honest here, though: as a customer, OTPs are *annoying*. Sure, it's an added layer of security (we'll get to why this is not necessarily true shortly). Still, for a vendor that relies on as few clicks or navigation points in a workflow (such as e-commerce), you're increasing the resistance to completing a flow (i.e. purchasing an item). E-commerce usually requires the path to least resistance, so from that perspective, Silent Authentication completely removes that extra step to logging in or checking out your cart.

Security is also a concern - it's a common misconception that 2FA makes user flow extremely hard to crack, but this does not take into consideration that [SMS phishing](https://cybersecurity.att.com/blogs/security-essentials/sms-phishing-explained-what-is-smishing) is a highly prevalent attack vector used by scammers. By moving authentication directly between the carrier and the mobile device, the threat of phishing is removed.

## **Any disadvantages?**

In true developer style, we wouldn't be doing our jobs right if we didn't point out that nothing is a "fix-all" when it comes to user verification - just like in engineering, there are tradeoffs. There are two main disadvantages to using it:

* Silent Authentication shares a common blocker with SMS verification in that the user requires a mobile device - there's no getting around this. While statistics show that smartphone ownership across Western nations is on a trajectory to hit above 90% soon ([88% of the UK population owns a smartphone, for instance](https://www.uswitch.com/mobiles/studies/mobile-statistics/#:~:text=As%20of%202021%2C%2088%25%20of,of%20adults%20had%20a%20smartphone.)), it does not take into account a global audience where service quality or economics might be active blockers.
* The users' mobile *must be using cellular data* in order for it to work. Users often have 3/4g enabled on their devices whilst *also* having Wi-Fi turned on. In this circumstance, Silent Authentication will not work as it requires a verified GSM response from the device to prove its credentials - Wi-Fi could be *anything*, and so will not work. It's a small step, but nevertheless an extra complication in a process you want to be as simple as possible.

**How to make a Silent Authentication Verify Request**

Silent Authentication is available in [v2 of the Verify API](https://developer.vonage.com/api/verify.v2) and has been added as an available channel. As it relies on an external vendor (Vonage) to complete your workflow, we've chosen to make this process asynchronous only using webhooks - it makes no sense to leave the process waiting when there are factors like a cellular network giving us a response (e.g. what if the network is down?).

We'll demonstrate this workflow using cURL. First off, we'll want to make a call to start the process:

```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"brand": "Vonage", "workflow": {"channel": "silent_auth", "to": "447700900000"}}' \
-H "Authorization: Bearer ZHNmZHNnc2Q="
https://api.vonage.com/v2/verify
```

Providing you have not hit throttle limits and the authorisation token is correct, you should get an HTTP `202` response with your `request_id` reference:

```json
{ "request_id": "c11236f4-00bf-4b89-84ba-88b25df97315" }
```

This is where we now rely on callbacks - the webhook responses showing the status of the request will be pushed to your Vonage application's events callback URL, defined in the Applications Dashboard. If the request is verified for use, you will get an event callback with the following payload:

```json
 {
  "request_id": "c11236f4-00bf-4b89-84ba-88b25df97315",
  "triggered_at": "2020-01-01T14:00:00.000Z",
  "type": "event",
  "channel": "silent_auth",
  "status": "action_pending",
  "action": {
    "type":"check",
    "check_url": "https://eu.api.silentauth.com/phone_check/v0.1/checks/c11236f4-00bf-4b89-84ba-88b25df97315/redirect"
  }
}
```

At this stage, it is possible that the request will fail for reasons such as an error in the network. In this case, the Verify workflow will fall back to the next available channel to use sequentially - we've not specified another channel so it will fail. You might want to fall back to SMS OTP - in which case you'd add that channel in the first request in the workflow array.

You now have a valid Silent Authentication URL to hit, contained in the `check_url` field. Until the request expires or is cancelled by you, the URL will perform a Silent Authentication check. **One of the most important aspects of the workflow** here is that the URL **MUST** be hit by an end device that is using cellular data. Without a cellular connection, the carrier has no device to verify. 

If you hit this URL with your cellular data on, Wi-Fi off and it is valid, you'll get a callback to your defined events URL like so:

```json
{
  "request_id": "c11236f4-00bf-4b89-84ba-88b25df97315",
  "triggered_at": "2022-12-10T14:00:00.000Z",
  "type": "event",
  "channel": "silent_auth",
  "status": "completed",
}
```

Success! You can send your user through. However, for those less fortunate:

```json
{
  "request_id": "c11236f4-00bf-4b89-84ba-88b25df97315",
  "triggered_at": "2020-01-01T14:00:00.000Z",
  "type": "event",
  "channel": "silent_auth",
  "status": "user_rejected",
}
```

Whoops! We've got a rejection, this person isn't who they say they are!

## **Stay Tuned**

Silent Authentication marks the start of a series of products we're working on this year. [You can register with us](https://www.vonage.com/communications-apis/verify/) if you want to start using Silent Authentication to activate developer preview, and in the meantime - we've got some nifty products we're excited about in 2023. Want to keep up to date? [Follow our Twitter Developer Account](https://twitter.com/VonageDev) or [check out our developer docs](https://developer.vonage.com/documentation). Happy Holidays!