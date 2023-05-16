---
title: Live Captions API Is in Beta!
description: The Vonage Live Captions API is in Beta and here's what you need to know.
author: dwanehemmings
published: true
published_at: 2023-05-16T20:30:36.817Z
updated_at: 2023-05-16T20:30:36.853Z
category: announcement
tags:
  - video-api
  - live-captions
  - demo
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
The Vonage Live Captions API is in Beta and ready to be used in your application. Here are a few things to know.

# Why Offer Live Captions?

**Accessibility**: It can not be assumed that everyone participating in a call can hear.

**Noisy Environments**: Even with the best noise-canceling headphones/earbuds, a loud area can be challenging.

**Translation**: It is just one more step to turn a caption into the language of the viewer.

**Regain context**: Missed what someone just said? You can most likely still see the caption in the feed.

**Preference**: According to a poll from [YouGov](https://yougov.co.uk/topics/media/survey-results/daily/2023/02/24/9a34f/3), a sizeable amount of people prefer to have captions/subtitles on. I know I do.

![](/content/blog/live-captions-api-is-in-beta/yougov-poll.jpeg)

# How the Live Captions API Works.

The Live Captions API takes the audio streams (from both Video and SIP dial-in participants) that come through the Media Router and passes them to a transcription service.

![](/content/blog/live-captions-api-is-in-beta/live-captions.png)

# Advantages for developers.

* Live Captions is enabled by default for all projects.
* Your application is already sending media streams to the Media Router.
* No need to further strain your users' computers and/or mobile devices by sending another stream to be transcribed.
* No third-party transcription library/service to learn and implement.

# Enabling Live Captions in Your Application.

A more detailed description can be found in the [documentation](https://tokbox.com/developer/guides/live-captions/).

First, a POST request will need to be made to the Live Captions API endpoint with some credentials. Then you can use any of the many Client SDKs that we offer to interact with the API to start/stop sending and receiving captions.

# Give It a Try.

Instantly deploy a Basic Live Captions API demo to [Stackblitz](https://stackblitz.com/fork/github/opentok/opentok-web-samples/tree/main/Basic-Captions) and point to a running server URL in config.js. The source code can be found in the GitHub [repository](https://github.com/opentok/opentok-web-samples/tree/main/Basic-Captions).

# Got Any Questions or Feedback?

We would love to hear from you. Please reach out to us on our [Community Slack Channel](https://developer.vonage.com/en/community/slack). If you are on Twitter, follow the [VonageDev](https://twitter.com/vonagedev) account to receive the latest updates.