---
layout: post
title: Backbone.js on WebSockets
author: myrlund
categories: HTML5, JavaScript
---

When we arrived at Comoyo for our summer internship, we were given a pretty exciting task: build a messaging web app, using any technologies at hand, but _without using a backend_. Well, not completely true – we already had a UNIX-socket backend running a slightly unusual JSON-based protocol that we were to hook into, so the problem soon became building a self-contained HTML app able to talk to it without any backend of its own.

The natural platform choice became WebSocket-heavy HTML5.

After [implementing a WebSocket handler in our Netty backend](...) we started thinking about our client-side technology stack. Apart from using WebSockets for server communication, we decided to use localStorage for client-side storage and cache, and Backbone.js for the actual front-end. We decided to go for CoffeeScript instead of pure JavaScript, to speed up development a bit.

# Event-driven web sockets

To let the components talk to each other in a nice and loose fashion, we needed some sort of event dispatch system. Luckily, it turns out Backbone supplies it for absolutely free through its `Backbone.Events` class:

{% highlight coffeescript %}
window.dispatch = _.clone(Backbone.Events)
{% endhighlight %}

Using the event dispatch is easy: you subscribe to and trigger events with `dispatch.on(eventName, payload)` and `dispatch.trigger(eventName, payload)` respectively.

## The server communicator

With event dispatch in place, we naturally started with the bottom layer: the communication layer.

A very simple `Communicator` class wound up looking like this:

{% highlight coffeescript linenos %}
class Communicator

  # The messages used in the protocol look like:
  #   {"com.comoyo.CommandName": {"key": value}}
  # 
  # ...so we keep the namespace handy.
  commandNamespace: 'com.comoyo.'
  
  # Set up the web socket and listen for incoming messages
  constructor: (@server) ->
    @webSocket = new WebSocket(@server)
    @webSocket.onmessage = @handleMessage
    @webSocket.onopen = -> dispatch.trigger('WebSocketOpen')

  handleMessage: (message) =>
  
    # The message string is the message object's data property
    strMessage = message.data
  
    # Now, parse that string
    jsonMessage = JSON.parse(strMessage)
  
    # Grab the command name, ie. the first root key
    fullCommandName = _.keys(jsonData)[0]
    commandName = _.last(fullCommandName.split('.'))
  
    # Trigger the command in event dispatch, passing the payload
    dispatch.trigger(commandName, jsonMessage[fullCommandName])

  sendMessage: (commandName, messageData) =>
  
    # Full command name with namespace
    fullCommandName = @commandNamespace + commandName
  
    # Build the message
    jsonMessage = {}
    jsonMessage[fullCommandName] = messageData
  
    # Serialize the object into a JSON string...
    strMessage = JSON.stringify(jsonMessage)
  
    # ...and send it!
    @webSocket.send(strMessage)
{% endhighlight %}

## The protocol controllers

With our communicator in place, we can set up controllers handling the various parts of communication with the backend protocol.
The controllers communicate through the `Communicator`: they listen to incoming messages through the event dispatch and send messages with `Communicator.sendMessage` calls.

An example? Let's take a look at our login controller.

A simplified version of out login process consists of two simple steps, to be executed in sequence:

1. Client registration
2. Account login

To tie these together, we could build a controller looking like this:

{% highlight coffeescript linenos %}
class LoginController
  
  # Let the initializer listen for relevant events,
  # delegating them to its respective event handlers.
  initialize: ->
    dispatch.on('WebSocketOpen',
                          @sendClientRegistrationCommand, this)
    dispatch.on('ClientRegistrationResponse', 
                          @handleClientRegistrationResponse, this)
    dispatch.on('AccountLoginResponse', 
                          @handleAccountLoginResponse, this)
  
  # The client registration command is a simple message 
  # with some metadata to initiate the connection
  sendClientRegistrationCommand: ->
    payload =
      clientInformation:
        clientMedium: 'web'
    communicator.sendMessage('ClientRegistrationCommand', payload)
  
  # Our client registration response handler should store
  # required metadata and send an account login command
  handleClientRegistrationResponse: (data) ->
    # Store client registration data somewhere handy
    
    # Send account login command
    payload =
      userInformation:
        username: "myrlund"
        password: "ihazpassword"
    
    communicator.sendMessage('AccountLoginCommand', payload)
  
  # Handle the response to our AccountLoginCommand
  handleAccountLoginResponse: (data) ->
    # Store session keys and user data somewhere handy
    
    # Check response data to see if login was successful
    if data.loggedIn
      beHappy()
    else
      tryAgain()
  
{% endhighlight %}

# The localStorage adapter

We want a persistence layer capable of two things: storing data received from the backend controllers, and talking to the front-end part of our app.
Since we've already looked at setting up the controller layer, let's start with the former: _storing data from the backend controllers_.

## Storing data

In case you're not familiar with localStorage, fear not: you don't need to be. It's as simple a key-value store as they come:

{% highlight javascript %}
localStorage.setItem('ourKey', 'someValue')
localStorage.getItem('ourKey') // => 'someValue'
{% endhighlight %}

Let's start by throwing out some sample code.

{% highlight coffeescript %}
class Store
  
  # We keep an in-memory cache to speed up reading of data
  data: {}
  
  # Set this.store to localStorage and load cache
  constructor: (@schemas) ->
    @store = localStorage
    @loadCache()
  
  # We'll call addItems from the backend controllers.
  # 
  # items: object, indexed on unique id.
  #   ex. {"1": {content: "Foo"}, "2": {content: "Bar"}}
  addItems: (schema, items) ->
    
    # Add or overwrite existing items
    _.extend(@data[schema], items)
    
    # Write cache to store
    @save()
  
  # Iterates over keys in cache, serializing
  save: ->
    for key in _.keys(@data)
      @store.setItem(key, JSON.stringify(@data[key]))
  
  # Populates cache with stored data
  loadCache: ->
    for schema in @schemas
      @data[schema] = @fetch(schema) || {}
  
  # Fetches object from store
  fetch: (schema) ->
    JSON.parse(@store.getItem(schema))
  
{% endhighlight %}

As you can see from the above code, we serialize our objects into the store.
This is due to localStorage not being able to store objects natively, but being pretty good at storing strings.

## Talking to Backbone

Although Backbone is designed for AJAX REST APIs out of the box, it supports any kind of backend through [an extremely simple synchronization interface](http://backbonejs.org/#Sync).
We simply set `Backbone.sync` to an object responding to the basic CRUD operations -- _create_, _read_, _update_ and _delete_.


