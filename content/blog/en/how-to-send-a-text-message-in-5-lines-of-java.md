---
title: How to Send a Text Message in 5 Lines of Java
description: This article will show you just how easily you can send a text
  message using Vonage's Java SDK!
thumbnail: /content/blog/how-to-send-a-text-message-in-5-lines-of-java/text-message_5lines_java.png
author: sina-madani
published: true
published_at: 2023-08-03T13:46:10.855Z
updated_at: 2023-08-03T13:46:10.869Z
category: tutorial
tags:
  - messages-api
  - java
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

If you are a Java developer, you may have heard about [JEP 445: Unnamed Classes and Instance Main Methods](https://openjdk.org/jeps/445). This preview feature introduced in Java 21 greatly reduces the boilerplate code required to write a simple "Hello, World!" program. Although most professional developers rarely need to write a single-method application, it can sometimes be useful for creating reproducible minimal examples or to test that your environment is set up correctly. In this article, I will demonstrate this new syntax by showing you how to send a text message in just five lines of code using our [Messages API](https://developer.vonage.com/en/messages/overview).

## Pre-requisites

* Download and install Java 21. You can find [early access builds](https://jdk.java.net/21/) before it is GA. If you are reading this after September 2023, then you can find builds of JDK 21 from several places, for example, [Eclipse Temurin](https://adoptium.net/en-GB/temurin/releases/) or [Oracle](https://jdk.java.net/).
* Open a terminal and navigate to / create a suitable working directory. For example, `cd ~/Projects && mkdir vonage-java-demo`.
* Set JDK 21 as your Java version. The way to do this will depend on your operating system, but it should, in theory, be no more involved than updating your `JAVA_HOME` environment variable to point to the JDK you downloaded in Step 1. You can check which JDK you are using by typing `java --version` in your terminal.
* [Sign up for a Vonage account](https://ui.idp.vonage.com/ui/auth/registration).
* [Download the Java SDK with all of its dependencies](https://repo1.maven.org/maven2/com/vonage/client/7.6.0/client-7.6.0-all.jar) and save it in your working directory. Replace the version number provided in the link with the latest version, which can be found [on the GitHub Releases page](https://github.com/Vonage/vonage-java-sdk/releases).

## The Code

Below is the code to send an SMS. Create a Java file in your working directory - e.g., `VonageTextDemo.java`. Then paste the code from below into the file and save it. Replace the number in `to` with your phone number, `apiKey` with your API key, and `apiSecret` with your secret (available from [the developer dashboard](https://dashboard.nexmo.com)).

```java
import com.vonage.client.*; import com.vonage.client.messages.*; import com.vonage.client.messages.sms.*;
void main() {
    var client = VonageClient.builder().apiKey("a1b2c3d4").apiSecret("0123456789Abcdef").build();
    client.getMessagesClient().sendMessage(SmsTextRequest.builder().from("Vonage Java").to("447418360119").text("Hello, World!").build());
}
```

Here’s how the code works.

* Line 1 imports the classes from the packages required for compilation.
* Line 2 is the new syntax introduced in Java 21. Although omitting a package declaration has always been possible, up until Java 21, the main method needed to be contained within a class and had to be declared exactly as `public static void main(String[] args)` (or even `public static void main(String… args)`). You can learn more about the new syntax from [Inside Java Newscast #49](https://www.youtube.com/watch?v=P9JPUbG5npI).
* Line 3 builds the Vonage client using your API key and secret as the authentication method. This is the simplest way to get started with using the Messages API, but we recommend that you use an application ID and private key in a production environment. 
* Line 4 is where the message is both constructed and sent. The `com.vonage.client.messages.MessagesClient#sendMessage(MessageRequest)` method is being invoked with an `SmsTextRequest` - a subclass of `MessageRequest`. All message requests follow the Builder pattern (you can read more about this in [one of my previous blog posts](https://developer.vonage.com/en/blog/how-an-sdk-can-add-value-to-rest-apis)]. This allows for declarative construction of the message payload. Finally, for completeness.
* Line 5 is, of course, the closing brace for the main method declared on line 2.

If you want to send a text over WhatsApp, replace `SmsTextRequest` with `WhatsappTextRequest` and update the import. Note that you will need to use the [Messages API Sandbox](https://developer.vonage.com/en/messages/concepts/messages-api-sandbox) for sending a message from WhatsApp; otherwise, you will need to set up a WhatsApp Business Account and use that number. The Java SDK provides the `useSandboxEndpoint()` method on `MessagesClient` for convenience.

```java
import com.vonage.client.*; import com.vonage.client.messages.*; import com.vonage.client.messages.whatsapp.*;
void main() {
    var client = VonageClient.builder().apiKey("a1b2c3d4").apiSecret("0123456789Abcdef").build();
    client.getMessagesClient().useSandboxEndpoint().sendMessage(WhatsappTextRequest.builder().from("14157386102").to("447418360119").text("Hello, World!").build());
}
```

Similarly, you can send a text over Viber like so:

```java
import com.vonage.client.*; import com.vonage.client.messages.*; import com.vonage.client.messages.viber.*;
void main() {
    var client = VonageClient.builder().apiKey("a1b2c3d4").apiSecret("0123456789Abcdef").build();
    client.getMessagesClient().useSandboxEndpoint().sendMessage(ViberTextRequest.builder().from("My Company").to("447418360119").text("Hello, World!").build());
}
```

And here's the same for Facebook Messenger - use your Recipient ID in `to` and the Messages Sandbox ID in `from`:

```java
import com.vonage.client.*; import com.vonage.client.messages.*; import com.vonage.client.messages.messenger.*;
void main() {
    var client = VonageClient.builder().apiKey("a1b2c3d4").apiSecret("0123456789Abcdef").build();
    client.getMessagesClient().useSandboxEndpoint().sendMessage(MessengerTextRequest.builder().from("107083064136738").to("6573130892744564").text("Hello, World!").build());
}
```

## Compile & Run

In your working directory, type the following command:

```sh
javac --enable-preview --release 21 -cp "client-7.6.0-all.jar" VonageTextDemo.java
```

Of course, replace `client-7.6.0-all.jar` with the name of the JAR file you downloaded, same with “VonageTextDemo” - this should be whatever you called the file you created earlier. If compilation is successful, you should see a `.class` file generated in your working directory, e.g., `VonageTextDemo.class`. Now you can run it as follows:

```sh
java --enable-preview -cp "client-7.6.0-all.jar:." VonageTextDemo
```

If you now check your phone, you should have a text!

You can check out more examples in [our Java code snippets repository](https://github.com/Vonage/vonage-java-code-snippets/tree/main/src/main/java/com/vonage/quickstart/messages/sandbox). The Messages API supports not only text messages, but also multimedia such as files, images, audio, video and more.

## Signing off

That's all for now! If you come across any issues or have suggestions for enhancements, feel free to [raise an issue on GitHub](https://github.com/Vonage/vonage-java-sdk/issues), reach out to us on [Twitter](https://twitter.com/VonageDev) or drop by our [Community Slack](https://developer.vonage.com/community/slack) if you have feedback or just want to say hello. I hope that you have a great experience using Vonage APIs in Java!