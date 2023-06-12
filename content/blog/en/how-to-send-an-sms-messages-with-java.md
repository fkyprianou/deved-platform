---
title: How to Send an SMS Messages with Java
description: This blog covers how to send an SMS message with Java.
author: diana-pham-1
published: true
published_at: 2023-06-06T21:23:49.560Z
updated_at: 2023-06-06T21:23:49.573Z
category: tutorial
tags:
  - java
  - sms-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
The [Vonage SMS API](https://developer.vonage.com/en/messaging/sms/overview) is a service that allows you to send and receive SMS messages anywhere in the world. Vonage provides REST APIs, but it's much easier to use the Java SDK that we've written for you.

In this tutorial, we'll show you how to send SMS messages with Java.

## Prerequisites

Before we dive in, make sure you have the following:

* A [Vonage API account](https://developer.vonage.com/sign-up). Access your Vonage API Dashboard to locate your API Key and API Secret, which can be found at the top of the page.
* A Vonage phone number. To purchase one, go to Numbers > Buy Numbers and search for one that meets your needs.
* [JDK](https://www.oracle.com/java/technologies/downloads/) (17 or higher) or its open-source equivalent OpenJDK
* [Gradle](https://gradle.org/) (version 3.4 or newer) 

If you need some help getting started, don't worry! We've got you covered. You can find Mark Smith's [source code](<>) for this tutorial on GitHub.

## Using the Vonage SDK

To begin, set up your Gradle project and download the [Vonage Java SDK](https://github.com/Vonage/vonage-java-sdk).

Create a directory for your project. Go inside your directory and run `gradle init`. If you're unfamiliar with Gradle, fret not - we won't be delving into anything overly complex!

Select *1: basic* as the type of project to generate.
*1: Groovy*  as build script DSL.
Name your project—or press Enter for the default option.

Once that's complete, open the build.gradle file and replace its contents with the following code snippet:

```java
    // We're creating a Java Application:
    plugins {
        id 'application'
        id 'java'
    }
    
    // Download dependencies from Maven Central:
    repositories {
        mavenCentral()
    }
    
    // Install the Vonage Java SDK
    dependencies {
        implementation 'com.vonage:client:7.2.0'
    }
    
    // We'll create this class to contain our code:
    application {
        mainClass = 'getstarted.SendSMS'
    }
```

Next, open your console in the directory that contains your build.gradle file and run:

`gradle build`

This command will download the Vonage Java SDK and store it for later. If you had any source code, it would also compile that—but you haven't written any yet. Let's fix that!

Because of the  `mainClass`  we set in the Gradle build file, you're going to need to create a class called  `SendSMS`  in the package  `getstarted`. In production code, you'd want the package to be something like  `com.mycoolcompany.smstool`, but this isn't production code, so  `getstarted`  will do.

Gradle uses the same directory structure as Maven, so you need to create the following directory structure inside your project directory:  `src/main/java/getstarted`.

On macOS and Linux, you can create this path by running:

`mkdir -p src/main/java/getstarted`

Inside the `getstarted` directory, create a file called `SendSMS.java`. Once you have it opened in your favorite IDE, add this code:

```java
package getstarted;

import com.vonage.client.VonageClient;
import com.vonage.client.sms.MessageStatus;
import com.vonage.client.sms.SmsSubmissionResponse;
import com.vonage.client.sms.messages.TextMessage;

static String API_KEY = "YOUR_API_KEY";
static String API_SECRET = "YOUR_API_SECRET";

public class SendSMS {

    public static void main(String[] args) throws Exception {
    // Our code will go here!
    }
}

```

What this does is import all necessary parts of the Vonage SDK and create a method for our code.

We then store our API Key and API secret as variables. Fill in `YOUR_API_KEY` and `YOUR_API_SECRET` with the values you copied from the [Vonage API Dashboard](https://dashboard.nexmo.com/).

If you run `gradle run`, it should run your main method. However, since we don't have anything there yet, it won't do anything. Let's change that!

## Send SMS Message

Add this snippet to your `main` method:

```java
    VonageClient client = VonageClient.builder()
    
    .apiKey(YOUR_API_KEY)
    .apiSecret(YOUR_API_SECRET)
    .build();
```

The following code creates a `VonageClient` object that can be used to send SMS messages. Now that your client object is configured, you can send an SMS message:

```java

    TextMessage message = new TextMessage(VONAGE_BRAND_NAME,
                    TO_NUMBER,
                    "A text message sent using the Vonage SMS API"
            );
    
            SmsSubmissionResponse response = client.getSmsClient().submitMessage(message);
    
            if (response.getMessages().get(0).getStatus() == MessageStatus.OK) {
                System.out.println("Message sent successfully. " + response.getMessages());
            } else {
                System.out.println("Message failed with error: " + response.getMessages().get(0).getErrorText());
            }
```

Again, you'll want to replace `VONAGE_BRAND_NAME` with the virtual number you purchased and `TO_NUMBER` with your own mobile phone number. They should both be written as strings. Make sure to provide the `TO_NUMBER` in [E.164 format](https://developer.vonage.com/en/voice/voice-api/guides/numbers)—for example, 447401234567.

Finally, save and run `gradle run`, and you should see something like this:

`Message sent successfully.[com.vonage.client.sms.SmsSubmissionResponseMessage@f0f0675[to=447401234567,id=13000001CA6CCC59,status=OK,remainingBalance=27.16903818,messagePrice=0.03330000,network=23420,errorText=<null>,clientRef=<null>]]`

...and a text message sent to your phone!

If it didn't work, there may be some text after `ERR:`. Correct the issue and try again.

- - -

**NOTE**

In some countries (US), VONAGE_BRAND_NAME has to be one of your Vonage virtual numbers. In other countries (UK), you're free to pick an alphanumeric string value—for example, your brand name like AcmeInc. Read about country-specific SMS features on the [dev portal](https://developer.vonage.com/en/messaging/sms/guides/country-specific-features).

- - -

## Conclusion

You now know how to send an SMS in Java using Vonage's API! For further information, please refer to the documentation links provided below.

## Further Reading

* [General information about the SMS API](https://www.vonage.com/communications-apis/verify
* [SMS API Documentation](https://developer.vonage.com/en/api/sms)
* [Send an SMS Documentation](https://developer.vonage.com/en/messaging/sms/code-snippets/send-an-sms#further-reading)

## Stay in Touch

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/dianasoyster). Thanks again for reading, and I will catch you on the next one!