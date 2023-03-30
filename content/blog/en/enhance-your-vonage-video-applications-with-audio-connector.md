---
title: Enhance your Vonage Video Applications with Audio Connector
description: An overview of what the Audio Connector feature is, how to use it,
  and a walk-through of a sample application which uses it for transcribing a
  video call.
thumbnail: /content/blog/enhance-your-vonage-video-applications-with-audio-connector/audio-connector.jpg
author: karl-lingiah
published: true
published_at: 2023-03-30T12:29:34.954Z
updated_at: 2023-03-30T12:29:35.027Z
category: inspiration
tags:
  - video-api
  - audio-connector
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Video communication has evolved rapidly over recent years, and there are lots of features these days for customising the video calling experience for users. Many of these features focus on manipulating the video feed, such as background replacement or background blur. The audio component of a video call shouldn't be neglected though. The ability to work with audio feeds opens up a ton of possibilities for enhancing the overall user experience. If you're using Vonage's video API, then the Audio Connector feature is designed exactly for this purpose!

### What is Audio Connector?

Audio Connector is a feature of the Vonage Video API that lets you send raw audio streams from a Vonage Video session, via your own WebSocket server, to external services for further processing.

There are a few ways in which this feature can be used:

* Send a single audio stream to a WebSocket URL
* Send multiple audio streams, each to a separate WebSocket URL
* Send multiple, mixed audio streams to a single WebSocket URL

![Diagram of the Audio Connector routing streams form client publishers via the Vonage Video Media Router to a WebSocket server](/content/blog/enhance-your-vonage-video-applications-with-audio-connector/audio-connector-diagram.png "Overview Diagram of Audio Connector")

### What can you do with Audio Connector?

There are many potential applications and use-cases for the Audio Connector feature. For example, processing audio streams to create live captioning, or live or offline transcriptions or translations of the audio could be useful for improving accessibility, as well as in a record-keeping context. Scanning for specific words in an audio stream could be used for content moderation, or for search and index applications. Audio streams can also be used in media intelligence applications, such as creating a textual summary of a call or meeting. Another application is using an audio stream for sentiment analysis of a conversation. These are just a few examples of the many possibilities made available via this feature.

### How can you use Audio Connector in your applications?

Adding Audio Connector functionality to your Vonage Video applications requires the following components:

1. A websocket, with a publicly accessible websocket URI
2. An external service to process the audio in some way
3. The Vonage Video /connect REST endpoint

