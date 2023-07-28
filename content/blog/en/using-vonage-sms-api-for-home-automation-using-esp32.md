---
title: Using Vonage SMS API for Home Automation using ESP32
description: Learn how to use the Vonage SMS for home automation using the ESP32
  microcontroller.
author: michael-crump
published: true
published_at: 2023-07-28T16:54:53.590Z
updated_at: 2023-07-28T16:54:53.604Z
category: tutorial
tags:
  - sms-api
  - iot
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
# Introduction

The Internet of Things (IoT) has become increasingly popular in recent years, with intelligent devices allowing individuals to remotely control various aspects of their homes. Vonage is a cloud communication platform enabling businesses to connect with customers via SMS, voice, and messaging apps.

This article will explore how to use Vonage SMS for home automation using the ESP32 microcontroller. We will walk you through the steps to set up the system and explain how it works. The goal is to control home devices using ESP32 with SMS' sent to Vonage via your phone.

## Prerequisites

* A [Vonage developer account](https://developer.vonage.com/sign-up)
* The [Arduino IDE](https://www.arduino.cc/en/software/)
* A microcontroller from the ESP32 family (ESP32-S2, ESP32-C3, or ESP32-S3) to test the sample.

## Technical Overview of the App

The Vonage SMS API will communicate with the ESP32 using a webhook. When Vonage receives an SMS, Vonage will send a POST request to the ESP32's endpoint via the webhook.

The ESP32 will then parse the JSON payload, perform a `digitalWrite()` function, and return a POST request to the Vonage SMS API with the state of operation (success/failure), letting the end-user know what happened.

![A diagram showing the process: Vonage SMS API sends a request to Ngrok - Ngrok sends it to ESP32, and ESP32 sends it back to Vonage](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/application-overview-diagram.png "application-overview-diagram.png")

## Step 1: How to Setup ESP32 Request Handler

To get started, you must install the `ArudinoJson` and `ESPAsyncWebSrv` libraries. These libraries will allow you to handle incoming requests from the Vonage SMS API.

1. Open the Arduino IDE
2. Open the `Library Manager` from the left panel.
3. Search for `ArduinoJson`

![Showing ArduinoJson in the Arduino IDE](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/arduinojson-arduino-ide.png "arduinojson-arduino-ide.png")

4. Press `Install`
5. Search for `ESPAsyncWebSrv` 

![Showing ESPAsyncWebSrv in the Arduino IDE](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/espasyncwebsrv-arduino-ide.png "espasyncwebsrv-arduino-ide.png")

6. Press `Install`

Once you have installed the libraries, you can set up the WiFi and the request handler. The request handler will handle incoming requests and route them to the appropriate function.

Create a new file (sketch) by going to `File > New Sketch`. Then, include all the libraries below.

```cpp
#include "AsyncJson.h"
#include "ArduinoJson.h"
#include "ESPAsyncWebSrv.h"
```

Next, initialize the web server.

```cpp
AsyncWebServer server(80);
```

Add WiFi credentials.

```cpp
const char *ssid = "Your WiFi name here";
const char *password = "Your WiFi password here";
```

Add a fallback function to handle [404](https://en.wikipedia.org/wiki/HTTP_404) requests.

```cpp
void onRequest(AsyncWebServerRequest *request) {
    request->send(404);
}
```

Now, add a `setup()` function to handle all the logic, starting with the WiFi setup.

```cpp
void setup() {
    // Start a serial connection at a 115200 baud rate
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);
    if (WiFi.waitForConnectResult() != WL_CONNECTED) {
        Serial.printf("WiFi Failed!\n");
        return;
    }
    Serial.println(WiFi.localIP());
    WiFiClient client;
    HTTPClient http;
}
```

Then, inside the `setup()` function, you will create an endpoint to handle the incoming JSON from Vonage.

```cpp
AsyncCallbackJsonWebHandler *handler = new AsyncCallbackJsonWebHandler("/inbound", [](AsyncWebServerRequest *request, JsonVariant &json) {
```

Start creating a dynamic JSON document inside the `*handler`.

```cpp
DynamicJsonDocument doc(2048)
```

Now, check for deserialization errors. Those shouldn't happen, but we must handle them so ESP32's cores don't panic.

```cpp
DeserializationError error = deserializeJson(doc, json); if (error) {
    // Handle the error
    Serial.println(error.c_str()); request->send(400, "text/plain", "Invalid JSON payload"); return;
}
```

Continue with creating a JSON object.

```cpp
JsonObject jsonObj = doc.as();
```

To keep things simple, we will only parse the `text` key from the payload, indicating what to do with the device we'll be controlling.

```cpp
String text = jsonObj["text"].as();
```

That's it for the JSON handling part. To control a digital pin, add some conditional logic that will send a request to Vonage depending on the text content. For example, if the user sends “ON” the request handler will send the “Turned ON!” message to Vonage.

A digital pin on the ESP32 is a pin that can be set to one of two states: HIGH (1) or LOW (0). It can be used for various purposes, such as reading the state of a sensor (digital input) or controlling the state of an actuator (digital output).  

```cpp
if (text == "ON") {
    digitalWrite(controlPin, HIGH);
    request->send(200, "text/plain", "Turned ON!");
} else {
    digitalWrite(controlPin, LOW);
    request->send(200, "text/plain", "Turned OFF!");
}
```

Add final server handlers outside the `*handler` to finish it up.

```cpp
server.addHandler(onBody);
server.addHandler(handler);
server.onNotFound(onRequest);
server.begin();
```

For this to work we need to define this digital pin. We can do this after we include the libraries at the beginning of the code.

```cpp
#include "AsyncJson.h"
#include "ArduinoJson.h"
#include "ESPAsyncWebSrv.h"
#define controlPin 13
```

## Step 2: How to Setup ESP32 Request Sender

Now that you have set up the request handler, once the pin has been updated, you can request the Vonage SMS API with the state of operation (success/failure). Add the following configuration at the beginning of the code.

```cpp
const char *ssid = "Your WiFi name here";
const char *password = "Your WiFi password here";
#define API_ENDPOINT = "https://rest.nexmo.com/sms/json"
#define FROM = "$VONAGE_BRAND_NAME"
#define TO = "Your phone number"
#define API_KEY = "$VONAGE_API_KEY"
#define API_SECRET = "$VONAGE_API_SECRET"
```

Create a new function that will send requests to Vonage.

```cpp
bool requestSender (String state) {
    http.begin(client, API_ENDPOINT.c_str());
    http.addHeader('Content-Type', 'application/json');
    String httpRequestData = "{\"from\":\"" + FROM.c_str() + "\",\"text\":\"," + state.c_str() + "\"\"to\":\"" + TO.c_str() + "\",\"api_key\":\"" + API_KEY.c_str() + "\",\"api_secret\":\"" + API_SECRET.c_str();
    int httpResponseCode = http.POST(httpRequestData);
    http.end();
    if (httpResponseCode == 200) return true;
    return false;
}
```

To not let the request hang, add a simple state-based function.

```cpp
void manageState (bool state, AsyncWebServerRequest *request) {
    if (state) {
        request->send(200, "text/plain", stateText);
        return;
    }
    request->send(200, "text/plain", "Something went wrong");
    return;
    }
```

Incorporate all this into `*handler` where the conditional logic happens.

```cpp
bool state = false;
String stateText = "";
if (text == "ON") {
    digitalWrite(controlPin, HIGH);
    stateText = "Turned ON!";
    Serial.println(stateText);
    state = requestSender(stateText);
    manageState(state);
} else {
    digitalWrite(controlPin, LOW);
    stateText = "Turned OFF!";
    Serial.println(stateText);
    state = requestSender(stateText);
    manageState(state);
}
```

## Step 3: How to Setup Ngrok with ESP32

To test the webhook locally, you can use [ngrok](https://ngrok.com/). Ngrok will allow you to expose your local endpoint to the internet and receive incoming requests from the Vonage SMS API.

You can install and run ngrok locally on your computer. To install ngrok, navigate to the [ngrok's download page](https://ngrok.com/download). Follow the operating system-specific instructions provided by ngrok.

![ngrok Download Page](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/ngrok-download-page.png "ngrok-download-page.png")

Next, power up your ESP32 and wait until it connects to the network. Connect it to a USB port on your computer and open the Serial Monitor inside Arduino IDE.

From the Serial Monitor, copy the ESP32's IP address shown upon the boot-up.

![ESP32's Local IP Address shown inside Arduino IDE's Serial Monitor](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/esp32-local-ip-arduino-ide.png "esp32-local-ip-arduino-ide.png")

Now, open a new terminal window on your computer. Run `ngrok http <esp32-ip-address>:80`. Replace `<esp32-ip-address>` with the local IP address of your ESP32.

For example, `ngrok http 10.10.1.10:80`. ngrok will now open a publicly-accessible tunnel to your ESP32.

![ngrok CLI Tunnel information shown inside a terminal window](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/ngrok-cli-output-terminal.png "ngrok-cli-output-terminal.png")

## Step 4: How to Setup the Vonage Webhook

Now that you have set up the ESP32 and the ngrok tunnel, you can set up the Vonage webhook. The webhook will send incoming SMS messages to the ngrok tunnel, which will forward them to the ESP32.

You can set up the webhook in your Vonage account dashboard by providing the ngrok tunnel URL. Navigate to [Vonage Settings](https://dashboard.nexmo.com/settings). In the API Keys tab, find SMS settings. Under SMS settings, set `SMS API` and webhook format to `POST-JSON`.

Back in your terminal window, copy the `Forwarding` URL.

![ngrok Forwarding URL shown inside a terminal window](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/ngrok-forwarding-url-terminal.png "ngrok-forwarding-url-terminal.png")

In the Vonage Settings, paste the following: `/rest/endpoint` under Inbound SMS webhooks. For example, `https://b462-31-223-156-70.ngrok-free.app/rest/endpoint`.

Press Save changes in the top-right corner to save the webhook.

## Step 5: Test the app

Send an SMS containing `ON` or `OFF` from your phone number to any [Vonage number](https://dashboard.nexmo.com/your-numbers).

After Vonage finishes processing the request, you should see a change on a device you're controlling (e.g., a light bulb) and the respective output in the Arduino IDE’s Serial Monitor.

![Serial output from ESP32 shown in the Arduino IDE](/content/blog/using-vonage-sms-api-for-home-automation-using-esp32-blog-md/serial-out-esp32-in-ide.png "serial-out-esp32-in-ide.png")

You should then receive a confirmation SMS back about the state of the operation.

## Next Steps and Resources

You can explore more advanced home automation systems using the ESP32 and Vonage SMS API. You can also check out the [Vonage Developer Documentation](https://developer.vonage.com/en/documentation) for more information on using the Vonage API.

To learn more about ESP32, you can read the official [ESP-IDF documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/).

## Conclusion

Using Vonage SMS for home automation using the ESP32 microcontroller is a powerful and convenient way to control various aspects of your home remotely. Following the steps outlined in this tutorial, you can set up a basic system and expand it to suit your needs. 

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send us a Tweet on [Twitter](https://twitter.com/vonagedev), and we'll get back to you. Thanks again for reading, and we'll catch you on the next one!