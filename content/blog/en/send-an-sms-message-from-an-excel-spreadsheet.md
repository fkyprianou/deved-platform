---
title: Send an Sms Message From an Excel Spreadsheet
description: Learn how to send an SMS Message from Excel using VBA
thumbnail: /content/blog/send-an-sms-message-from-an-excel-spreadsheet/sms-excel.png
author: michael-crump
published: true
published_at: 2023-02-02T10:02:40.830Z
updated_at: 2023-02-02T10:02:41.758Z
category: tutorial
tags:
  - sms-api
  - office
  - excel
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

Microsoft offers a way to add new functionality to Office applications such as Microsoft Excel to prompt and interact with the user in ways specific to your business needs. You can perform these tasks and accomplish more by using Visual Basic for Applications (VBA). It is a simple but powerful programming language that you can use to do such things as get your contacts from Microsoft Outlook into a Microsoft Excel spreadsheet.

In this blog post, we'll use VBA in Microsoft Excel to send an SMS Message. Just enter the cell phone number and text to be sent and press a button to send it! You could modify the code to add additional functionality your business may require, such as scheduling. Let's get started!

## Create a Vonage Developer Account

We will need to create a [Vonage Developer Account](https://developer.vonage.com) to send SMS Messages from Vonage. Once you create an account, you'll see your API Key and API Secret on the Vonage API Dashboard. Once you have those values, store them in a safe place, as we will use those shortly

![API Dashboard that shows API Key/Secret](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/apidashboard.png "APIDashboard.png")

## Refer to the Vonage SMS API Documentation

Before we start coding, we should first consult the [Vonage SMS API documentation](https://developer.vonage.com/en/messaging/sms/code-snippets/send-an-sms) to examine what fields the API expects us to provide when called. Upon visiting the documentation, we can see several programming SDKs are supported. Since Excel only uses VBA, we'll need to call the REST Endpoint that Vonage provides.

After examing the REST code snippet, we can see that there are several required fields, such as: 

* A `from` number
* A `to` number
* The `text` that you want to send. 
* An `api_key` and `api_secret`, which you should have from the step above.

The cURL example below shows what it looks like when called.

```curl
curl -X "POST" "https://rest.nexmo.com/sms/json" \
  -d "from=$VONAGE_BRAND_NAME" \
  -d "text=A text message sent using the Vonage SMS API" \
  -d "to=$TO_NUMBER" \
  -d "api_key=$VONAGE_API_KEY" \
  -d "api_secret=$VONAGE_API_SECRET"
```

If you run this code with valid information and credentials, you will see the following output appear in your terminal:

```json
{
  "messages": [
    {
      "to": "14259999999",
      "message-id": "3c153507-8ade-4bd1-ab6f-12cb6f7f9efe",
      "status": "0",
      "remaining-balance": "39.08381985",
      "message-price": "0.00869000",
      "network": "310260"
    }
  ],
  "message-count": "1"
}
```

Note that `status` returns a status code of 0 which means it was sent successfully. If you sent this to your phone number, a text message should have appeared. Next, let's set up Microsoft Excel.

## Setting up our Excel Spreadsheet

As noted earlier, Microsoft Excel can use VBA (Visual Basic for Applications) to interact with the data programmatically.
Let's begin by creating a spreadsheet with a couple of rows of data containing the numbers we want to send the text messages to and the message we want to send.

Let's begin by creating a spreadsheet with a couple of rows of  data containing the number that we want to send the text message to as well as the message that we want sent. 

Structure your Excel Sheet like the following: 

* A1 & A2 will include the cell phone numbers.
* B1 & B2 will include the text to be sent. 
* H1 will include the "API Key" text.
* H2 will include the "API Secret" text.
* I1 is your API Key from the Vonage API Dashboard.
* I2 is your API Secret from the Vonage API Dashboard.
* I3 will be a button that says "Send SMS." (I'll cover the button creation shortly.)
* Give your worksheet a name, such as `Numbers`.

Now you should have a screen that looks like the following. 

![Excel with sample data](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/excelstart.png "ExcelStart.png")

## Enabling and Configuring Developer Mode in Microsoft Excel

We need to add an option to see the **Developer** menu option to use VBA. You can turn this on by going to **File** -> **Options** -> **Customize Ribbon** and placing a checkmark in the **Developer** option, as shown below.

![Excel Dev Mode](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/exceldevmode.png "ExcelDevMode.png")

The **Developer** menu option should be showing now in the Menu Bar. Select it and then the **Visual Basic** option to allow code to be entered into the system.

![Visual Basic Option](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/visualbasic.png "VisualBasic.png")

A new Window will appear with Microsoft Visual Basic for Applications running. Select **Insert** from the Menu options and select **Module** to input Visual Basic code that interacts with the Excel sheet. 

![Excel New Module](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/newmodule.png "NewModule.png")

We need to add a **Reference** to the **Microsoft XML 6.0** library, which will allow us to call a REST Endpoint. Go to **Tools** -> **Reference** and check **Microsoft XML, v6.0**, as shown below:

![Add Reference Dialog](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/addreference.png "AddReference.png")

## Adding VBA Code to call the Vonage SMS API

Copy and paste the following code block to your application and note the comments that I left that describe what each section is doing.

```vb
Sub SendSMS()

'Authentication
ApiKey = ActiveSheet.Range("I1").Value
ApiSecret = ActiveSheet.Range("I2").Value

'Define our Worksheet so we can loop through the data for the number and message to send. 
Dim wb As Workbook
Dim ws As Worksheet

Set wb = ThisWorkbook
Set ws = wb.Worksheets("Numbers")

'Loop through Worksheet content - I only had two rows in this example, but you can modify this to your needs. 
For i = 1 To 2
   toNumber = ws.Range("A" & i).Value 'our number to text
   bodyText = ws.Range("B" & i).Value 'the text we wish to send
   
    'Use Microsoft's XML Library to make a web request.
    Set Request = CreateObject("MSXML2.ServerXMLHTTP.6.0")
    
    'Concatenate the required data that Vonage's REST Endpoint is expecting.
    Url = "https://rest.nexmo.com/sms/json?from=18335787204" & "&to=" & toNumber & "&text=" & bodyText & "&api_key=" & ApiKey & "&api_secret=" & ApiSecret
    
    'Open POST Request
    Request.Open "POST", Url, False
    
    'Set the Request Header
    Request.setRequestHeader "Content-Type", "application/x-www-form-urlencoded"
    
    'Send Request
    Request.send Url
Next i

'OPTIONAL: You can get the response text after a message is sent by calling MsgBox Request.responseText
End Sub
```

Lastly, we need to add a **Button** to trigger this event. Select **Insert** from the **Developer Menu**, Click **Button**, and place it in I3.

![Select VBA Button](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/excelbutton.png "ExcelButton.png")

Once you let go of your mouse, you will be asked to **Assign Macro** to that **Button**. Select the **SendSMS** Macro we just pasted, as shown below, and press **OK**.

![Assign Macro to Worksheet](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/assignmacro.png "AssignMacro.png")

It is time to test it! Make sure you have a working, valid phone number for your location in A1 & A2 and text in B1 & B2, and press the **SendSMS** button.

![Excel with sample data](/content/blog/send-an-sms-message-from-an-excel-spreadsheet/excelstart.png "ExcelStart.png")

If everything went well, you should have received a text with the information provided in the spreadsheet.

## Wrap-up

VBA is very effective and efficient regarding solutions for Microsoft Office-related products, as we saw today. As I was doing research for this blog post, I learned that it has been in use for almost thirty years already. This makes me confident that it is a mature platform; chances are others have already run across the same problems I might have.

If you want to do this in Google Sheets, you should [look here](https://developer.vonage.com/en/blog/how-to-send-sms-from-a-spreadsheet-dr). Google Sheets and Microsoft Excel share many similarities, and you'll use the same great Vonage SMS API discussed here.

As always, if you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!