There are many ways in which you could set up components 1 and 2, and how you decide to do this is really up to you. Later in this article,we’ll walk through of a sample application which uses [Koa Websocket](https://github.com/kudos/koa-websocket) for the websocket component, and [Symbl.ai](https://symbl.ai/) for the audio processing component.

With regards to component 3, the Vonage Video /connect REST endpoint, detailed documentation is available in our [REST API reference](https://tokbox.com/developer/rest/#starting_audio_connector) and [Developer Guide](https://tokbox.com/developer/guides/audio-connector/). In brief though, an HTTP POST request to the endpoint will start streaming the audio from the specified Vonage Video session to the designated websocket URI. The request body requires certain information to be included, such as the sessionId for the Video Session from which the audio will be streamed, a valid token for that session, and the uri for the websocket that the audio is to be streamed to. There are also some optional properties, including a streams array which allows you to specify individual audio streams to be streamed to the websocket; we’ll look at this property in more detail as part of the sample application walkthrough.

On the Vonage Video side of things, that’s pretty much it. The audio from the session is streamed to the websocket. There are then endless possibilities for what you do with those audio streams.

Let’s look at one example of what you can do!

### Sample Application Walkthrough

In this demo application, we implement a transcription feature for video calling using [Symbl.ai](https://symbl.ai/)’s [Streaming API](https://docs.symbl.ai/docs/streaming-api). You can check out (and clone) the [full demo app](https://github.com/Vonage-Community/demo-video-node-audio_connector) on our [Vonage Community GitHub org](https://github.com/Vonage-Community). In this post, we’ll walk through some of the key features and components of the app.

Note: this is a high-level walkthrough of some of the key elements of the application rather than an in-depth tutorial. Also, it assumes some knowledge of building an Express-style Node application and some familiarity with how the Vonage Video API works.

#### Application Overview

The first thing we need to do is join the video session by entering a name and clicking on ‘Join’. The name entered will later be used to identify the speaker for the specific audio stream.

![Screenshot of the application UI showing a field to enter a name and a Join button](/content/blog/enhance-your-vonage-video-applications-with-audio-connector/join-call-screenshot.png "Join Call screenshot")

You can then share the meeting link with other participants. Once all the participants have joined the session, clicking on ‘Start Transcription’ will start the transcription process.

Conduct the video call as normal. When you want to see the transcription of the call, clicking the ‘Get Transcription’ button will render a view showing the speaker names and transcribed audio.

![Screenshot of the Transcription View of the application with the participant names and their transcribed audio](/content/blog/enhance-your-vonage-video-applications-with-audio-connector/transcription-view-screenshot.png "Transcription View screenshot")

You can watch a video of the application in action below:

<youtube id="TThQeGthdAo"></youtube>

#### Application Components

The demo is a Node application, and uses the Koa Webserver framework. If you’re already familiar with Express.js, then Koa is fairly similar. The app also uses a bunch of Koa middlewares for functionality such as routing, rendering views, serving static assets and so on. One of the middlewares used is [Koa Websocket](https://github.com/kudos/koa-websocket) library. This library enables you to create and use websockets as part of your overall Koa application.

Additionally the application uses the [Symbl.ai JavaScript SDK](https://docs.symbl.ai/docs/js-sdk) for simplifying interactions with the Symbli.ai streaming API. On the Vonage Video side of things, it uses the Vonage Video [OpenTok Node server SDK](https://tokbox.com/developer/sdks/node/) to handle the server-side API calls, and the [Opentok JavaScript Client SDK](https://tokbox.com/developer/sdks/js/) in our application’s view templates to handle client-side interactions such as publishing and subscribing to video streams.

**The index.js file**

The `index.js` file is the entry point for our application.

At the top of the file we require the various dependencies our app needs to function:

```javascript
require('dotenv').config();

const Koa = require('koa');
const Router = require('@koa/router');
const render = require('koa-ejs');
const path = require('path');
const serve = require('koa-static');
const websockify = require('koa-websocket');
const OpenTok = require("opentok");
```

We then instantiate a new Koa app and enable it for websocket connections, before instantiating a new Koa Router that we will use for our websocket routes. Finally, we make that router available in all of our application’s routes by adding it as a `ws` variable on `app.context`.

```javascript
const app = new Koa();
const socket = websockify(app);
const ws = new Router();
app.context.ws = ws;
```

Next we require some application routes that we’ve defined in other files:

```javascript
const basicHttp = require('./routes/basic');
const symblTranscriptionHttp = require('./routes/symbl/transcription');
```

We then initialize our Vonage Video SDK and Symbl.ai SDK objects using credentials we’ve set as environment variables:

```javascript
const opentok = new OpenTok(process.env.VONAGE_API_KEY, process.env.VONAGE_API_SECRET);
app.context.opentok = opentok;

const symblSdk = require('@symblai/symbl-js').sdk;
app.context.symblSdk = symblSdk;
app.context.transcriptions = [];

symblSdk.init({
  appId: process.env.SYMBL_APP_ID,
  appSecret: process.env.SYMBL_APP_SECRET,
  basePath: 'https://api.symbl.ai'
})
.then(() => console.log('Symbl.ai SDK Initialized.'))
.catch(err => console.error('Error in initialization.', err));
```

In the above code, we also make both SDKs available in all of our application’s routes by adding them to `app.context`, and we do the same with a `transcriptions` array, which we’ll later use to store all of our transcription objects.

We then create a new video session by calling the SDK’s `createSession` method:

```javascript
opentok.createSession({ mediaMode: "routed" }, function (err, session) {
  if (err) throw err;
    app.context.openTokSession = session;
});
```

An important thing to note here is the `mediaMode` for the session is set to `routed`. We can only use the Audio Connector for sessions where the streams are routed via the Vonage Video Media Servers.

We then have some code which sets up the serving of static assets and rendering of view templates (which we won’t detail here), before finally setting up the up to use the routes that have been defined in the routes files as well as the websocket routes which we will later define as part of the transcription functionality. Finally, we initialize the application.

```javascript
app.use(basicHttp.routes()).use(basicHttp.allowedMethods());
app.ws.use(ws.routes()).use(ws.allowedMethods());

app.use(symblTranscriptionHttp.routes()).use(symblTranscriptionHttp.allowedMethods());

app.listen(3000, console.log('Listening on port 3000'));
```

**Routes**

The application’s http routes are defined in a couple of files:

* `/routes/basic.js`
* `/routes/symbl/transcription.js`

We’re not going to explore all of these in detail, since most of these routes just render views. One of the key routes though is the `/transcribe` route. When the ‘Start Transcription’ button is clicked in the video call UI, a POST request is sent to this route, which in turn invokes the `postSymblTranscription` controller action, which we’ll look at next.

**The postSymblTranscription controller action**

This is really the key component in managing the transcription functionality. This controller does a number of things, but the first thing it needs to do is get a list of all the streams being published to the Vonage Video session. It does this by using the Vonage OpenTok Node SDK’s listStreams streams method. This provides an array of objects, with each object representing a stream which is published to the session. 

```javascript
opentok.listStreams(otSession.sessionId, function(error, streams) {

  // rest of code

 });
```

We then iterate through the array of streams, and for each stream perform a certain set of actions. We do this specifically so that we can later identify the speaker for each piece of transcribed audio.

```javascript
streams.forEach(async stream => {
  let stream_id = stream.id;
  let stream_name = stream.name;
  let symblConnection;
  let socketUriForStream = socketURI + '/' + stream_id;

  // 1. start a Symbl.ai realtime streaming request

  // 2. create a websocket on our application

  // 3. request the Audio Connector to start streaming audio to that websocket

 });
```

*Start a Symbl.ai realtime streaming request*

Here we use the `startRealtimeRequest` method of the Symbl.ai SDK to start a streaming request:

```javascript
symblConnection = await symblSdk.startRealtimeRequest({
  id: stream_id,
  speaker: {
		name: stream_name
  },
  insightTypes: \['action_item', 'question'],
  config: {
		meetingTitle: 'My Test Meeting',
		confidenceThreshold: 0.9,
		timezoneOffset: 0, // Offset in minutes from UTC
		languageCode: 'en-GB',
		sampleRateHertz: 16000,
  },
  handlers: {
		onSpeechDetected: (data) => {
	   if (data && data.isFinal) {
	     const {user, punctuated} = data;
	     console.log('Live: ', punctuated.transcript);
	     transcriptions.push({id: user.id, name: user.name, transcription: punctuated.transcript});
	   }
		}
  }
});
```

You can read more about exactly how this works in the [Symbl.ai documentation](https://docs.symbl.ai/docs/code-snippets-streaming-api#streaming-audio-in-real-time). Some things to note though are:

* We set an `id` which is the `stream_id` we obtained from the Vonage Video session
* We define a `speaker` object, with a `name` property, the value of which is the name set for the stream from the Vonage Video session
* We set some `config` options. Of particular interest are the `confidenceThreshold` and `languageCode`, both of which can help improve the accuracy of your transcriptions.
* We define an event handler for the `onSpeechDetected` event. This takes the data returned by the Symbl.ai streaming API and uses that data to populate an object with name and transcription properties which we then push to our `transcriptions` array.

*Create a websocket on our application*

We also need to create individual websocket routes for each of our audio streams:

```javascript
ws.get('/socket/' + stream_id, ctx => {
  let connection = symblConnection;
  console.log(connection);
  ctx.websocket.on('message', function(message) {
		try {
		   const event = JSON.parse(message);
		   if (event.event === 'websocket:connected') {
		     console.log(event);
		   }
		} catch(err) {
		   if (connection) {
		     connection.sendAudio(message);
		     return;
		   }
		}
  });
});
```

We do this by creating a `get` route on our Koa Websocket object with the `stream_id` for that specific audio stream as part of the path for the route. Within the route, we declare a `connection` variable assigned to the symbl stream object for this specific audio stream, to which we then send the message `data` received by the websocket using the Symbl.ai SDK’s `sendAudio` method.

*Request the Audio Connector to start streaming audio*

The final action we need to carry out is to send a request to the Vonage Video API Audio Connector endpoint to start streaming audio for the specific audio stream to the websocket that we’ve defined for that stream:

```javascript
opentok.websocketConnect(otSession.sessionId, token, socketUriForStream, {streams: \[stream_id]}, function(error, socket) {
  if (error) {
		console.log('Error:', error.message);
	} else {
		console.log('OpenTok Socket websocket connected');
  }
});
```

Two of the arguments to the method are key here:

* The `socketUriForStream` which is the websocket URI we earlier created to receive streaming data for this specific audio stream
* The object specifying an array of which `streams` we want to the Audio Connector to stream audio for. Note that there is only one element in the array: the `stream_id` for the specific stream in the current iteration.

The above steps are repeated for each stream in the session, so that each of them is transcribed separately with an identifiable speaker.

**The symbl-transcription Route and View**

The final thing to mention is the `/symbl-transcription` route and its equivalent view template. When the ‘Get Transcription’ button is clicked, this sends a `GET` request to the route, which in turn calls a controller action to render the view. 

The controller action sets a `transcriptions` variable to the array of transcriptions objects, and passes this to the view:

```javascript
exports.getSymblTranscription = (ctx) => {
    let transcriptions = ctx.transcriptions;
    return ctx.render('symbl-transcription', {
    transcriptions: transcriptions
   }
 );
};
```

In the body of the view, each transcription is rendered as a paragraph, showing the name of the speaker and the transcribed text.

```html
<body>
  <h2>Vonage Video Demo - Audio Connector &amp; Symbl.ai: Transcription</h2>
  <% transcriptions.forEach(transcription => { %>
		<p><strong><%= transcription.name %>:</strong>  <%= transcription.transcription %></p>
  <% }); %>
</body>
```

### Next Steps

What would you build with the Vonage Video API Audio Connector? You can use our demo app as a starting point for your own project, or start from scratch if you prefer. If you want to use a language other than JavaScript for your app, the Audio Connector feature is also available in our other [Vonage Video Server SDKs](https://tokbox.com/developer/sdks/server/).

Happy building! If you have any comments or questions, feel free to reach out to us in our [Vonage Developer Slack](https://developer.vonage.com/community/slack).