---
title: Monitoring Application Health with Java and Vonage Messages API
description: Learn to build an application monitoring system for your Java
  application with Spring Boot Actuator and Vonage Messages API.
author: benjamin-aronov
published: true
published_at: 2023-02-16T11:32:07.206Z
updated_at: 2023-02-16T11:32:07.235Z
category: tutorial
tags:
  - java
  - springboot
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
# Monitoring Application Health: Java + Vonage Messages API

## Introduction

Generating and sending Actuator health reports is an important process that can help ensure the smooth operation of an application. 

Spring Boot is a popular framework for building Java applications that provides a set of tools for monitoring and managing an application. One feature is the Actuator, which provides a set of endpoints for monitoring the health of an application. These endpoints can be used to retrieve information about the application's performance, such as memory usage, CPU usage, and response times.

By using Vonage Messages API, we can send application health reports via texts, and in case of any critical health issues, we can send alternative messages as an additional alert to a designated phone number. In this article, we will discuss how to set up and configure Spring Boot to generate health reports for an application and Vonage Voice, SMS,  and Messages APIs to deliver them. 

## Spring Boot

Spring Boot is a Java-based framework that is used to create stand-alone, production-grade applications that are easy to build and run. It is built on top of the Spring Framework and uses its modules to provide a wide range of functionality to developers. Follow [this](https://spring.io/guides/gs/spring-boot/) article to set up a project in Spring Boot.

To enable Actuator in your Spring Boot application, you need to add the Spring Boot Actuator starter dependency to your project and configure it in the `application.properties` file of your project, as shown below.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Also add the following properties in your application.properties file:

```xml
management.endpoints.web.exposure.include=*
management.endpoints.web.base-path=/actuator
```

Other dependencies are `lombok` and `web`. Actuator provides a comprehensive set of features to help you monitor and manage your Spring Boot application, and it is an essential tool for any production-ready Spring Boot application.

## Spring Boot Actuator

The Actuator provides a number of useful features such as:

* Health check: Actuator provides a health check endpoint. This endpoint returns a JSON object that contains information about the health of the application, such as whether it is currently running or unavailable, and whether all its dependencies are available.
* Metrics: Actuator provides an endpoint to retrieve metrics, such as the number of requests, the number of errors, and the response time.
* Logging: Actuator provides an endpoint for retrieving log files, which can be useful for troubleshooting issues.
* Configuration: Actuator provides an endpoint for understanding how the application is configured and for troubleshooting issues.
* Thread dump: Actuator provides an endpoint for troubleshooting issues related to threading.
* Auditing: Actuator provides an endpoint to track the actions of users, such as the number of requests they have made.
* HTTP trace: Actuator provides an endpoint for retrieving a trace of all HTTP requests and responses, which can be useful for troubleshooting issues related to the HTTP layer of the application.

### Vonage Messages API

The Vonage Messages API is a set of RESTful web services that allow developers to send and receive messages, such as SMS and MMS, as well as rich messages services such as WhatsApp, Viber and Facebook Messenger. The API is designed to be simple to use and easy to integrate with other systems and services.

### Getting Started with Vonage
To get started working with the Vonage APIs, you'll need to sign up for a Vonage account and then create an application within that account. The application will be assigned an API key and secret, which you will use to authenticate your requests to the API.



<sign-up></sign-up>


## Application Health Report Generation and Sending - The Implementation

For the sake of simplicity, we are going to track our application for statuses where the server-side generates a 5xx error code for requests sent to it. According to HTTP Codes Standard, 5XX implies an `Internal Server Error`. This essentially means that our application might be having some challenges requiring attention. You can extend this to handle different response codes and diverse response codes, generated from your application in response to requests made to it.

If you would like to save responses from Actuator, you would need to create a model and a repository to do this. The model will carry fields that correspond to key-value pairs from the actuator JSON output. This is not covered in this article.

### Implementing the `HttpTraceRepository`

Create a class `RemoteRepository` that implements the "HttpTraceRepository" interface. The class is using the "Getter" and "Setter" annotations from the "lombok" library, which provide getter and setter methods for the class's fields without the developer having to explicitly write them.

The class has two methods that are required to be implemented by the HttpTraceRepository interface: findAll() and add(HttpTrace trace). The findAll() method returns an empty list of HttpTrace objects and the add(HttpTrace trace) method takes an object of type HttpTrace. This is where your logic comes in. In this case, it checks the response status code of that trace object coming from `actuator` and if it is equal to 500, it calls services to send SMS messages via Vonage API. You can also instruct it to send WhatsApp messages.

```java
import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.actuate.trace.http.HttpTrace;
import org.springframework.boot.actuate.trace.http.HttpTraceRepository;
import java.util.Collections;
import java.util.List;

@Getter
@Setter
public class RemoteRepository implements HttpTraceRepository {

   @Override
   public List<HttpTrace> findAll() {
       return Collections.emptyList();
   }

   @Override
   public void add(HttpTrace trace) {

       int responseStatusCode = trace.getResponse().getStatus();

       if (responseStatusCode == 500) {
           // call the services to send SMS and Messages via Vonage API
       }
   }
}
```

### The Actuator `Configuration` class

Next, create a configuration class for `actuator`. This configuration class, `ActuatorConfig`, configures certain aspects of Spring's Actuator feature.

* `@Configuration`: Indicates that this class contains one or more @Bean definitions for Spring's Application Context.
* `@ConditionalOnWebApplication`: This annotation is a Spring Boot feature, it indicates that this configuration should be applied only if the application is a web application.
* `@ConditionalOnProperty(prefix = "management.trace.http", name = "enabled", matchIfMissing = true)`: This annotation checks for the presence of a specific configuration property, in this case management.trace.http.enabled in the application's configuration. If the property is not present, it defaults to true.
* `@EnableConfigurationProperties(HttpTraceProperties.class)`: This annotation tells Spring to enable support for automatically injecting configuration properties from the application's configuration files into instances of the specified class (HttpTraceProperties)
* `@AutoConfigureBefore(HttpTraceAutoConfiguration.class)`: This annotation indicates that the ActuatorConfig class should be processed before HttpTraceAutoConfiguration.

The class also defines one method `traceRepository()` annotated with `@Bean` and `@ConditionalOnMissingBean(HttpTraceRepository.class)`, this means that it will only be executed if no other bean of type HttpTraceRepository has been defined in the application context. The method returns a new instance of `RemoteRepository` which is expected to be used as an `HttpTraceRepository`.

```java
import org.springframework.boot.actuate.trace.http.HttpTraceRepository;
import org.springframework.boot.actuate.autoconfigure.trace.http.HttpTraceProperties;
import org.springframework.boot.actuate.autoconfigure.trace.http.HttpTraceAutoConfiguration;
import org.springframework.boot.autoconfigure.AutoConfigureBefore;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConditionalOnWebApplication
@ConditionalOnProperty(prefix = "management.trace.http", name = "enabled", matchIfMissing = true)
@EnableConfigurationProperties(HttpTraceProperties.class)
@AutoConfigureBefore(HttpTraceAutoConfiguration.class)
public class ActuatorConfig {

	@Bean
	@ConditionalOnMissingBean(HttpTraceRepository.class)
	public RemoteRepository traceRepository() {
    	return new RemoteRepository();
	}
}
```

## Utility `Service` Classes for Vonage API Services

For each of the Vonage services to be worked with, a `service` class will be created. Essentially, one service class for each of our Messages Services (SMS, WhatsApp, Viber) and one Voice Service.

### `VoiceService` class

To use the Vonage Voice API to send a voice message, you will need to use the Vonage Java SDK. You can send voice messages to a phone number using Text-to-Speech (TTS) or by playing a pre-recorded audio file. Here is a demonstration of how to use the Vonage Java SDK to send a voice message:

First, you need to add the Vonage Java SDK as a dependency in your project. You can do this by adding the following code to your pom.xml file if you're using Maven:

```xml
<dependency>
 	<groupId>com.vonage</groupId>
    			<artifactId>client</artifactId>
    			<version>7.1.1</version>
</dependency>
```

Next, you'll need to create a new class that will handle sending the voice message. In this class, you'll need to import the com.vonage.client.voice.Call and com.vonage.client.VonageClient classes. In this class, you'll need to pass your Application ID and Private Key, to authenticate your application during calls. 

```java
String applicationId = "your-application-id";
String privateKeyPath = "path/to/private.key";
```

Next, create a new instance of the `VonageClient` class and pass in your Application ID and path to your Private Key.

```java
VonageClient client = VonageClient
.builder()
.applicationId(applicationId)
.privateKeyPath(privateKey)
.build();
```

Now, you can create a new instance of the Call class and set the from, to, and answer_url properties. The `from` property is the phone number or virtual number you're sending the message from, the `to` property is the number you're sending the message to, and the `answer_url` property is the URL that will be requested when the call is answered.

```java
Call call = new Call();
call.setFrom(FROM_NUMBER);
call.setTo(TO_NUMBER);
call.setAnswerUrl(ANSWER_URL);
```

For simplicity, create a class `VonageClientProvider` where you build an instance of your Vonage client - with your credentials. This is vital so that you do not have to repeat code in other classes.

```java
import com.vonage.client.VonageClient;

public class VonageClientProvider {

    private static final String APPLICATION_ID = "VONAGE_APPLICATION_ID";
    private static final String PRIVATE_KEY = "VONAGE_PRIVATE_KEY_PATH";
    private static VonageClient instance;

    private VonageClientProvider() {}

    public static VonageClient getInstance() {
        if (instance == null) {
            instance = VonageClient.builder()
                    .applicationId(APPLICATION_ID)
                    .privateKeyPath(PRIVATE_KEY)
                    .build();
        }

        return instance;
    }
}
```

### `messagesService` class

Here is a class in Java that demonstrates how to send an SMS, a WhatsApp message and a Viber message using the Vonage Messages API:

```java
import com.vonage.client.VonageClient;
import com.vonage.client.messages.MessageResponse;
import com.vonage.client.messages.MessagesClient;
import com.vonage.client.messages.sms.SmsTextRequest;
import com.vonage.client.messages.viber.Category;
import com.vonage.client.messages.viber.ViberTextRequest;
import com.vonage.client.messages.whatsapp.WhatsappTextRequest;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class MessagingService {

    private static final String VONAGE_NUMBER = "YOUR_VONAGE_NUMBER_HERE";

    public static void sendSms(String toNumber, String text) {

        VonageClient client = VonageClientProvider.getInstance();

        MessagesClient smsClient = client.getMessagesClient();

        var message = SmsTextRequest.builder()
                .from(VONAGE_NUMBER).to(toNumber)
                .text(text)
                .build();


            MessageResponse response = smsClient.sendMessage(message);
            log.info("Message sent successfully. ID: "+response.getMessageUuid());

    }

    public static void sendWhatsApp(String toNumber, String text) {

        VonageClient client = VonageClientProvider.getInstance();

        MessagesClient whatsAppClient = client.getMessagesClient();

        var message = WhatsappTextRequest.builder()
                .from(VONAGE_NUMBER).to(toNumber)
                .text(text)
                .build();

            MessageResponse response = whatsAppClient.sendMessage(message);
            log.info("Message sent successfully. ID: "+response.getMessageUuid());
    }
```

### `sendViber()` method

```java
public static void sendViber(String toNumber, String text) {

        VonageClient client = VonageClientProvider.getInstance();

        MessagesClient viberClient = client.getMessagesClient();

        var message = ViberTextRequest.builder()
                .from(VONAGE_NUMBER).to(toNumber)
                .text(text)
                .category(Category.TRANSACTION)
                .build();

            MessageResponse response = viberClient.sendMessage(message);
            log.info("Message sent successfully. ID: " + response.getMessageUuid());
    }
```

## Improving `RemoteRepository` class

All the services needed for sending messages via the Vonage API have now been created. Go into the `RemoteRepository` class and autowire each of these services, so that, when a “500 - Internal Server Error” is generated, each of these services are called and executed in whatever order and format you would like them to. The full structure of the `RemoteRepository` class is shown below:

```java
import com.vonage.tracer.service.MessagingService;
import com.vonage.tracer.service.VoiceService;
import lombok.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.trace.http.HttpTrace;
import org.springframework.boot.actuate.trace.http.HttpTraceRepository;

import java.util.Collections;
import java.util.List;

@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class RemoteRepository implements HttpTraceRepository {
    @Autowired
    private MessagingService messagingService;

    @Autowired
    private VoiceService voiceService;

    @Override
    public List<HttpTrace> findAll() {
        return Collections.emptyList();
    }

    @SneakyThrows
    @Override
    public void add(HttpTrace trace) {

        int responseStatusCode = trace.getResponse().getStatus();
        String errorMessage = "Check Server - 500 generated. Kindly check the server for an Internal Server Error";

        if (responseStatusCode == 500) {

            // call the services to send SMS, Messaging and Voice Messages via Vonage API

            MessagingService.sendSms(
                    "<sender-number-here",
                    "<recipient-number-here"
            );

            MessagingService.sendWhatsApp(
                    "<recipient-number-here", errorMessage
                    );

            MessagingService.sendViber(
                    "<recipient-number-here", errorMessage);

            voiceService.sendVoiceMessage(
                    "<recipient-here", errorMessage);

        }
    }
}
```

Now you can create your controllers and whenever any call is made to your application and it results in a response code of 500, the services will be called one after the other. You have the liberty of controlling how this works but this should get you on your way to great things.

## Conclusion

In this article, we looked at how to generate and send messages to respective addresses or locations when an HTTP Error 500 Code is created from our server. With this, you can check the content of the trace object from Actuator and see what other things you can do with it. You can also do a lot more with the Vonage API. Check the documentation here. The code for this article can be found [here](https://github.com/teevyne/spring-boot-actuator-vonage) also.

Come join the conversation on [Vonage Community Slack](https://developer.vonage.com/en/community/slack) or send a message on [Twitter](https://twitter.com/VonageDev).