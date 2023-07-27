---
title: Generate JWTs With Vonage’s Online JWT Generator
description: Unlock the full potential of Vonage's developer portal today with
  the Online JWT Generator. Seamlessly integrate JWT authentication into your
  Vonage applications, enhancing security and trust. Use JSON web tokens to
  secure your applications.
thumbnail: /content/blog/generate-jwts-with-vonage’s-online-jwt-generator/vonage-jwt-generator.png
author: benjamin-aronov
published: true
published_at: 2023-06-22T09:38:15.398Z
updated_at: 2023-06-22T09:38:15.459Z
category: tutorial
tags:
  - jwt
  - javascript
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Discover the untapped potential of the Vonage developer portal, a treasure trove of resources for developers. In addition to our documentation and insightful blog posts, the portal offers a range of powerful tools. One such gem is the [Online JWT Generator](https://developer.vonage.com/en/getting-started/tools/jwt-generator), enabling seamless creation of JSON Web Tokens ([JWTs](https://developer.vonage.com/en/getting-started/concepts/authentication#json-web-tokens)) for Vonage applications. Unleash the power of JWTs in your development workflow with ease!

### Introducing the Online JWT Generator

The online JWT Generator is an intuitive tool that empowers developers to instantly generate JWTs for Vonage applications, streamlining the setup process. This can be super useful when setting up Vonage applications. For instance, if you’re building with the Messages API, you can quickly test that you’ve set everything up properly just by sending SMS from your virtual number with a bash script like this:

```
curl -X POST https://api.nexmo.com/v1/messages \
  -H 'Authorization: Bearer '$JWT\
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d $'{
          "message_type": "text",
          "text": "Testing Vonage Messages API.",
          "to": "'$TO_NUMBER'",
          "from": "'$FROM_NUMBER'",
          "channel": "sms"
}'
```

But notice that we need to add a $JWT to this operation to let Vonage know that this is a legitimate request, i.e. authenticated.

### Effortless JWT Authentication

JWTs play a vital role in secure authentication for your Vonage applications. Learn best practices for secure authentication with JWTs in our [comprehensive guide](https://developer.vonage.com/en/getting-started/concepts/authentication#json-web-tokens).

## Creating a JWT

To create a JWT for your Vonage application, follow these simple steps:

### Step 1: Access Your Vonage Dashboard

Open your [Vonage Dashboard](https://dashboard.nexmo.com/applications) and navigate to the desired application. Once you’ve opened your application it should look something like this:

![Vonage application in the developer dashboard.](/content/blog/generate-jwts-with-vonage’s-online-jwt-generator/screenshot-2023-06-21-at-21.53.49.png "application-in-vonage-dashboard.png")

### Step 2: Retrieve the Application ID

Copy the APPLICATION ID from the dashboard and enter it into the generator:

![Vonage JWT Generator With Application ID](/content/blog/generate-jwts-with-vonage’s-online-jwt-generator/screenshot-2023-06-21-at-22.04.13.png "vonage-jwt-generator-with-application-id.png")

The little prompt is already telling us that we’re missing something: our private key! What is a private key? This a unique alphanumeric code that is used to encrypt access.

### Step 3: Obtain the Private Key

In order to get the private.key, we need to generate it. So click on “edit”. This will then open a page similar to this one. 

![Vonage application private key generation](/content/blog/generate-jwts-with-vonage’s-online-jwt-generator/screenshot-2023-06-21-at-22.14.17.png "vonage-application-private-key-generation.png")

Now under the authentication section, we can see that it talks about JWTs and using keys as signatures. We’ll want to click the button “Generate public and private key”. This will download a file called “private.key” to our computer. Be sure to save it securely where you will remember it. 

It’s very important that even though you’ve downloaded your private.key, you need to save the new status of the application! Scroll to the bottom and click save. Each time a new key is generated, the old key is no longer valid. So you must be using the current, valid key.

### Step 4: Open and Verify the Private Key

Open the private key file using a text editor (like VS Code or Sublime Text), ensuring no extraneous spaces or line breaks. The private key serves as a secure key to unlock access to your application's API endpoints. It should look something like this:

![Vonage Private Key Example](/content/blog/generate-jwts-with-vonage’s-online-jwt-generator/screenshot-2023-06-21-at-22.21.10.png "vonage-private-key-example.png")

### Step 5: Generate a JWT with the Generator

We can now add the private key to the generator:

![Vonage JWT Generator Complete Example](/content/blog/generate-jwts-with-vonage’s-online-jwt-generator/screenshot-2023-06-21-at-22.22.21.png "vonage-jwt-generator-complete-example.png")

And now the generator will create a JWT instantly!

## Final Thoughts

### Enhance JWT Validity and Verification

To validate the integrity of your JWT, you can use [jwt.io](http://jwt.io/). These tools offer comprehensive JWT analysis and debugging capabilities, ensuring your tokens are valid and secure.

#### Bonus: Set JWT Permissions

If you'd like to grant certain permissions for a user, you can set the ACL in the JWT Generator. For example, to allow a user to be able to create/manage conversations as well as send/receive texts. images, and audio, the ACL would look like this:

![ACL Options Example](/content/blog/generate-jwts-with-vonage’s-online-jwt-generator/image-45-.png "acl-options-example.png")

There are many options for your JWTs, which you can explore [here](https://developer.vonage.com/en/getting-started/concepts/authentication#vonage-client-sdks).

### Vonage’s Developer Community

Join the Vonage developer community on the [Vonage Community Slack](https://developer.vonage.com/en/community/slack). Collaborate with fellow developers, share insights, and exchange knowledge. Connect with us on [Twitter](https://twitter.com/VonageDev) for further assistance or inquiries.