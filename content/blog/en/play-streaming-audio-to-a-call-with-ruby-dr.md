---
title: Play Streaming Audio to a Phone Call with Ruby
description: Learn how to create a new phone call and send streaming audio to it
  with the Nexmo Voice API and Ruby with just a few lines of code.
thumbnail: /content/blog/play-streaming-audio-to-a-call-with-ruby-dr/Stream-Audio-into-a-Phone-Call-with-Ruby.png
author: ben-greenberg
published: true
published_at: 2019-01-24T20:46:22.000Z
updated_at: 2021-05-12T02:27:07.211Z
category: tutorial
tags:
  - ruby
  - voice-api
comments: true
redirect: ""
canonical: ""
---
In this post, we are going to look at streaming an audio file into an in-progress phone call. There are many use cases for streaming audio files into calls, and with the Nexmo Voice API and the Nexmo Ruby gem, it is a relatively straightforward process. Whether you want to share music with someone over the phone or a snippet from a work meeting, you can do so with a few lines of Ruby and Nexmo.

The source code for this blog post is available on [GitHub](https://github.com/nexmo-community/ruby-voice-stream-audio-demo).

## Prerequisites

To work through this post, you'll need:

<sign-up number></sign-up>

* Ruby 2.5.3 or newer 
* The [dotenv](https://github.com/bkeepers/dotenv) gem 
* [Sinatra](https://github.com/sinatra/sinatra) 
* [Nexmo CLI](https://github.com/Nexmo/nexmo-cli)

## Configure your Nexmo Account and Application

Let's start by purchasing a phone number we can use for testing. We'll use the Nexmo CLI to purchase the number.

```bash
nexmo number:buy --country_code US
```

Go ahead and save that phone number you just purchased. We will be using it in just a moment in our code. 

We'll also use the Nexmo CLI to create a new application. We need to create an application to use the Voice API. 

```bash
nexmo app:create "Test Application 1" http://example.com/answer http://example.com/event --keyfile private.key
```

Executing that command, we have now created a new application called "Test Application 1" and defined three parameters: 

* `answer_url`: Where our application delivers the Nexmo Call Control Object (NCCO) that governs the call 
* `event_url`: Where Nexmo sends the event information asynchronously when the `call_status` changes
* `keyfile`: This is where we store the private key that we need for authentication. 

We will redefine both the `answer_url` and the `event_url` in our code. It is important to note that they need to be **externally accessible** URLs for the Nexmo platform to access. For our local installation, we can use ngrok to make our local server externally available. You can find more information on using Nexmo with ngrok at [this blog post](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/).

The `nexmo app:create` command also provides us with an application ID, which we will use in a moment in our code, alongside the new phone number we acquired.

## Setting up Our Credentials

At this point, we now have the information we need to create our application. We are going to create a Sinatra web server that serves four routes, three `GET` requests and a `POST` request. The `GET` requests initiate the phone call, provide the NCCO instructions to the Nexmo platform and streams a silent file to keep the call open. The `POST` request streams the audio to the call once it has been answered.

To access our Nexmo account and to authenticate ourselves we need to provide the correct credentials. You should take care to store these securely and avoid checkling them into source control. In this example, we are going to use the `dotenv` gem to store our credentials as environment variables and access them in our code accordingly. We do that by first creating a file called `.env` and putting the following inside:

```
NEXMO_API_KEY=
NEXMO_API_SECRET=
NEXMO_APPLICATION_ID=
NEXMO_APPLICATION_PRIVATE_KEY_PATH=private.key
NEXMO_TO_NUMBER=
NEXMO_NUMBER=
```

Your `NEXMO_API_KEY` and `NEXMO_API_SECRET` were provided to you when you signed up for a Nexmo account at the beginning of this walkthrough. You can reaccess them in your [account settings](https://dashboard.nexmo.com/settings). Your `NEXMO_APPLICATION_ID` is what was returned to you when you created a new application with the Nexmo CLI. Likewise, your `NEXMO_APPLICATION_PRIVATE_KEY_PATH` is the file path to the file you specified when initializing a new application with the CLI. In our example, it is `./private.key`. Your `NEXMO_NUMBER` is the phone number you purchased, and the `NEXMO_TO_NUMBER` is the phone number you wish to call. 

If you are committing this to GitHub, please make sure also to create a `.gitignore` file and adding `.env` inside of it to ensure that your credentials do not get published online by accident.

## Sending Streaming Audio to a Call with Ruby

Let's create a new file called `server.rb` and make sure we are requiring our necessary dependencies and loading our `dotenv` environment variables:

```ruby
require 'sinatra'
require 'nexmo'
require 'dotenv'
require 'json'

Dotenv.load
```

Next, we are going to initialize a new Nexmo client instance utilizing those credentials and the Nexmo gem.

```ruby
client = Nexmo::Client.new(
  api_key: ENV['NEXMO_API_KEY'],
  api_secret: ENV['NEXMO_API_SECRET'],
  application_id: ENV['NEXMO_APPLICATION_ID'],
  private_key: File.read(ENV['NEXMO_APPLICATION_PRIVATE_KEY_PATH'])
)
```

At this point, we now have a credentialed Nexmo client instance and our streaming audio defined. What are we missing? The phone call of course! 

First, let's get our `ngrok` externally accessible URL that we will use to make our server available to Nexmo to receive our call instructions from and to send call status updates back to. Once you have installed `ngrok`, open a new console window and execute `ngrok http 4567` from the command line. This will initiate a new `ngrok` server on port 4567 that parallels our Sinatra server, which is also running on port 4567. You will see a status displayed to you that contains the `ngrok` URL under the `Forwarding` parameter. Go ahead and copy and paste that URL into the `BASE_URL` constant variable in `server.rb`.

```ruby
BASE_URL = 'PASTE URL HERE'
```

While we are defining our variables, let's go ahead and set the path to our streaming audio file, and we'll define it as a `CONSTANT` variable.

```ruby
AUDIO_URL = "#{BASE_URL}/voice_api_audio_streaming.mp3"
```

We now can create a URL path to `/new` that when accessed will start the phone call.

```ruby
get '/new' do
  response = client.calls.create(
    to: [{ type: 'phone', number: ENV['NEXMO_NUMBER'] }],
    from: { type: 'phone', number: ENV['NEXMO_TO_NUMBER'] },
    answer_url: ["#{BASE_URL}/answer"],
    event_url: ["#{BASE_URL}/event"]
  )
  puts response.inspect
end
```

The `get /new` action above creates a new phone call with the `to:`, `from:`, `answer_url:` and `event_url:` parameters provided. 

The `to:` and `from:` parameters require both a `type` and a `number`. In our case, the `type` for both is `phone`. The `answer_url` and `event_url` need to be externally accessible URLs that the Nexmo platform can receive instructions from and send call status updates to, respectively. 

Once we initiate the phone call, Nexmo accesses the `answer_url` path looking for call instructions. We need to create that action as well. The action will render and deliver the instructions as JSON. The instructions provide the `action`, the URL path to a silent audio file that will play in the background keeping the connection open and a `loop` parameter, which we set to `0`.

```ruby
get '/answer' do
    # Provide the Nexmo Call Control Object (NCCO) as JSON to the Nexmo Platform
    content_type :json
    [{ :action => 'stream', :streamUrl => ["#{BASE_URL}/stream/silent"], :loop => 0 }].to_json
end
```

At this point, we need to create one more `GET` path to deliver the silent audio file that we specified in the NCCO instructions:

```ruby
get '/stream/silent' do
    # Stream a silent file in the background to keep the call open
    send_file File.join(settings.public_folder, 'silence.mp3')
end
```

Lastly, we also create a `POST` path to `/event` that receives the call status from the Nexmo platform and send the streaming audio once it has been answered. The action parses the data received and uses the `status` parameter to determine when the call has been answered. A call's status changes from `started` to `ringing` and then `answered` and finally, when it is disconnected, to `completed`. Our code plays the audio file once the `status` has reached `answered`. We also receive the unique identifier for this call, the `uuid`, from the data received, which is required for us to inject our streaming audio into this specific call.

```ruby
post '/event' do
  data = JSON.parse(request.body.read)
  response = client.calls.stream.start(data['uuid'], stream_url: [AUDIO_URL]) if data['status'] == 'answered'
  puts response.inspect
end
```

So with that `POST` action, we have finished creating our application. Now, we just need to run `ruby server.rb` from the command line to boot up our server and make sure we initialized ngrok to make our local server externally accessible. Once that is done, go ahead and navigate to `http://localhost:4567` from your web browser and you will receive the phone call. When you answer the call, the audio will begin streaming. 

## Conclusion

In approximately 40 lines of code, we have created a fully functioning web server that can call any phone number and play streaming audio into that call. There is a lot more that can be done with the Nexmo Voice API and you can explore it entirely on the [Nexmo Developer Platform](https://developer.nexmo.com/voice/voice-api/overview).

If you have any questions about this post feel free to email devrel@nexmo.com or [join the Nexmo community Slack channel](https://developer.nexmo.com/community/slack), where we're waiting and ready to help.