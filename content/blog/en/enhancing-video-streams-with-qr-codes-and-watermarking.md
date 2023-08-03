---
title: "Video + AI: Enhancing Video Streams with QR Codes and Watermarking"
description: Explore how to enhance your video streams using QR Codes and
  watermarks with the Vonage Media Processor API. Discover real-world use cases,
  technical implementation, and get a detailed walkthrough of the code.
author: enrico-portolan
published: true
published_at: 2023-07-11T09:01:14.340Z
updated_at: 2023-07-11T09:01:14.364Z
category: tutorial
tags:
  - video-api
  - javascript
comments: true
spotlight: true
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

In today's digital era, enhancing video streams has become an increasingly popular practice, be it for branding, advertising, or providing extra information in a non-intrusive way. This blog post aims to walk you through how to add QR Codes and watermarks to a video stream using the Vonage Video API. 

In order to accomplish this, we'll leverage the capabilities of the Media Processor API. This innovative API provides the functionality to personalize both audio and video streams by integrating transformers into the stream. A detailed exploration of the API and its potential uses will follow later in this blog post.

**tl;dr:** If you would like to skip ahead and get right to deploying it, you can find all the code for the app on [GitHub](https://github.com/Vonage-Community/blog-video_api-javascript_enhancing_video_streams_with_qr_codes_and_watermarking)

## Project Setup

1. Install Node and npm
2. Copy the [GitHub repository](https://github.com/Vonage-Community/blog-video_api-javascript_enhancing_video_streams_with_qr_codes_and_watermarking)
3. Run `npm install`
4. Run `npm run serve`

## Media Processors API

The Vonage Media Processor library, available via npm, is a powerful tool that simplifies the use of the Insertable Streams API for transforming video and audio tracks. For the purpose of this blog post, we will use only the video methods.

The library introduces two primary classes:

* MediaProcessor: This class manages transformers and comes with a `setTransformers()` method to set up the transformers.
* MediaProcessorConnector: This class requires a MediaProcessor object for its constructor and is responsible for handling input and output tracks. While the implementation of the tracks is taken care of by OpenTok.js, you only need to supply the instance of MediaProcessorConnector.
  You can find the relevant documentation at https://tokbox.com/developer/guides/vonage-media-processor/js/

Let’s now dive into the creation of the video processors for the QR Code and Watermarking feature. 

## Using QR Codes

QR Codes have found numerous uses in real-world scenarios. They can hold data like URLs, text, etc., which can be scanned using a smartphone camera. In a video stream, QR Codes can be used for various reasons like redirecting the user to a specific URL, displaying user-specific text, and more. 

Imagine hosting a webinar where you want to share resourceful links with your viewers. By integrating a QR code into your live stream, you can effortlessly direct your audience to these links, providing them with an easy, real-time method of accessing relevant content.

### Technical Implementation

We're generating QR Codes using a custom transformer. To generate the QR Code, we are using a 3rd party library called [QrCode](https://github.com/davidshimjs/qrcodejs). 
The URL for the QR Code is taken as user input. We use the `qrCode` transformer from our `transformers` directory to create and position the QR Code on the video stream. You can customize the position and size of the QR Code.

```javascript
// file add-qr-code.js

_writeQrCode() {
    const qrContainer = document.createElement("div");
    qrContainer.id = "qr-container";
    new QRCode(qrContainer, {
      text: this.text,
      width: this.width,
      height: this.height,
      x: this.x,
      y: this.y,
    });
    return qrContainer;
  }

async transform(frame, controller) {
    this.canvas_.width = frame.displayWidth;
    this.canvas_.height = frame.displayHeight;
    const timestamp = frame.timestamp;

    this.ctx_.drawImage(frame, 0, 0);
    const imageData = this.ctx_.getImageData(
      0,
      0,
      this.canvas_.width,
      this.canvas_.height
    );
    frame.close();
    const qrCode = this._writeQrCode(imageData);
    this.ctx_.drawImage(
      qrCode.querySelector("canvas"),
      this.x,
      this.y,
      this.width,
      this.height
    );
    controller.enqueue(
      new VideoFrame(this.canvas_, { timestamp, alpha: "discard" })
    );
  }
```

From the user interface, you can set the URL for the QR code, the size, and the position of the QR code in the video stream. 

![QR code generator transformer](/content/blog/enhancing-video-streams-with-qr-codes-and-watermarking/qr-code-generator.png "QR code transformer")

## Using Watermarks

Watermarks have been commonly used for branding and protecting digital content. In the context of video streaming, a watermark could serve multiple purposes: it could be a company logo for branding, a copyright symbol for content protection, or any other image for that matter.

### Technical Implementation

For watermarking, we accept an image file from the user as input. The selected image is then flipped horizontally to counteract the mirroring effect of the video stream. The `watermark` transformer from our `transformers` directory then positions the watermark on the video stream. You can customize the position of the watermark.

```javascript
_computePosition(frame) {
    if (this._position === "top-left") {
      return { x: frame.displayWidth - 150, y: 0 };
    } else if (this._position === "top-right") {
      return { x: 0, y: 0 };
    } else if (this._position === "bottom-left") {
      return { x: frame.displayWidth - 150, y: frame.displayHeight - 150 };
    } else if (this._position === "bottom-right") {
      return { x: 0, y: frame.displayHeight - 150 };
    }
    return { x: 0, y: 0 };
  }

  async transform(frame, controller) {
    this.canvas_.width = frame.displayWidth;
    this.canvas_.height = frame.displayHeight;
    const timestamp = frame.timestamp;

    this.ctx_.drawImage(frame, 0, 0, frame.displayWidth, frame.displayHeight);

    const { x, y } = this._computePosition(frame);
    this.ctx_.drawImage(this._image, x, y, 100, 80);
    frame.close();
    controller.enqueue(
      new VideoFrame(this.canvas_, { timestamp, alpha: "discard" })
    );
  }
```

![Watermark transformer form](/content/blog/enhancing-video-streams-with-qr-codes-and-watermarking/watermark.png "Watermark transformer")

## Code Walkthrough

On the main page, we start by importing the required packages and initializing the video stream publisher. We have two forms, one for each functionality (QR Code and Watermark). The appropriate form is displayed based on the user's selection.

In the `applyQrCode` function, we retrieve user inputs for the QR Code (URL, size, and position), set up the QR Code transformer, and apply it to the video stream.

Similarly, in the `applyWatermark` function, we take the uploaded image, adjust it as needed (flip horizontally), set up the watermark transformer, and apply it to the video stream.

The 'Apply' button calls the appropriate function based on the selected option, and the 'Clear' button can be used to remove all transformations from the video stream.

![QR code generator example](/content/blog/enhancing-video-streams-with-qr-codes-and-watermarking/qrcodeexample.png "QR code generator example")

![Watermark generator example](/content/blog/enhancing-video-streams-with-qr-codes-and-watermarking/watermark-example.png "Watermark generator example")

## Conclusion

Enhancing video streams by adding QR Codes or watermarking is a powerful way to increase interactivity, add value, and protect content. With the Vonage Video API and a little bit of JavaScript, it's fairly straightforward to add these features to a video stream. I hope you found this blog post helpful, and I encourage you to experiment with different ways of using video stream transformations to enhance your projects.

You can find the [GitHub code repo here](https://github.com/Vonage-Community/blog-video_api-javascript_enhancing_video_streams_with_qr_codes_and_watermarking). If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send us a Tweet on [Twitter](https://twitter.com/VonageDev), and we will get back to you!