---
title: "Action flows that you can do with the Voice API "
description: This article will describe how to use NCCO builder with Python code
  snippets as an example.
author: oleksii-borysenko
published: true
published_at: 2023-03-14T10:47:04.939Z
updated_at: 2023-03-14T10:47:04.955Z
category: tutorial
tags:
  - puthon
  - voice-api
  - ncco
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
You can build high-quality programmable voice applications in the cloud with Vonage Voice API. For example, you can manage outbound and inbound calls in Call Control Objects, record and store calls, create a conference call, send text-to-speech messages in [many languages](https://developer.vonage.com/en/voice/voice-api/concepts/text-to-speech#supported-languages) with varieties of voices and accents.

NCCO - is Call Control Objects (NCCOs) that describe the flow of a Voice API call. NCCO is represented by a JSON array.

This article will describe how to use [NCCO builder](https://github.com/Vonage/vonage-python-sdk#ncco-builder) with Python code snippets as an example.

## NCCO builder, Python SDK

Use the builder to construct valid NCCO actions, which are modeled in the SDK as [Pydantic](https://docs.pydantic.dev/) models, and build them into an NCCO. 
In Pydantic models [described](https://github.com/Vonage/vonage-python-sdk/blob/5458a68765584e54fbdfd26efa99d306c4682290/src/vonage/ncco_builder/ncco.py) required and optional parameters for related NCCO action.
The NCCO actions supported by the builder are:

| NCCO action | Sample of NCCO Python object | 
| :--- | :--- |    
| Record | `record = Ncco.Record(eventUrl=['https://example.com'])` |
| Conversation | `conversation = Ncco.Conversation(name='Audio demo')` |
| Connect | `connect = Ncco.Connect(endpoint={'type':'phone','number':'123456789'})` |
| Talk | `talk = Ncco.Talk(text='Press 1 for maybe and 2 for not sure followed by the hash key', language='en-GB')` |
| Stream | `stream = Ncco.Stream(streamUrl='https://nexmo-community.github.io/ncco-examples/assets/voice_api_audio_streaming.mp3')` |
| Input | `input = Ncco.Input(type=['dtmf', 'speech'], dtfm={'maxDigits':'1'})` |
| Notify | `notify = Ncco.Notify(payload={"foo": "bar"}, eventUrl='https://example.com/webhooks/event')` |


Do you want to use NCCO builder in your application? First, you need to import Vonage packages

```python
import vonage
from vonage import Ncco
```


If you use an NCCO object, you can validate it [here](https://dashboard.nexmo.com/voice/playground?adobe_mc=MCMID%3D08826528706681624421971769058934110597%7CMCORGID%3DA8833BC75245AF9E0A490D4D%2540AdobeOrg%7CTS%3D1678744202). 

If you are using NCCO builder, you can receive error messages in the terminal 
The following string can show a terminal error message if one of the required action options doesn't valid
```bash
string does not match regex "^[1-9]\d{6,14}$" (type=value_error.str.regex; pattern=^[1-9]\d{6,14}$)
```


## Build into an NCCO

Method `Ncco.build_ncco` can create an NCCO from the actions. This will be returned as a list of dicts representing each action and can be used in calls to the Voice API.

List of required and optional parameters you can find on [NCCO reference page](https://developer.vonage.com/en/voice/voice-api/ncco-reference)


Let's create NCCO that can connect two users.

The following code construct actions, talk and connect, that we need
```python
talk = Ncco.Talk(text='Please wait while we connect you.')
connect = Ncco.Connect(endpoint={'type':'phone','number':'123456789'}, from_ = '12345678', timeout='20')
```

The Connect action has each valid endpoint type (phone, application, WebSocket, SIP, and VBC) specified as a Pydantic model, so these can be validated, though it is also possible to pass in a dict with the endpoint properties directly into the `Ncco.Connect` object.

Let's create NCCO with `build_ncco` method and two actions
```python
ncco = Ncco.build_ncco(talk, connect)
```

The following JSON will be created

```json
[
   {
      "action": "talk",
      "text": "Please wait while we connect you."
   },
   {
      "action": "connect",
      "endpoint" :[
         {
            "type": "phone",
            "number": "123456789"
         }
      ],
      "from": "12345678",
      "timeout":20
   }
]
```

## Wrap-up
The new NCCO builder simplifies and can automate action creation. Based on official documentation and examples in this tutorial, you can use NCCO builder for your application.
