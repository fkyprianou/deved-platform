---
title: Adding End-To-End Media Encryption to Your Video Calls
description: When privacy for your video calls are essential, having End-To-End
  Media Encryption is a must! Vonage now offers this feature.
author: dwanehemmings
published: true
published_at: 2023-07-28T16:07:46.125Z
updated_at: 2023-07-28T16:07:46.162Z
category: announcement
tags:
  - end-to-end-media-encryption
  - video-api
  - webrtc
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Privacy is essential, especially in communications on the internet. We want to ensure only the people allowed can participate in a video call. Luckily, WebRTC media (audio/video) streams are encrypted by default when making peer-to-peer connections using [Secure Real-time Transport Protocol (SRTP)](https://developer.mozilla.org/en-US/docs/Glossary/RTP) in browsers. This is what happens when you create a Vonage Video session with the media mode set to relayed.

![A diagram of two people icons with a two-sided arrow between them to represent a session with the media mode set to relayed.](/content/blog/adding-end-to-end-media-encryption-to-your-video-calls/media-mode-relayed.jpg)

This works great when just two to three participants are on a video call. The bandwidth needed to send media to each participant dramatically increases as the number of participants increases which can decrease the quality of the streams.

![A diagram with 6 person icons with two-sided arrows criss-crossing over each other connecting each user with the others to represent the complexity that happens as the number of participants increase.](/content/blog/adding-end-to-end-media-encryption-to-your-video-calls/multiple-participants-relayed.jpg)

For many participants (multiparty) in a video call, Vonage Video has a routed media mode. Here the created session uses the OpenTok Media Router to route the media streams between clients (still encrypted with STRP), thus saving on the bandwidth required.

![A diagram of 6 people icons with a box labeled Media Router in the center with two-sided arrows pointing from each user icon to the box to show that less streams are required when using the media router..](/content/blog/adding-end-to-end-media-encryption-to-your-video-calls/media-router.jpg)

What if we need to encrypt the media between the participants going through the OpenTok Media Router? This is where the new [End-To-End Encryption feature](https://tokbox.com/developer/guides/end-to-end-encryption/) comes into play. A participant’s audio and video streams are encrypted with a 2nd layer encryption secret set at the client, sent through the OpenTok Media Router encrypted, and decrypted on the receiving client. Since the media is encrypted while passing through the media router, archiving, live streaming broadcasts, experience composer, audio connector, and SIP Interconnect are not available since they require decoding media.

## How It Works

It all starts on the server where a session is created with the media mode set to routed and the end-to-end encryption key set to true. When a client joins the session, it sets the encryption secret for its media. Every client in the session that has the same encryption secret will be able to decode the others’ media.

## Try It Out

Before you can use the demo application, you will need to add the end-to-end encryption feature in your [Vonage Video Account](https://tokbox.com/account/#/settings).

To take a look at some code and deploy the NodeJS-based demo application to StackBlitz, visit the [GitHub repo](https://github.com/opentok/opentok-web-samples/tree/main/End-To-End-Media-Encryption) 

## What’s Next?

Read more about the Vonage Video API End-To-End Encryption feature in the [documentation](https://tokbox.com/developer/guides/end-to-end-encryption/). If you have any questions or comments, please let us know in our [Community Slack Channel](https://developer.vonage.com/en/community/slack).



