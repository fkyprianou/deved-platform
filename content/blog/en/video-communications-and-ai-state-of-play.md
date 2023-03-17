---
title: Video Communications and AI - state of play
description: This article will consider what AI/ML tools and frameworks can be
  valuable to developers working with Vonage Video API. We will also consider a
  few sample integrations using standard tools.
author: oleksii-borysenko
published: true
published_at: 2023-03-06T13:11:10.203Z
updated_at: 2023-03-06T13:11:10.218Z
category: inspiration
tags:
  - Video
  - AI
  - API
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
In this article, we'll explore the AI/ML tools and frameworks that could benefit developers working with Vonage Video API. Additionally, we'll examine some sample integrations using standard tools.

Since the pandemic, many people have moved to home offices, and as of today, many IT companies continue to optimize their expenses by closing or selling office properties. Many employees continue to work from home. You can find more data in the [customer engagement report](https://www.vonage.com/resources/publications/global-customer-engagement-report/). This also increased the demand for new features that help with privacy and focus during video calls, like background blur, noise reduction, and speaker tracking. These features are beneficial when you work from home, in a coworking space, and occasionally, from the airport.

The pandemic has pushed us to rethink how we work, and video conferencing has played a significant role in that transformation. Video applications are driving tech evolution, and many AI solutions are now getting integrated into video conferencing.

## Simplifying work with Insertable Streams

Laying the foundation for AI media manipulation, we created the Media Processor - a feature that simplifies the use of webRTC Insertable Streams API for use with custom video transformers. The Media Processor lets you easily connect a series of transformers through which video frames will travel.
Background blur or noise cancellation are examples of possible transformers. 
Vonage Media Processor is an accelerator library for web developers who want to use insertable streams on Chrome-based browsers using any Vonage SDK (voice and video).

* **@vonage/media-processor** is the library which simplifies the use of Insertable Streams
* Media pipeline orchestration handled by the library 
  **@vonage/media-processor/MediaProcessor**
* Controls the transformers
* Exposes *setTransformers, setVideoTransformers, setAudioTransformers*
* Allows connecting series of transformers
  **@vonage/media-processor/MediaProcessorConnector**
* Connects MediaProcessor with the OT Publisher
  **Transformers**
* Class where transformation algorithms are implemented
* ML transformers provides reference implementation

![Web application scheme](/content/blog/video-communications-and-ai-state-of-play/web-application.png)

## Applying Vonage ML Transformers to video & audio streams

With the Media Processor as our foundation for working with insertable streams, we can now incorporate transformers that will facilitate the movement of video and audio frames. Vonage ML transformers is a library that implements machine learning algorithms for the web. This library is based on proprietary and open-source libraries, including [@vonage/media-processor](https://www.npmjs.com/package/@vonage/media-processor) for orchestration, and MediaPipe and TensorFlow Lite (TFLite) for the actual video enhancements. TensorFlow Lite is a mobile library for deploying models on mobile, microcontrollers, and other edge devices. The library allows users to run arbitrary TFLite models on the web by loading a TFLite model from a URL, setting the model's input data with TFJS tensors, running inference, and getting the output back in TFJS tensors. Additionally, it includes helper classes to aid with certain model types, such as Object Detection models.

MediaPipe library is an open source library under MIT license. For our solution of background blur/replacement we use the [Selfie Segmentation](https://google.github.io/mediapipe/solutions/selfie_segmentation.html) solution of MediaPipe. The library adds the support for all MediaPipe JS solutions. This helps developers create cool projects with any MediaPipe JS module.

![Transformers](/content/blog/video-communications-and-ai-state-of-play/media-flow.png)

Library from Vonage that provides ML algorithms:

* Based upon [MediaPipe](https://mediapipe.dev) and TFLite
* Access to all MediaPipe (js) algorithms 

  * Selfie Segmentation
  * Face Mesh
  * Iris Detection
  * Hand detection
  * Objectron
  * Holistic
  * Pose
* Provides implementations for visual effects:

  * Background Blur
  * Virtual Background
  * Silhouette Blur
  * Video Background

**Implementation details:​**

* Uses the [MediaPipe Selfie Segmentation](https://www.npmjs.com/package/@mediapipe/selfie_segmentation) solution.
* The process runs in a web worker.
* MediaPipe solutions are based on WebGL and wasm (SIMD).
* The solution does not come with MediaPipe binaries bundled. We added static assets under AWS Cloud Front CDN. Here are white-listed IPs for cloud front.
* MediaProcessorConfig allows you to define mediapipeBaseAssetsUri which allows the user to self-host MediaPipe assets. However, we do NOT recommend this.