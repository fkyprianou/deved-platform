---
title: How to Make a Text-to-Speech Phone Call in ASP.NET
description: Building high quality voice applications is easy with the Nexmo
  Voice API. In this tutorial, learn how to make a Text-to-Speech phone call
  with ASP.NET.
thumbnail: /content/blog/text-to-speech-phone-call-dr/text-to-speech-asp.png
author: bibi
published: true
published_at: 2017-07-28T13:13:24.000Z
updated_at: 2021-05-18T09:34:15.585Z
category: tutorial
tags:
  - voice-api
  - dotnet
comments: true
redirect: ""
canonical: ""
---
*This is the first tutorial on how to use Voice APIs with ASP.NET series.*

Building high quality voice applications is now made easy with The Nexmo Voice API. It allows you to make and receive phone calls with your ASP.NET applications among other things.

In this tutorial, we will see how to make a Text-to-Speech phone call with ASP.NET.

[View on GitHub](https://github.com/nexmo-community/nexmo-dotnet-quickstart/blob/ASPNET/NexmoDotNetQuickStarts/Controllers/VoiceController.cs)

The Nexmo Voice API makes it so easy to make a call. You send an authenticated call request to the API. Once it’s accepted, Nexmo makes a GET request to retrieve an NCCO (Nexmo Call Control Objects)and executes the actions within that NCCO.

![Flow of an outbound call](/content/blog/how-to-make-a-text-to-speech-phone-call-in-asp-net/call-outbound-diagram.gif)

## Prerequisites

* Visual Studio 2017
* A project set up for this tutorial series which you can find on [Github](https://github.com/nexmo-community/nexmo-dotnet-quickstart/tree/ASPNET/NexmoDotNetQuickStarts)
* Optional: [Nexmo CLI](https://github.com/Nexmo/nexmo-cli)

<sign-up number></sign-up>

## Configuration

In order to be able to use [The Nexmo Voice API](https://developer.nexmo.com/voice/overview), you'll have to create a voice application.

Before diving into the code, you need to follow the configuration steps details in the [“Nexmo Voice API with ASP.NET: Before you start”](https://learn.vonage.com/blog/2017/07/28/nexmo-voice-api-asp-net-configure-dr) post.

Once the configuration is done successfully, you are ready to make a Text-to-Speech phone call with The Nexmo Voice API! 

## Making a Text-to-Speech call with The Nexmo voice API

Congratulations! You made it so far which means you managed to configure your voice application and you created an ASP.NET project. 

In **appsettings.json** file, make sure to initialize Nexmo with your API credentials as well as the app ID and private key you just created.

```json
{
 "appSettings": 
 {
  "Nexmo.UserAgent": "NEXMOQUICKSTART/1.0",
  "Nexmo.Url.Rest": "https://rest.nexmo.com",
  "Nexmo.Url.Api": "https://api.nexmo.com",
  "Nexmo.api_key": "NEXMO-API-KEY",
  "Nexmo.api_secret": "NEXMO-API-SECRET",
  "NEXMOFROMNUMBER": "NEXMO-VIRTUAL-NUMBER",
  "Nexmo.Application.Id": "NEXMO-APP-ID",
  "Nexmo.Application.Key": "PATH:\TO\private.key"
 }
}
```

Then create [a new controller called VoiceController.cs](https://github.com/nexmo-community/nexmo-dotnet-quickstart/blob/b9b3ba0dcf7a2e35d8b14b06680e89ab989c0d88/NexmoDotNetQuickStarts/Controllers/VoiceController.cs) in which create [an action method called MakeCall](https://github.com/nexmo-community/nexmo-dotnet-quickstart/blob/b9b3ba0dcf7a2e35d8b14b06680e89ab989c0d88/NexmoDotNetQuickStarts/Controllers/VoiceController.cs).

Above the method, add a <strong>HttpGetAttribute</strong> to allow the user to navigate to the corresponding view.

```
[HttpGet]
public ActionResult MakeCall()
{
   Return View();
}
```

Afterwards, click on the Views folder and add a new folder called *Voice*. Within this folder, create a new view ([`MakeCall.cshtml’](https://github.com/nexmo-community/nexmo-dotnet-quickstart/blob/b9b3ba0dcf7a2e35d8b14b06680e89ab989c0d88/NexmoDotNetQuickStarts/Views/Voice/MakeCall.cshtml)). Then, add a form to the view with an input tag (type = “text”) for the destination number. Lastly, add an input tag (type = “submit”) to submit the form.
Back to the VoiceController, add the following using statement to the top of the file.

```
using Nexmo.Api
```

Add another [action method named MakeCall](https://github.com/nexmo-community/nexmo-dotnet-quickstart/blob/b9b3ba0dcf7a2e35d8b14b06680e89ab989c0d88/NexmoDotNetQuickStarts/Controllers/VoiceController.cs#L20-L49) with a string parameter: to.

Within this method, you will make a call using the parameter as the to. The from number is your Nexmo virtual number (retrieved from the appsettings.json), the answer_url is [the NCCO](https://developer.nexmo.com/api/voice/ncco) ( Nexmo Call Control Object) which is a JSON array used to control the flow of your call. 

You can use one of the [Nexmo’s community NCCO examples](https://github.com/nexmo-community/ncco-examples/) or create your own and host it somewhere reachable by The Nexmo Voice API. An easy hosting solution is [GitHub Gist](https://gist.github.com/). 

Since we are trying to make a Text-to-Speech call in this demo, the action required for the NCCO is *[Talk](https://developer.nexmo.com/api/voice/ncco#talk)* , you also need to provide the text to be synthesised into speech in the call and the last thing to provide is a [voice name](https://developer.nexmo.com/api/voice/ncco#voice-names). Nexmo provides a list a voiceNames in different languages and accents for both genders.

In my NCCO below, I chose the voiceName to be “Amy” which is a female British-english voice. 

```json
[
  {
    "action": "talk",
    "voiceName": "Amy",
    "text": "Hi, this is Amy. You are listening to a Call made with the Nexmo Voice API"
  }
]
```

Now back to MakeCall method, make sure <strong>NEXMO_CALL_ANSWER_URL</strong> points to your NCCO json file.

```
[HttpPost]
public ActionResult MakeCall(string to)
{
    var NEXMO_FROM_NUMBER =    Configuration.Instance.Settings["appsettings:NEXMO_FROM_NUMBER"];
    var NEXMO_TO_NUMBER = to;
    var NEXMO_CALL_ANSWER_URL = "https://nexmo-community.github.io/ncco-examples/first_call_talk.json";

    var results = Call.Do(new Call.CallCommand
    {
        to = new[]
        {
                    new Call.Endpoint {
                        type = "phone",
                        number = NEXMO_TO_NUMBER
                    }
                },
        from = new Call.Endpoint
        {
            type = "phone",
            number = NEXMO_FROM_NUMBER
        },
        answer_url = new[]
        {
                    NEXMO_CALL_ANSWER_URL
                }
    });
    var result = new HttpStatusCodeResult(200);

    return RedirectToAction("Index", "Voice");
}
```

Now, let's run the app and make a Text-to-Speech phone call.

When it is successful, it retrieves the [NCCO](https://developer.nexmo.com/voice/guides/ncco-reference) from your webhook, executes the actions, and terminates the call.

You may observe some delay before your phone rings depending on your  phone carrier.

## Learn more

### API References and Tools

* [Application API](https://developer.nexmo.com/concepts/guides/applications)
* [Voice API](https://developer.nexmo.com/voice/overview)
* [Nexmo REST client for .NET](https://github.com/Nexmo/nexmo-dotnet)

### Nexmo Getting Started Guide for ASP.NET

* [How to Send SMS Messages with ASP.NET MVC Framework](https://learn.vonage.com/blog/2017/03/23/send-sms-messages-asp-net-mvc-framework-dr/)
* [How to Receive SMS Messages with ASP.NET MVC Framework](https://learn.vonage.com/blog/2017/03/31/recieve-sms-messages-with-asp-net-mvc-framework-dr/)
* [How to Get an SMS Delivery Receipt in ASP .NET MVC](https://learn.vonage.com/blog/2017/07/21/get-sms-delivery-receipt-asp-net-mvc-dr/)