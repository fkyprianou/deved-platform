---
title: "Using Vonage APIs with MongoDB Atlas - Part 5 of 5"
description: In Part 5 of using Vonage APIs and MongoDB Atlas, we add in-app alerting to our app to notify the restaurant of new meeting invites
thumbnail: 
author: christankersley
published: true
published_at: 
updated_at: 
category: tutorial
tags:
 - mongodb
 - node
 - conversations-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---

In this series:

* [Part 1 - What is MongoDB Atlas?]()
* [Part 2 - Using Vonage Verify with Logins]()
* [Part 3 - Using Vonage for Customer Interactions]()
* [Part 4 - Using Atlas for User Authentication]()
* Part 5 - Using Vonage In-App Messaging for Notifications

We wrap up our series on MongoDB Atlas and Vonage APIs with a look at our [In-App Messaging API](https://developer.vonage.com/en/client-sdk/overview). Through the other parts of the series, we toured setting up MongoDB Atlas and Vonage in Part 1, how to add Vonage Verify to an application in Part 2, allowing customer interaction through our Messages API and Meetings API, and finally will show how we can utilize our In-App Messaging API to send notifications to users logged into the admin section of our site.
 
## In-App Messaging

In-App Messaging is part of a comprehensive set of API call conversations, with the other half of the API being our In-App Voice product. As their names suggest, In-App Messaging handles sending textual messages from one client device to another, while In-App Voice will handle voice traffic. These APIs have SDKs for iOS, Android, and modern web browsers. We will be showing off the Web Client SDK for our demo, but you can extend this to allow inter-platform communication from any combination of Web, iOS, and Android.

In-App Messaging and In-App Voice are supported by two other APIs that comprise our Conversations product. These are the Conversation API, which are events between one or more Members, and the Users API, which handles user context information. The general workflow is that a user will authenticate through your application and be associated with a User in our Users API. This User will become a Member of various Conversations, or [event stores](https://en.wikipedia.org/wiki/Event_store) comprised of different triggered events like `text`, `message:submitted`, `member:joined`, etc.

Each Client SDK will talk to these backend servers to facilitate real-time communication through voice or text events. This allows you as a developer to have one interface to interact with our APIs, and SDKs handle all the work connecting and working with our APIs. This frees you from having to come up with platform-dependent messaging systems or ways to shunt messages between different platforms.

We will use the In-App Messaging API for our restaurant demo to message any logged-in admin user. This will cause a pop-up to appear on their screen with a link to the meeting that has been requested. While we use it for just a single notification, you can expand this to an entire chat system. We will use the [`nexmo-client`](https://www.npmjs.com/package/nexmo-client) to handle all the in-browser work, and we will use a custom client in our back-end server code to talk to the Conversations APIs.

## The Admin Notifications

Let's start by looking at how the admin section works. We want to have a notification pop up when a user requests a meeting. For the pop-up notifications, we can use [@meforma/vue-toaster](https://www.npmjs.com/package/@meforma/vue-toaster), which is a VueJS plugin that generates toast-style pop-ups (toasts are small notifications that pop-up at the bottom of the screen and then disappear).

How do we get the notification, though? We must use the `nexmo-client` web SDK to connect to our Vonage Application and then subscribe to the correct Conversation or series of messages. We created a [Notification Component](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo/blob/main/webapp/src/views/Notifications.vue) that will connect to the Conversation and pop-up any new notifications that come in.

```typescript
import ConversationClient from 'nexmo-client'
import { createToaster } from '@meforma/vue-toaster'
import { authenticationStore } from '../stores/authenticationStore';

const authStore = authenticationStore();
const toaster = createToaster({ dismissable: true });

async function boot() {
    const data = await fetch(import.meta.env.VITE_API_URL + '/jwt', {
        method: 'POST',
        body: JSON.stringify({username: authStore.username}),
        headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer ' + authStore.token
        }
    })
        .then(resp => resp.json())
        .catch(err => console.log(err));
    
    const jwt = data.token
    const conversationID = data.conversation

    const client = new ConversationClient({ debug: true })
    let app = await client.createSession(jwt)
    const conversation = await app.getConversation(conversationID)

    conversation.on("support_request", async (sender: any, event: any) => {
        toaster.show(`${event.body.email} has request support: <a target="_blank" href="${event.body.host_url}">Join Meeting</a>`);
    });
}

boot()
```

We need an authorization token to connect the web SDK to Vonage. This token is a signed JWT using the private key we generated in Part 1 when we created a Vonage Application. This JWT will allow the web SDK to talk to the Vonage APIs and authenticate. This is much the same way MongoDB Realms authenticated our admin user.

For complete information on creating client JWTs for the Conversations API, [check out our documentation](https://developer.vonage.com/en/conversation/guides/jwt-acl). We will review how we make the JWT in just a second so you can see an example, but for now, let's continue with the VueJS code. Right now, we only need to worry about asking our back-end server to generate a JWT for our users.

The back-end server will return two bits of information - the signed Vonage JWT and the conversation ID for us to connect to. We will need this to tell the web SDK what conversation to listen to.

After that, we create a new `ConversationClient()`, an object from the `nexmo-client` package. We call `createSession(jwt)` on that object and pass in the JWT that our backend created for us. This authenticates our web application to Vonage. Then we tell it to get a specific conversation, the ID our backend server returned to us.

Our application is now listening to events but not handling anything. We must use the `conversation.on()` method to handle incoming messages. We will use a custom message called `support_request` to pass that name in as the first parameter. The second parameter is a handler function. Our handler function will extract the information from the support request and pop up a notification.

That's it. If you log into the admin section and go to **View Orders**, you can see this in action. If you open a new window, log in as a customer, and send an order, initiate a video call, and a new notification should pop up in the admin window.

### How did we generate a token?

Before we look at the backend code, let's look at how we made the JWT. We must add two crucial pieces of information to the JWT to create a valid client-side token. This is `sub`, which is the user that the token is being generated for. The other is `path`, a list of paths the JWT can access.

```typescript
app.post('/jwt', async (req, res) => {
    const { username } = req.body;
    const conversation = await conversations.fetchByName('pos-notifications');
    const conversationPath = `/*/conversations/${conversation.id}/**`;
    let acl = {
        "paths": {
            "/*/sessions/**": {},
          }
    }
    acl.paths[conversationPath] = { methods: ['GET'] }

    const key = readFileSync(process.env.VONAGE_PRIVATE_KEY);
    const token = tokenGenerate(process.env.VONAGE_APPLICATION_ID, key, {
        sub: username,
        acl: acl,
    });
    res.json({ token, conversation: conversation.id })
})
```

To help restrict our token to just what we need, we set our ACL to `/*/sessions/**` and `/*/conversations/<conversation-id>/**`. This means our JWT is only valid for the specific conversation that our notifications are sent to, called `pos-notifications`. If someone were to hijack the token, it would only help access one conversation.

We further secure the JWT by telling it we will only allow `GET` requests to the conversation. This means our front end or anyone who tries to use the JWT will only be able to read the conversation and not modify it. In your application, you should restrict the paths and methods to be as strict as possible for the JWT.

## Creating a Notification

We know how to read from a conversation, but how do we push an event into a conversation? If we pop into our [`server.ts`](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo/blob/main/webapp/server/server.ts#L208) file and go to our `/api/website/video-call` route, we generate a new event there.

```typescript
const orderRecord = await client.db('restaurant_pos_demo')
    .collection('orders')
    .updateOne(
        { _id: new ObjectId(orderNumber) },
        { $set: { meetingUrl: data._links.host_url.href}}
    )
    .then(async (document) => {
        const userRecord = await client.db('restaurant_pos_demo')
            .collection('users')
            .findOne({ _id: new ObjectId(decodedToken.user_id) });

        await conversations.addEventToConversation(
            'pos-notifications',
            userRecord.username, 
            {
                type: 'custom:support_request',
                body: { 
                    host_url: data._links.host_url.href,
                    email: userRecord.username
                }
            });

        res.json({
            guest_url: data._links.guest_url.href
        })
    });
```

We update the order with the new meeting URLs. Once that is finished, we look up the user in MongoDB using the `users` collection and `findOne()`. This gets us the user record for the authenticated user, and then we can use `conversations.addEventToConversation()` to push a new event into our conversation. For all the notifications, we use a conversation named `pos-notifications`.

`addEventToConversation()` is a wrapper that handles a bunch of the underlying work it takes to add an event to a conversation and is part of a collection of methods in [`conversation.ts`](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo/blob/main/webapp/server/conversations.ts) that will do a lot of lookups and setup for us.

As I mentioned before, the Conversations API is a combination of a Conversations and a Users API. When we call `addEventToConversation()`, the method will attempt to fetch a conversation by a specified name. That conversation lookup will generate a conversation with a name (in our case, `pos-notification`) if one does not exist.

We next need a member to attach the event too. Members are Users that are part of a specific conversation. A Member is only a part of one conversation. Still, a User can be a member in multiple conversations. `addEventToConversation()` will attempt to look up a member by the specified email address. If that member is not a part of the conversation, we are adding the event, too. It will look up a user with that email. If that user does not exist, it creates that user. That user is then added to the conversation as a member, and the member information is returned.

We can then push the new event into the conversation. The event is propagated to any connected clients listening to the conversation. When we submit an event on the server, any logged-in users on the "Orders" page will get the event and the pop-up notification.

## Conclusion

This is a tiny taste of what the Conversations API can do. You can build chat interfaces with all the features users expect, like typing notifications, image sharing, and message storage. On top of that, it is all real-time and cross-platform. A web user can send a notification to an iOS client the same way as an Android client or another web client.

You can also couple this with the In-App Voice to provide real-time voice between multiple clients or combine this with our Voice API to allow an application user to call a telephone or vice versa. The customer could instead call the restaurant, and the restaurant could answer in their browser!

This ends our tour through MongoDB Atlas and some complimentary Vonage APIs. Hopefully, this series has inspired you to see what each of our platforms provides and how they might be helpful to you. As always, contact our developer advocates if you have any questions.

Happy coding!

* [Part 1 - What is MongoDB Atlas?]()
* [Part 2 - Using Vonage Verify with Logins]()
* [Part 3 - Using Vonage for Customer Interactions]()
* [Part 4 - Using Atlas for User Authentication]()
* Part 5 - Using Vonage In-App Messaging for Notifications