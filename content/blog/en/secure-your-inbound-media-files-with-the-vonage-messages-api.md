---
title: Secure your Inbound Media Files with the Vonage Messages API
description: An overview of the Vonage Messages API's Secure Inbound Media
  feature, with an explanation of how to enable and use it
author: karl-lingiah
published: true
published_at: 2023-03-27T16:00:09.913Z
updated_at: 2023-03-27T16:00:10.000Z
category: tutorial
tags:
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Vonage’s [Message API](https://developer.vonage.com/en/messages/overview) enables two-way conversations across a number of different messaging channels. Many of these channels support sending and receiving media files, such as images, audio, and video.

This multi-media messaging is extremely useful for a wide variety of situations. Examples might include a customer needing to send a business a video of a faulty product malfunctioning or images of the damage caused in transit to a delivery.

Inbound messages are received via an [Inbound Message Webhook](https://developer.vonage.com/en/api/messages-olympus#inbound-message). In order to receive inbound messages, you first need to set up this webhook. This can be done in the [Vonage Dashboard](https://dashboard.nexmo.com/), via the following steps:

* Create a Vonage Application (if you haven’t already done so)
* Within that application:
* * Enable it for the Messages API
  * Set the Inbound URL to whichever URL you wish to receive your inbound messages

![Vonage Dashboard Messages Application Inbound URL setting](/content/blog/secure-your-inbound-media-files-with-the-vonage-messages-api/messages-api-inbound-url.png "Vonage Dashboard Messages Application Inbound URL setting")

### Inbound Message Payload

Inbound messages include a JSON payload containing details of the message. For messages of type text, this will include the text of the message itself, for example:

```json
{
   "channel": "messenger",
   "message_uuid": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
   "to": "9876543210",
   "from": "0123456789",
   "timestamp": "2023-01-01T14:00:00.000Z",
   "message_type": "text",
   "text": "Hey there!"
}
```

For messages where the type is some sort of media, such as image or video, that media is stored on Vonage’s media servers, and the message includes a unique url to access the media file, for example: 

```json
{
   "channel": "messenger",
   "message_uuid": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
   "to": "9876543210",
   "from": "0123456789",
   "timestamp": "2023-01-01T14:00:00.000Z",
   "message_type": "image",
   "image": {
      "url": "https://api-us.nexmo.com/v3/media/1b456509-974c-458b-aafa-45fc48a4d976"
   }
}
```

### Standard Security

The media is stored for only 48 hours, and you need to issue a GET request to the specific URL provided in the inbound message payload in order to access the media file. For example:

```
GET /v3/media/1b456509-974c-458b-aafa-45fc48a4d976
Host: api-us.nexmo.com
```

The fact that you need the specific URL and that it expires after a set time provides a certain degree of inbuilt security. There may, however, be some minor risks associated with this level of security; for example, a brute force could reveal the URL, and anyone in possession of the URL could access the media file without needing any additional credentials.

### Enhanced Security with Secure Inbound Media

For certain use-cases, an increased level of security may be desirable, such as a customer-care situation where the customer needs to send sensitive information in image or video form. This is where the Messages API [Secure Inbound Media](https://developer.vonage.com/en/messages/concepts/secure-inbound-media) feature can help!

To enable the feature, in the Vonage Dashboard edit the Application where you set the Inbound URL for the webhook, and toggle the Enhanced Inbound Media Security switch to the ‘on’ position.

![Vonage Dashboard Messages Application Enhanced Inbound Media Toggle](/content/blog/secure-your-inbound-media-files-with-the-vonage-messages-api/enhanced-inbound-media-toggle.png "Vonage Dashboard Messages Application Enhanced Inbound Media Toggle")

With the feature enabled, inbound message webhooks will still contain a URL to access the media, but now the GET request to access the media must contain an Authorization header with a scheme of Bearer and the credentials being a [JSON Web Token](https://developer.vonage.com/en/getting-started/concepts/authentication#json-web-tokens) (JWT). The JWT must be generated using the Application ID and Private Key from the Vonage Application where you set the Inbound webhook URL. For example:

```
GET /v3/media/1b456509-974c-458b-aafa-45fc48a4d976
Host: api-us.nexmo.com
Authorization: Bearer eyJ0eXAiOiJKV1Qi...
```

With the feature enabled, making a GET request to the media URL without a correctly set Authorization header will result in an 401 Unauthorized response.

### Generating JWTs

You can generate a JWT via our [online tool](https://developer.vonage.com/en/jwt), or [by using the Vonage CLI](https://developer.vonage.com/en/getting-started/concepts/authentication#using-the-vonage-cli-to-generate-jwts). In the context of a server application, you can use one of our [Server SDKs](https://developer.vonage.com/en/tools) to generate the JWT as part of your overall application workflow. Below is an example of generating a JWT using the Ruby Server SDK.

```ruby
require 'vonage'

jwt = Vonage::JWT.generate(
 application_id: ENV['VONAGE_APPLICATION_ID'],
 private_key: ENV['VONAGE_PRIVATE_KEY_PATH']
)
```

(The example assumes that 'VONAGE_APPLICATION_ID' and 'VONAGE_PRIVATE_KEY_PATH' are set as environment variables in the Ruby application).

### Next Steps

Could Secure Inbound Media be useful for your app or use-case? Then why not get started now, with Vonage APIs! Check out the [documentation](https://developer.vonage.com/en/documentation). If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack), and we will get back to you.