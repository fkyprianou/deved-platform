---
title: Best Practices for Implementing Silent Authentication
description: Discover more about silent authentication using the Verify V2 API,
  with useful hints and tips for implementation.
author: helena-bower
published: true
published_at: 2023-05-17T09:43:46.590Z
updated_at: 2023-05-17T09:43:46.604Z
category: tutorial
tags:
  - verify-api
  - java
  - swift
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
With Verify V2 just being [released for General Availability](http://www.developer.vonage.com/vonage-verify-v2-is-now-ga-for-2fa-integrations), we wanted to dive a bit deeper into our Silent Authentication capability! In this article, we’ll talk you through how it works, along with some useful tools and tips for implementation.

## How Does Silent Authentication Work?

Silent Authentication is a channel in the Verify V2 API that allows you to complete authentication without a 2FA code. Once a user has entered their login credentials, it proves their identity by checking their SIM information against their carrier's records to ensure that their phone number is active and genuine. Once a request has been verified, you can continuously authenticate the user until either the request expires or the user cancels it.

In short, it’s an authentication method that uses a mobile phone's Subscriber Identity Module (SIM) to prove a user's identity *without user input*.

You can read more about how silent authentication works in our [previous blog post](https://developer.vonage.com/en/blog/introducing-vonage-silent-authentication) or the [developer docs](https://developer.vonage.com/en/verify/verify-v2/guides/silent-authentication); the rest of this article will talk you through some useful tips and features that you may want to consider when implementing silent authentication.

## Force a Mobile Data Connection

Silent Authentication is not just reliant on the user having a mobile phone - it also needs a cellular network connection. If the request is sent over Wi-Fi, it will throw an error. To help with this, we’ve provided iOS and Android SDKs that can help you to make an HTTP request over cellular even when on WiFi:

### Android

The [Android SDK is available on GitHub](https://github.com/Vonage/verify-silent-auth-sdk-android), where you can find information on permissions, compatibility, and examples of how to integrate silent auth into your applications:

```java
import com.vonage.silentauth.VGSilentAuthClient
// instantiate the sdk during app startup
VGSilentAuthClient.initializeSdk(this.applicationContext)
val resp: JSONObject = VGSilentAuthClient.getInstance().openWithDataCellular(URL(endpoint), false)
 if (resp.optString("error") != "") {
    // error
} else {
    val status = resp.optInt("http_status")
    if (status == 200) {
        // 200 OK
    } else {
        // error
    }
}
```

### i﻿OS

The [iOS SDK is also available on GitHub](https://github.com/Vonage/verify-silent-auth-sdk-ios), where you can find information on installation, compatibility, and usage examples:

```swift
import VonageClientSDKSilentAuth
let client = VGSilentAuthClient()
client.openWithDataCellular(url: url, debug: true) { response in
    if (response["error"]) != nil {
      // Handle error
    } else {
      let status = resp["http_status"] as! Int
      if (status == 200) {
          // Handle response
      } else {
        // Handle error
      }
    }
}
```

## Silent Authentication Sandbox

Testing silent authentication can be difficult. To test a successful verification, code must be run from an application running on a phone over a mobile network, which can be tricky to set up. To help with this, Vonage has provided a sandbox that bypasses the check with the carrier. Instead, you’ll use the data returned in your callbacks to complete the check yourself, removing the need for a mobile network connection.

To do this, add `"sandbox": "true"` to your workflow:

```bash
curl -X POST https://api.nexmo.com/v2/verify \
-H "Content-Type: application/json" \
-H "Authorization: Bearer XXXXX" \
-d '{"brand": "Your Brand",
    "workflow": [
    {"channel": "silent_auth", "to": "447700900002", "sandbox": "true"}
}'
```

This will bypass the carrier and send your request to the sandbox. You should then get a response containing your `request_id` reference:

```json
{ "request_id": "31eaf23d-b2db-4c42-9d1d-e847e75ab330" }
```

From this point on, we’ll be referring to your callbacks - the status of your request is pushed to your Vonage application’s callback URL as defined in the developer dashboard. You’ll need to find the callback for your request containing a `check_url`; open this URL in your browser to complete the check:

```json
{
   "request_id": "31eaf23d-b2db-4c42-9d1d-e847e75ab330",
   "triggered_at": "2020-01-01T14:00:00.000Z",
   "type": "event",
   "channel": "silent_auth",
   "status": "action_pending",
   "action": [
      {
         "type": "check",
         "check_url": "https://eu.api.silent.auth/phone_check/v0.1/checks/:id/redirect"
      }
   ]
}
```

Once you’ve done this, you’ll receive another callback showing the check was completed:

```json
{
    "request_id": "31eaf23d-b2db-4c42-9d1d-e847e75ab330",
    "submitted_at": "2023-05-09T14:05:20.000Z",
    "finalized_at": "2023-05-09T14:05:40.000Z",
    "type": "summary",
    "channel": "silent_auth",
    "status": "completed"
}
```

Note: if you do not open the `check_url` within 25 seconds, the status will be `expired`.

The [complete guide on using the silent authentication sandbox can be found here](https://developer.vonage.com/en/verify/verify-v2/guides/silent-auth-sandbox).

## Fallback to Other Channels

There are several situations where silent authentication may not work, for example, if the user is:

* On a desktop device
* Out of their network’s coverage
* On a Wi-Fi connection instead of their cellular network

In these cases, you have the option to fallback to other channels. You can do this by configuring the workflow in your request; in this example, silent auth will be attempted first. If that fails, an SMS will be sent:

```bash
curl -X POST https://api.nexmo.com/v2/verify \
-H "Content-Type: application/json" \
-H "Authorization: Bearer XXXXX" \
-d '{"brand": "Your Brand",
    "workflow": [
    {"channel": "silent_auth", "to": "447700900002"},
    {"channel": "sms", "to": "447700900002"}]
}'
```

Read more about [workflows and the available channels here](https://developer.vonage.com/en/verify/verify-v2/overview#workflows).

## Conclusion

Hopefully, you have found this roundup of hints and tips useful, and it’ll help you with your future projects using Silent Authentication! You can [sign up for an account](https://dashboard.nexmo.com/) to start using Verify V2 now, follow our [Twitter Developer Account](https://twitter.com/VonageDev) to keep updated and check out our [developer docs](https://developer.vonage.com/en/verify/verify-v2/overview) for more information.