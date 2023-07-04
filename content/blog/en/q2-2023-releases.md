---
title: Spring 2023 at a Glance
description: All Vonage releases from April to June 2023
thumbnail: /content/blog/spring-2023-at-a-glance/quaterly-releases_q223.png
author: diana-pham-1
published: true
published_at: 2023-07-06T21:35:45.175Z
updated_at: 2023-07-06T21:35:45.189Z
category: release
tags:
  - releases
  - video-api
  - verify-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
And just like that, we're halfway into 2023! Our Vonage DevRel team wrapped up another season of dropping game-changing releases. Let's check them out.

# Leveling Up in Accessibility, Availability, and Armor (Security)

## [Live Captions API Is in Beta!](https://developer.vonage.com/en/blog/live-captions-api-is-in-beta)

Dwane announces the Beta release of Vonage's Live Captions API, now ready to run in your applications. Live captioning provide many benefits. They improve accessibility for users by providing dialogue visually when environments are noisy. They allow developers the ability to provide real-time multilingual translation.

## [New Vonage Client SDKs for Android and iOS](https://developer.vonage.com/en/blog/introducing-the-new-vonage-client-sdk-for-android-and-ios)

We're revving things up in Q2 with the grand unveiling of the newly renamed Vonage Client SDK! It's coming in hot with a new major version full of fresh features and fixes. This SDK now uses Kotlin Multiplatform, sharing core functionality between Android and iOS SDKs.

Our developer advocate Abdul covers key concepts, including Session, Invite, Call, and Leg, to familiarize you with its workings. He also guides you through creating a session, making and receiving a call, and push notifications.

## [Get Even More Out of Your 2FA Solution With Verify](https://developer.vonage.com/en/blog/vonage-verify-v2-is-now-ga-for-2fa-integrations)

In a significant Q2 milestone, we're delighted to share the [General Availability of Version 2 of our Verify API for Two-Factor Authentication (2FA)](https://developer.vonage.com/en/blog/vonage-verify-v2-is-now-ga-for-2fa-integrations). This revised 2FA solution caters to developers, employing Webhooks for asynchronous integrations, which offers more choices and flexibility. With the synchronous flow of V1 behind us, we can now use webhooks with V2, which gives a unique GUID upon initiating a request. We've also bolstered fraud protection with the integration of the Verify Anti-Fraud System to ward off malicious activities.

V2 also expanded the communication channel options, supporting WhatsApp, Email, and Silent Authentication, alongside existing SMS and Voice Text to Speech (TTS). Developers now have complete control over the structure of their request workflow. Lastly, the enhanced usage of HTTP response codes offers better protocol compliance. [Try Verify V2 now ](https://developer.vonage.com/en/verify/overview)and experience a more intuitive, secure, and flexible 2FA integration!

# Roll the Cameras: Our Video Toolkit Just Got Bigger!

## [Vonage Video API - Secure Callbacks Public Beta Announcement](https://developer.vonage.com/en/blog/vonage-video-api-secure-callbacks-public-beta-announcement)

Our Video API team has rolled out a new feature called Secure Callbacks, currently in public beta and anticipated for a full launch. Michael explains how the feature validates webhook callback requests from Vonage and ensures their payload hasn't been tampered with. It's a big win for app security and also a plus for compliance-heavy industries.

## [Vonage Video API - React Native SDK Gains Official Support](https://developer.vonage.com/en/blog/vonage-video-api-react-native-sdk-gains-official-support)

The React Native SDK for the Vonage Video API transitions from community to official Vonage support! This means improved, consistent assistance for developers and boosted confidence when integrating the SDK, assured of routine updates and long-term maintenance. Need help? Our Vonage support team is ready to assist. For guides and samples, check our official documentation, the React Native package on npm, and the SDK source code on the SDKs GitHub repository.

## [Vonage Video API macOS SDK Hits General Availability](https://developer.vonage.com/en/blog/vonage-video-api-macos-sdk-goes-ga)

After an awesome Beta ride, the Vonage Video API MacOS SDK is now officially making its mark with its first-ever stable release. Now, you can flexibly use the MacOS SDK in production, and expect the same maintenance and frequent updates as our other SDKs (JavaScript, Android, iOS, Windows, and Linux). Whether you're just starting out or eager to dive straight into the code, we've got you covered. Our official documentation and MacOS SDK sample repo are filled with all the details and sample apps to kick-start your journey with the SDK.

# Keeping Our Head in the Cloud

## [A Better Way to Build & Deploy Programmable Communications](https://developer.vonage.com/en/blog/announcing-cloud-runtime-marketplace)

We're thrilled to introduce the Beta availability of the[ Vonage Cloud Runtime Marketplace](https://developer.vonage.com/en/cloud-runtime), our innovative cloud-native, serverless development platform! Vonage Cloud Runtime, the very same tool that our Solutions and Services teams use to deploy API interactions, is now at your fingertips. We have easy-to-use guides and tools that take your application build, deployment, and production from zero to hero. And here's the kicker - we host this serverless platform, so you don't have to worry about maintaining infrastructure to use Vonage APIs! During this Beta period, there's no fee to access the Vonage Cloud Runtime, so you can dive right in and start crafting extraordinary customer engagement applications today!

## MongoDB series

1. [Part 1 - What is MongoDB Atlas?](https://developer.vonage.com/en/blog/using-vonage-apis-with-mongodb-atlas-part-1)
2. [Part 2 - Using Vonage Verify with Logins](https://developer.vonage.com/en/blog/using-vonage-apis-with-mongodb-atlas-part-2)
3. [Part 3 - Using Vonage for Customer Interactions](https://developer.vonage.com/en/blog/using-vonage-apis-with-mongodb-atlas-part-3)
4. [Part 4 - Using Atlas for User Authentication](https://developer.vonage.com/en/blog/using-vonage-apis-with-mongodb-atlas-part-4)
5. [Part 5 - Using Vonage In-App Messaging for Notifications](https://developer.vonage.com/en/blog/using-vonage-apis-with-mongodb-atlas-part-5)

In this blog series, Chris Tankersley demonstrates how MongoDB Atlas and Vonage APIs can be seamlessly integrated to build sophisticated applications. MongoDB Atlas, a cloud-based database service, alleviates the challenge of managing databases across multiple platforms such as AWS, Azure, and Google Cloud, enabling developers to focus on creating rich user experiences. Its built-in user authentication system adds an extra layer of security, ensuring the safety of user credentials.

On the other hand, Vonage APIs, used in conjunction with MongoDB Atlas, allow developers to establish secure, multifaceted communication with users. The Vonage Verify API, when tied with MongoDB's database, presents an added security measure via two-factor authentication. Further, the Messages API enhances customer engagement by enabling multi-channel communication including SMS, MMS, WhatsApp, Facebook Messenger, and Viber. When a situation requires direct customer interaction, the Vonage Meetings API can be employed to quickly establish video chats. With the help of MongoDB Atlas and the In-App Messaging API, the application can send real-time notifications to authenticated users. Through the example of a restaurant demo application, Chris shows the practical usage and combined strength of MongoDB Atlas and Vonage APIs.

# Join the Party

Spring was a fabulous season for our team, and we can't wait for you to see everything in store for summer. Until then, follow us on [Twitter](https://twitter.com/VonageDev) and join our [Community Slack](https://developer.vonage.com/community/slack) group. We love chatting with our developers and are always open to feedback. Cheers!