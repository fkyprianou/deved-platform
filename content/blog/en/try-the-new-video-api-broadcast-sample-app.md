---
title: Try The New Video API Broadcast Sample App
description: Try the new Video API Broadcast Sample App to easily stream live
  video to multiple viewers
thumbnail: /content/blog/try-the-new-video-api-broadcast-sample-app/video-broadcast-app.png
author: enrico-portolan
published: true
published_at: 2023-03-08T14:31:27.417Z
updated_at: 2023-03-08T14:31:27.452Z
category: announcement
tags:
  - video-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

The Video API broadcast sample app is a simple, yet powerful application that demonstrates how to use Video API's broadcasting capabilities to stream live video to multiple viewers. This app is an excellent starting point for developers who want to build live streaming applications, such as live events, webinars, or video conferencing tools.

The Vonage Video API makes it easy to build a custom video experience within any mobile, web, or desktop application, and is built on the WebRTC industry standard that is available on billions of devices. For more information, please visit [Vonage Video API](https://www.vonage.com/communications-apis/video/)

## What is the broadcast sample app?

The app is built using the Video API JavaScript SDK, which allows developers to create real-time communications applications on the web. With this SDK, developers can quickly add video chat, voice, and messaging to their websites and applications. The SDK also provides tools for building custom video and audio filters, capturing and sharing user screens, and more.

![Broadcast sample app](/content/blog/download-and-try-the-new-video-api-broadcast-sample-app/start_broadcast.png)

The broadcast sample app is available on [GitHub](https://github.com/opentok/broadcast-sample-app), and it comes with everything developers need to get started. The app includes a simple HTML page that displays a live video feed from a camera or microphone. It also includes JavaScript code that manages the Video API session and streams the video to multiple viewers.

To get started with the app, developers need to sign up for an (Video API account)\[https://tokbox.com/account/user/signup] and create an API key and secret. They can then copy these credentials into the app's configuration file (./config.json), along with other settings like the session ID and token. With these settings in place, developers can launch the app and start broadcasting their live video.

## Features of the broadcast sample app

One of the most powerful features of the broadcast sample app is its ability to scale. The app can easily handle hundreds or even thousands of viewers, thanks to Video API’s cloud infrastructure. The app uses a scalable media server to distribute the video feed to viewers, which means that developers don't need to worry about building their own media server infrastructure.

In addition to its scalability, the broadcast sample app also provides developers with a lot of customization options. For example, developers can customize the look and feel of the app's user interface, add custom branding or logos, and configure other settings like the video resolution and bitrate. This makes it easy to build a live-streaming application that meets the specific needs of a business or organization.

The broadcast sample app has several roles that can be defined based on the use case and requirements of the application. These roles include the host, guest, viewer, and experience composer.

## The role of the host in the broadcast sample app

The host is the person who initiates the broadcast and is responsible for setting up the Video API session. The host's video and audio feed are streamed to the viewers, and they have control over various settings such as video quality, audio quality, and more. The host also has the ability to manage guests, control access to the broadcast, and moderate any chat or Q&A sessions that are part of the broadcast.

![The role of the host in the broadcast sample app](/content/blog/download-and-try-the-new-video-api-broadcast-sample-app/host.jpg)

## The role of the guest in the broadcast sample app

A guest is someone who is invited to join the broadcast by the host. Guests may have limited access to the broadcast and may not have the same level of control as the host. For example, guests may not be able to start or stop the broadcast, but they may be able to participate in chat sessions or ask questions during a Q&A.

## Other roles in the broadcast sample app

### Viewer

The viewer is someone who is watching the broadcast but does not have the ability to interact with the host or other guests directly. Viewers can join the broadcast by clicking on a link or by entering a unique URL. They can see and hear the host's video and audio feed in real-time, and they may be able to participate in chat sessions or Q&A depending on the settings enabled by the host.

![The viewer in the broadcast sample app](/content/blog/switch_to_live_mode.jpg)

### Experience Composer

The experience composer is a role that involves creating a custom user experience for the broadcast. This may include custom branding, design elements, or other features that are tailored to the specific needs of the broadcast. The experience composer may work closely with the host to create a seamless user experience that is tailored to the specific requirements of the broadcast.

![Experience composer in the broadcast sample app](/content/blog/download-and-try-the-new-video-api-broadcast-sample-app/experience_composer.png)

## Conclusion

Overall, the (broadcast sample app)\[https://github.com/nexmo-se/broadcast-sample-app] is a flexible and customizable platform that can support many different use cases, depending on the roles that are defined. Whether you're using the app to host a live event, deliver a webinar, or build a video conferencing tool, the different roles in the broadcast sample app are essential components of the live streaming experience.

In conclusion, the Video API  broadcast sample app is a powerful tool for building live streaming applications on the web. It provides developers with a simple starting point and all the tools they need to create high-quality, scalable, and customizable video streaming applications. Whether you're building a live event platform or a video conferencing tool, the broadcast sample app is an excellent starting point.