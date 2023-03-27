---
title: Using WhatsApp Product Messages with the Vonage Messages API
description: Learn how to use WhatsApp Product Messages to showcase products and
  services, and customers can browse and add products to a cart.
author: michael-crump
published: true
published_at: 2023-03-23T18:34:14.171Z
updated_at: 2023-03-23T18:34:14.222Z
category: tutorial
tags:
  - whatsapp
  - messages
  - api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

The [Vonage Messages API](https://developer.vonage.com/messages/overview) integrates with SMS, MMS, and popular social chat apps—so you can communicate with your customers on whichever channel they love most. That, paired with rich features and instant results, makes for one engaging, cost-effective chat experience.

Recently, we've made some enhancements to the WhatsApp communication platform by adding support for [WhatsApp Product Messages](https://developer.vonage.com/en/messages/concepts/whatsapp-product-messages). They allow a business to showcase products and services, and customers can browse and add products to a cart without leaving the WhatsApp conversation.

In this tutorial, we'll go step-by-step through setting up WhatsApp Product Messages with the Vonage Messages API.

## Prerequisites

Before you begin, make sure you have the following accounts activated and tools installed:

* A [Vonage Developer Account](https://developer.vonage.com/en/) - If you don't have one, you can create one, and we'll give you free credit to play around with our APIs.
* A [Meta Commerce Account](https://business.facebook.com/commerce/) - You will need the proper permissions to add a [product catalog](https://www.facebook.com/business/help/890714097648074) to it. 
* [ngrok](https://ngrok.com/) - A free account is required. This tool enables developers to expose a local development server to the Internet. 

## Adding Products to the catalog

To get our products in front of customers, we need a place to store the metadata, including the title, category, price, etc., and an image. Meta lets you keep all this information on their servers with the [Meta Commerce Manager](https://business.facebook.com/commerce/) page.

Go ahead and create an account if you haven't already and navigate to the [Meta Commerce Manager](https://business.facebook.com/commerce/) page and click on **Add catalog**. 

![Add a Catalog](/content/blog/getting-started-with-whatsapp-product-messages/add-catalog.png "add-catalog.png")

You'll need to specify your **Catalog Type**. Select **E-commerce** and press **Next**.

Next, we have two options to upload our product data. We can select to add our product **manually** or use a **partner platform**. We'll choose  **Upload product info**, leave the other items as they are, and press **Create**. 

![Configure settings](/content/blog/getting-started-with-whatsapp-product-messages/configure-settings.png "configure-settings.png")

You'll now have a new catalog, and if you click on **Items**, you'll see that you don't have any. Let's upload a couple of items by pressing the **Add items** button. 

![Add a item](/content/blog/getting-started-with-whatsapp-product-messages/add-items.png "add-items.png")

We'll add items by manually uploading them to our catalog. 

Select **Manual**, then **Next** to continue.

![Add a item manually](/content/blog/getting-started-with-whatsapp-product-messages/add-items-manually.png "add-items-manually.png")

Here we'll need to supply information for the following fields: **Title**, **Description**, **Price**,  **Website Link**, **Price**, **Sales price**, **Facebook Product Category**, **Condition**, **Availability** and **Status**. You'll also need an image of the product. Go ahead and add a couple of items. 

![Add a item manually - before](/content/blog/getting-started-with-whatsapp-product-messages/before-add-items.png "before-add-items.png")

In my instance, I am creating a Pizza shop, so I added a couple of pizzas and a drink, as shown below.

![Add a item manually - after](/content/blog/getting-started-with-whatsapp-product-messages/after-add-items.png "after-add-items.png")

Before proceeding, please ensure that all items are showing without errors.

Copy the **Content ID** of your items (from the previous screenshot), as we'll need those later. We will also need the **catalog_id**, which you can find on the **Catalog** page.

> Note that Vonage documentation refers to the **Content ID** as **product_retailer_id**.

If you would like further details about catalogs, please consult the [Meta Documentation](https://www.facebook.com/business/help/1275400645914358).

## Information about WhatsApp Business Account and working with Catalogs

To include product items in Product Messages sent from a number associated with your WhatsApp Business Account (WABA), the **Catalog** containing those items must be linked to your WABA.

There are primarily two ways of linking your WhatsApp Business Account to your Catalog. 

1. **Owned Account** - is created through the embedded sign-up process (via the [Vonage Dashboard](https://dashboard.nexmo.com/)).
2. **Managed Account** - is created for you by a partner business such as Vonage. To connect a Catalog to a Managed Account, you must assign the partner business (such as Vonage) to the Catalog.

We'll use an **Owned Account** for this example, but documentation on Connecting a **Managed Account** is located [here](https://developer.vonage.com/en/messages/concepts/whatsapp-product-messages#connecting-a-managed-account).

## Buy a Virtual Number

Before we set up an **Owned Account**, we'll need to buy a virtual phone number that we can use for our Pizza business. 

To purchase one, log into the [Vonage Dashboard](https://dashboard.nexmo.com/), go to **Numbers** > **Buy Numbers**, and search for one that meets your needs. 

![Adding a number to the application](/content/blog/getting-started-with-whatsapp-product-messages/buy-numbers.png "buy-numbers.png")

You'll want to note the number you purchased, as we'll use it in the next portion of the setup.

## Setting up an Owned WhatsApp Business Account

Log into the [Vonage Dashboard](https://dashboard.nexmo.com/), go to **External Account** and click on the **WhatsApp** Social Channel to begin. 

![Social channels](/content/blog/getting-started-with-whatsapp-product-messages/whatsapp-social-channels.png "whatsapp-social-channels.png")

You'll see a setup screen and press **Continue with Meta**, as shown below.

![Vonage Setup screen](/content/blog/getting-started-with-whatsapp-product-messages/whatsapp-vonage-setup.png "whatsapp-vonage-setup.png")

Next, you must complete your Meta Business Account and associate your WhatsApp Business Account and Profile. 

![Setup process](/content/blog/getting-started-with-whatsapp-product-messages/setupprocess.gif "setupprocess.gif")

Once complete, you'll see a popup that provides a few helpful tips on how to get unrestricted use of your WhatsApp Number, so press **Next**. You'll need to select the phone number you want to use and the API Key that WhatsApp message traffic will go to. You can leave the preferred hosting to WhatsApp. Press **Get my WhatsApp number Live** to continue.  

![Associate a number with Vonage](/content/blog/getting-started-with-whatsapp-product-messages/associate-number-with-vonage.png "associate-number-with-vonage.png")

You should see your connected WhatsApp account on the **External Accounts** screen in the Vonage Dashboard.

![Your connected social channels](/content/blog/getting-started-with-whatsapp-product-messages/connected-wa-account.png "connected-wa-account.png")

## Connecting the catalog to WhatsApp

Head over to your [Meta Commerce Account](https://business.facebook.com/commerce/) and navigate to  **Business Settings**, **WhatsApp Accounts**, then **WhatsApp Manager**, as shown below. 

![WhatsApp Accounts Settings](/content/blog/getting-started-with-whatsapp-product-messages/wa-accounts.png "wa-accounts.png")

Go to **Account tools**, **Catalog**, then **Choose a Catalog**

![WhatsApp Manager - Choose a catalog](/content/blog/getting-started-with-whatsapp-product-messages/wa-manager.png "wa-manager.png")

You'll now have the option to connect a catalog. Go ahead and select the one that we created earlier, called **Catalog_products**.

![WhatsApp Manager - Connect a catalog](/content/blog/getting-started-with-whatsapp-product-messages/connect-catalog.png "connect-catalog.png")

Head to your **Meta Business Settings**, click on **Data Sources**, then **Catalogs**, and input Vonage as a partner using a Business ID. The ID is `2290848174274168`. Contact Vonage Support, who will arrange to approve the partner request and connect the Catalog.

![WhatsApp Manager - Partners](/content/blog/getting-started-with-whatsapp-product-messages/partners.png "partners.png")

## Create a Messages-Enabled Vonage Application

To interact with the Messages API, we'll need to create a Vonage API application to authenticate our requests. Think of applications more like containers and metadata to group all your data on the Vonage platform. We'll [create one using the Vonage API Dashboard](https://dashboard.nexmo.com/applications/new). 

Provide a name (such as "WhatsAppProductMessage") and click on **Generate public and private key**. You'll be asked to save a keyfile to disk—the private key. It's usually a good idea to keep it in your project folder, as you'll need it later. We'll use the private key to generate a JSON Web Token (JWT) later.

> Applications work on a public/private key system, so when you create an application, a public key is generated and kept with Vonage. A private key is generated, not kept with Vonage, and returned to you via the application creation.

Next, you need to enable the **Messages** capability and provide an **Inbound URL** and a **Status URL**. This is typically where you'd want to use something like ngrok to forward port 4040. You can do this from a terminal or command prompt by entering `ngrok http 4040`. You'll now have a forwarding address that you can input in both the **Inbound** and **Status** URLs. 

![Ngrok running](/content/blog/getting-started-with-whatsapp-product-messages/ngrok-running.png "ngrok-running.png")

When a message reaches the **Messages API**, the data about it is sent to the **Inbound URL**. When you send a message using the API, the data about the message status gets sent to the **Status URL**.

Finally, once it is created, go to **Link social channels**, select the WhatsApp number created earlier, and press **Link**.

![Linking a number](/content/blog/getting-started-with-whatsapp-product-messages/link-wa-account.png "link-wa-account.png")

## Create a JWT Token

JSON Web Token (JWT) defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. For security measures, it is recommended to use a JWT Token when working with the Messages API and other Vonage products. 

We have created a site where you can create a token with just a few steps. Click [here](https://developer.vonage.com/en/jwt) and add your **Private Key** as well as your **Application ID**. Then you'll need to specify how long the token will be good for. Select three days, and your JWT token is available. Copy that to a safe place for now. 

![JWT Generator](/content/blog/getting-started-with-whatsapp-product-messages/jwt-generator.png "jwt-generator.png")

## Configure Message Templates

Keep in mind that a **Product Message** cannot be used to initiate a WhatsApp conversation. It can only be sent as part of an existing conversation that has been started either by the customer messaging the business or the business sending a Template Message to the customer. In this example, we'll send a template message to the customer using **WhatsApp Manager**. Once logged in, select **Account Tools**, then **Message Template**.

![WhatsApp Manager - Message Templates](/content/blog/getting-started-with-whatsapp-product-messages/wa-message-templates.png "wa-message-templates.png")

Select **Create Message Template**. Under Category, select **Utility**. Give it the name of **sample_utility_message** and choose your language of choice. Once complete, press **Continue**. 

![WhatsApp Manager - New Message Template](/content/blog/getting-started-with-whatsapp-product-messages/utility-message.png "utility-message.png")

Provide a **Body** text such as "Welcome to Michael's Grocery Store! Feel free to browse." Now hit the **Submit** button. You'll see the following screen; it takes some time before Meta approves it. (It took almost 10 hours to get approved when writing this article.)

![WhatsApp Manager - Message Template Status](/content/blog/getting-started-with-whatsapp-product-messages/message-template-status.png "message-template-status.png")

## Testing the application

Once your **Message Template** has been approved, save the three scripts below to test the application by making only the following changes.

* Replace **ADD_YOUR_JWT_TOKEN** with the one created in the previous step. 
* Replace **ADD_YOUR_VONAGE_NUMBER** with the  Vonage Number that you bought earlier.
* Replace **ADD_YOUR_DESTINATION_NUMBER** with a number that you can reach.

**template.sh** - Run this after your message template was approved by Meta. 

```bash
curl --location 'https://api.nexmo.com/v1/messages' \
--header 'Authorization: Bearer ADD_YOUR_JWT_TOKEN' \
--header 'Content-Type: application/json' \
--data '{
  "message_type": "template",
  "template": {
      "name": "sample_utility_message"
         },  
    "from":"ADD_YOUR_VONAGE_NUMBER",
    "to":"ADD_YOUR_DESTINATION_NUMBER",
  "channel": "whatsapp",
   "whatsapp": {
      "policy": "deterministic",
      "locale": "en_US"
   }
}'
```

**single.sh** - Run this after you have received and interacted with the message. You should also add the **catalog_id** and **product_retailer_id** to show a single message.

```bash
curl --location 'https://api.nexmo.com/v1/messages' \
--header 'Authorization: Bearer ADD_YOUR_JWT_TOKEN' \
--header 'Content-Type: application/json' \
--data '{
    "from":"ADD_YOUR_VONAGE_NUMBER",
    "to":"ADD_YOUR_DESTINATION_NUMBER",
    "channel": "whatsapp",
    "message_type": "custom",
    "custom": {
        "type": "interactive",
        "interactive": {
            "type": "product",
            "body": {
                "text": "New products from Michaels Grocery Store"
            },
            "footer": {
                "text": "Thanks for shopping with us."
            },
            "action": {
                "catalog_id": "166239142936236",
                "product_retailer_id": "p1di9amxxj"
            }
        }
    }
}'
```

multiple.sh - Run this anytime after the initial template has been received. You should also add the **catalog_id** and **product_retailer_id** to show multiple messages.

```bash
curl --location 'https://api.nexmo.com/v1/messages' \
--header 'Authorization: Bearer ADD_YOUR_JWT_TOKEN' \
--header 'Content-Type: application/json' \
--data '{
    "from":"ADD_YOUR_VONAGE_NUMBER",
    "to":"ADD_YOUR_DESTINATION_NUMBER",
    "channel": "whatsapp",
    "message_type": "custom",
    "custom": {
        "type": "interactive",
        "interactive": {
            "type": "product_list",
            "header": {
                 "type": "text",
                 "text": "Movie Night Food"
             },
            "body": {
                "text": "New products from Michaels Grocery Store"
            },
            "footer": {
                "text": "Thanks for shopping with us."
            },
            "action": {
                "catalog_id": "166239142936236",
                "sections": [
                    {
                        "title": "Pizza Choices",
                        "product_items": [
                            {
                                "product_retailer_id": "p1di9amxxj"
                            },
                            {
                                "product_retailer_id": "es2knk1593"
                            },
                            {
                                "product_retailer_id": "l3sgic2vmv"
                            }
                        ]
                    }
                ]
            }
        }
    }
}'
```

## Viewing the results in WhatsApp

After you send the **template.sh** and the **single.sh**, you'll see a conversation initiated by the business showing only the Cola. 

![WhatsApp Message sent to the customer](/content/blog/getting-started-with-whatsapp-product-messages/iphone1.png "iphone1.png")

Clicking on **View**, you will have the option to **Add to Cart** or **Message Business**. We'll add the Cola to the cart and we can now send this order request back to the business. 

![WhatsApp Message sent to the customer and added to the cart](/content/blog/getting-started-with-whatsapp-product-messages/iphone2.png "iphone2.png")

For multiple products, run the **multiple.sh** script and you'll see the following. We can also add multiple items to the cart as shown below. 

![WhatsApp Message sent to the customer](/content/blog/getting-started-with-whatsapp-product-messages/iphone3.png "iphone3.png")

![WhatsApp Message sent to the customer](/content/blog/getting-started-with-whatsapp-product-messages/iphone4.png "iphone4.png")

## Wrap-up

Now that you have seen how to use WhatsApp Product Message, it is time to create your own! You could extend this project by using a payment processor to automatically accept payment without any user intervention. Also, don't forget to refer to the  [WhatsApp Product Messages](https://developer.vonage.com/en/messages/concepts/whatsapp-product-messages) page for more information.

If you have questions or feedback, join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack) or send me a Tweet on [Twitter](https://twitter.com/mbcrump), and I will get back to you. Thanks again for reading, and I will catch you on the next one!