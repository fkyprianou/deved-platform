---
title: "Using Vonage APIs with MongoDB Atlas - Part 5: Vonage In-App Messaging"
description: MongoDB Atlas and its associated products are a great complement to Vonage APIs. In Part 5, we add in-app alerting to our app to notify the restaurant of new meeting invites
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

We wrap up our series on MongoDB Atlas and Vonage APIs with a look at our [In-App Messaging API](https://developer.vonage.com/en/client-sdk/overview). Through the other parts of the series we toured setting up MongoDB Atlas and Vonage in Part 1, how to add Vonage Verify to an application in Part 2, allowing customer interaction through our Messages API and Meetings API, and finally will show how we can utilize our In-App Messaging API to send notifications to users logged into the admin section of our site.
 
## In-App Messaging

In-App Messaging is part of an overall set of APIs call Conversations, with the other half of the API being our In-App Voice product. As their names suggest, In-App Messaging handles sending textual messages from one client device to another while In-App Voice will handle voice traffic. These APIs have SDKs for iOS, Android, and modern web browsers. For our demo, we will be showing off the Web Client SDK, but you can extend this to allow inter-platform communication from any combination of Web, iOS, and Android.

In-App Messaging and In-App Voice are supported by two other APIs that comprise our Conversations product. These are the Conversation API itself, which are events between one or more Members, and the Users API, which handles user context information. The general workflow is that a user will authenticate through your normal application, and will be associated with a User in our Users API. This User will become a Member of various Conversations, or [event stores](https://en.wikipedia.org/wiki/Event_store) comprised of different triggered events like `text`, `message:submitted`, `member:joined`, etc.

Each Client SDK will talk to these backend servers to facilitate real-time communication through voice or text events. This allows you as a developer to have one single interface to interact with, our APIs, and our SDKs handle all the work connecting and working with our APIs. This frees you from having to come up with platform-dependent messaging systems, or ways to shunt messages between different platforms. 

For our restaurant demo, we will use the In-App Messaging API to send a message to any logged-in admin user. This will cause a pop-up to appear on their screen with a link to the meeting that has been requested. While we are using it for just a single notification, you could expand this to an entire chat system if you wanted. We will use the [`nexmo-client`](https://www.npmjs.com/package/nexmo-client) to handle all the in-browser work, and we will use a custom client in our back-end server code to talk to the Conversations APIs.

## The Admin Notifications

Let's start by looking at how the admin section works. What we want to do is have a notification pop up when a user requests a meeting. For the pop-up notifications we can use [@meforma/vue-toaster](https://www.npmjs.com/package/@meforma/vue-toaster) which is a VueJS plugin that generates toast-style pop-ups (toasts are small notifications that pop-up at the bottom of the screen and then disappear). 

How do we actually get the notification though? We will need to use the `nexmo-client` web SDK to connect to our Vonage Application, and then subscribe to the correct Conversation, or series of messages. To handle this, we created a [Notification Component](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo/blob/main/webapp/src/views/Notifications.vue) that will connect to the Conversation and pop-up any new notifications that come in.

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

To connect the web SDK to Vonage, we need to get an authorization token. This is a token that is a signed JWT using the private key we generated way back in Part 1 when we created a Vonage Application. This JWT will allow the web SDK to talk to the Vonage APIs and authenticate. This is much the same way MongoDB Realms authenticated our admin user.

For full information on creating client JWTs for the Conversations API, [check out our documentation](https://developer.vonage.com/en/conversation/guides/jwt-acl). We will go over how we make the JWT in just a second so you can see an example, but for now, let's continue with the VueJS code. For right now all we need to worry about is that we ask our back-end server to generate a JWT for our user.

The back-end server will return two bits of information - the signed Vonage JWT, and the conversation ID for us to connect to. We will need this to tell the web SDK what conversation to listen to.

After that, we create a new `ConversationClient()`, which is an object from the `nexmo-client` package. We call `createSession(jwt)` on that object and pass in the JWT that our backend created for us. This authenticates our web application to Vonage. Then we just tell it to get a specific conversation, which is the ID that our backend server returned to us.

Our application is now listening for events, but we are not handling anything. We need to use the `conversation.on()` method to handle incoming messages. We will use a custom message called `support_request`, so we pass that name in as the first parameter. The second parameter is a handler function. Our handler function will extract the information from the support request and pop up a notification.

That's it. You can see this in action if you log into the admin section and go to **View Orders**. If you open a new window and log in as a customer and send an order, initiate a video call and a new notification should pop up in the admin window. 

### How did we generate a token?

Before we look at the backend code, let's look at how we made the JWT. There are two important pieces of information we need to add to the JWT to make a valid client-side token. This is `sub`, which is the user that the token is being generated for. The other is `path`, which is a list of paths that the JWT can access. 

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

To help restrict our token to just what we need, we set our ACL to `/*/sessions/**` and `/*/conversations/<conversation-id>/**`. This means our JWT is only valid for the specific conversation that our notifications are sent to, which is called `pos-notifications`. If someone were to hijack the token it would only be useful for accessing one singular conversation.

We further secure the JWT by telling it that we will only allow `GET` requests to the conversation. This means our front end or anyone who tries to use the JWT will only be able to read the conversation and not modify it. In your application, you should try and restrict the paths and methods to be as strict as possible for the JWT.

## Creating a Notification

We now know how to read from a conversation, but how do we push an event into a conversation? If we pop into our [`server.ts`](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo/blob/main/webapp/server/server.ts#L208) file and go to our `/api/website/video-call` route, we generate a new event there. 

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

We update the order with the new meeting URLs. Once that is finished, we look up the user in MongoDB using the `users` collection and `findOne()`. This gets us the user record for the authenticated user, and then we can use `conversations.addEventToConversation()` to push a new event into our conversation. For all the notifications we use a conversation named `pos-notifications`.

`addEventToConversation()` is a wrapper that actually handles a bunch of the underlying work it takes to add an event to a conversation, and is part of a collection of methods in [`conversation.ts`](https://github.com/Vonage-Community/sample-mongodb-vonage-integration-restaurant-demo/blob/main/webapp/server/conversations.ts) that will do a lot of lookups and setup for us. 

As I mentioned before, the Conversations API is a combination of a Conversations and a Users API. When we call `addEventToConversation()`, the method will attempt to fetch a conversation by a specified name. That conversation lookup will generate a conversation with a name (in our case, `pos-notification`) if one does not exist.

We next need a member to attach the event too. Members are Users that are part of a specific conversation. A Member is only ever a part of one conversation, but a User can be a member in multiple conversations. `addEventToConversation()` will attempt to look up a member by the specified email address. If that member is not a part of the conversation we are adding the event too, it will look up a user with that email. If that user does not exist, it creates that user. That user is then added to the conversation as a member, and the member information is returned.

We can then push the new event into the conversation. The event is propagated to any connected clients that are listening to the conversation. This means when we submit an event on the server, any logged-in users on the "Orders" page will get the event, and the pop-up notification.

## Conclusion

This is but a very small taste of what the Conversations API can do. You can build chat interfaces that have all of the features users expect like typing notifications, image sharing, and message storage. On top of that it is all real-time and cross-platform. A web user can send a notification to an iOS client exactly the same way as an Android client or another web client.

You can also couple this with either the In-App Voice to provide real-time voice between multiple clients, or combine this with our Voice API to allow an application user to call a telephone, or vice versa. The customer could instead call the restaurant and the restaurant could answer in their browser!

This brings an end to our tour through MongoDB Atlas and some complimentary Vonage APIs. I hope that this series has inspired you to see what each of our platforms provides and how they might be useful to you. As always, feel free to reach out to our developer advocates if you have any questions.

Happy coding!

* [Part 1 - What is MongoDB Atlas?]()
* [Part 2 - Using Vonage Verify with Logins]()
* [Part 3 - Using Vonage for Customer Interactions]()
* [Part 4 - Using Atlas for User Authentication]()
* Part 5 - Using Vonage In-App Messaging for Notifications