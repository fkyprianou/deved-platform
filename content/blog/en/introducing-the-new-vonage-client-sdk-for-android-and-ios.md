---
title: Introducing the New Vonage Client SDK for Android and iOS
description: Introducing a new major version of the Vonage Client SDK. Including
  a lot of new features and some interesting technology under the hood.
thumbnail: /content/blog/introducing-the-new-vonage-client-sdk-for-android-and-ios/new-vonage-client-sdk.png
author: abdul-ajetunmobi
published: true
published_at: 2023-05-02T11:06:44.513Z
updated_at: 2023-05-02T11:06:44.721Z
category: announcement
tags:
  - client-sdk
  - swift
  - kotlin
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
The Vonage Client SDK has a new package name! With that new name comes a new major version with a lot of new features and fixes. Under the hood, the SDK is now utilizing Kotlin Multiplatform to share core functionality code between the native Android and iOS SDKs

Let's get into how you can start with the SDK. 

## Adding the SDK

As mentioned the SDK now has a new name, to add the SDK to your app you need to do the following:

### Android 

Add the dependency to your app's `build.gradle` file:

```
implementation 'com.vonage:client-sdk-voice'
```

### iOS

Add the dependency to your app's `Podfile`:

```
pod 'VonageClientSDKVoice'
```

## Key Concepts

When working with the SDK there are some concepts that you will need to be familiar with. 

### Session

A session is a live communication stream between the SDK and the Vonage servers. You can learn more about how to create a session further on in the blog.

### Invite

An invite is sent to the SDK when the current user has been invited to join a call. The invite listener/delegate function contains the requisite information about a call. Invites can either be accepted or rejected.

### Call

Once you successfully accept an invite you will get a call ID. This ID allows you to make changes to the voice call using the functions available. For example, hanging up the call, muting the call or using text-to-speech amongst others. 

### Leg

A leg is a connection between a user and a call, each leg is assigned an ID. A call is made up of multiple legs. So in the case of two SDK users being on a call, the connection between User A and Vonage would be leg 1, and the connection between User B and Vonage would be leg 2. With the call being a link being leg 1 and 2.

## Creating a Session

Now that you have the key concepts down, let us look at how to use the SDK. Before you make a call you need to create a session. You can create a session using the `createSession` function. 

### Android 

```kotlin
val config = ClientConfig(ConfigRegion.US) 
val client = VoiceClient(this.application.applicationContext)
client.setConfig(config)

client.createSession("JWT") { err, sessionID ->
    if err == null {
    // Connected!
    }
}
```

### iOS

```swift
let config = VGClientConfig(region: .US) 
let client = VGVoiceClient()
client.setConfig(config)

client.createSession("JWT") { error, sessionID in
    if error == nil {
    // Connected!
    }
}
```


This function takes a [JWT](https://jwt.io/) to identify which user the session would belong to. Check out the [JWT Guide](https://developer.vonage.com/en/conversation/guides/jwt-acl) for more information about creating JWTs for use with the Client SDK.

## Making a Call

Once you have successfully created a session, you can now make a call using the `serverCall` function:

### Android 

```kotlin
client.serverCall(mapOf("to" to "CALLEE")) { err, callID ->
    if err == null {
    // Connected to the call!
    }
}
```

### iOS 

```swift
client.serverCall(["to": "CALLEE"]) { error, callID in
    if error == nil {
    // Connected to the call!
    }
}
```

`serverCall` makes an HTTP request (via Vonage servers) to the Answer URL specified on your Vonage application. Your Answer URL has to return with a [Call Control Object](https://developer.vonage.com/en/voice/voice-api/ncco-reference) telling Vonage how to handle the call. 

## Receiving a Call

Let us see how it looks on the other side when you are receiving a call:

### Android 

On Android, you can set the call invite listener which will be run when the SDK gets an invite:

```kotlin
client.setCallInviteListener { callId, from, channelType ->
    client.answer(it) { err ->
        if err == null {
        // Connected to the call!
        }
    }
}
```

### iOS 

On iOS make sure to set the `VGVoiceClient`'s delegate using `client.delegate = self` first. When the SDK gets an invite the `didReceiveInviteForCall` function on the `VGVoiceClientDelegate` will be called:

```swift
func voiceClient(_ client: VGVoiceClient, didReceiveInviteForCall callId: String, from caller: String, withChannelType type: String) {
    client.answer(callId) { error in
        if error == nil {
        // Connected to the call!
        }
    }
}
```

### Push Notifications

The new Vonage Client SDK has made push notifications the default way to receive incoming calls, to ensure that all calls are received by devices whether your application is in the foreground or not. For the above behaviour you will need to enable WebSocket invites on your client's configuration:

```
config.enableWebsocketInvites = true
```

For more in-depth information on how to configure push notifications for the Vonage Client SDK, please refer to the new Push Notification guides for [Android](https://developer.vonage.com/en/vonage-client-sdk/set-up-push-notifications/android) and [iOS](https://developer.vonage.com/en/vonage-client-sdk/set-up-push-notifications/ios). 

## What's Next?

Learn more about the Vonage Client SDK on [developer.vonage.com](https://developer.vonage.com/en/vonage-client-sdk/overview), where you can try out the [step-by-step tutorial](https://developer.vonage.com/en/tutorials/vg-app-to-phone/introduction/kotlin) on how to make outbound calls.