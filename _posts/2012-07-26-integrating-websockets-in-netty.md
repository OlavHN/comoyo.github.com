---
layout: post
author: bruun
title: Integrating WebSockets in Netty
category: Backend
published: false
---

Comoyo is more than just a provider of TV and movies online. Behind the scenes, other exciting products are emerging. Together with my partner in crime, [Jonas](https://github.com/myrlund), I was recruited as a student intern for Comoyo Communications in late 2011. There we met a team of eager Comoyans working intensely on the future of communication technology. 

Without going into detail, we are building a messaging service. This consists of a solid backend structure, and several frontends. Our task? Create a web frontend using technologies so fresh that computer hipsters worldwide would worship us!

Enter [WebSockets](http://www.w3.org/TR/websockets/). 

Present in all modern widespread desktop and mobile browsers, with the current exception of Opera ([it's almost here, though](http://www.opera.com/docs/specs/presto2.11/#m211-337)), WebSockets is an independent protocol layered **on top of TCP**, providing a persistent connection between the web site and a server. 

Cue [Netty](https://netty.io/)

> Netty is an asynchronous event-driven network application framework
> for rapid development of maintainable high performance protocol servers & clients.

So, the backend we were to communicate asynchronously with was, in its present form, a TCP socket served by Netty.
Incoming packets were decoded into JSON, and sent further down the chain for processing. Simple.

A simplified overview of how Netty fits into our project is displayed here. 

![Infrastructure](/assets/img/posts/netty/communications_infrastructure.png)

A web frontend for the messaging service *did* exist when we joined the team. However, the site used a [Django](https://www.djangoproject.com/) backend that took care of the communication with Netty, and provided a REST API for the frontend. The goal was to remove the Django node, leaving one less component to maintain.

**Problem:** We couldn't simply open a WebSocket connection to our existing Netty server.

##How Netty works 
A brief workflow description for those new to Netty.

When sending a packet to a Netty server it goes into a [Channel](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/channel/Channel.html) in the form of a [ChannelBuffer](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/buffer/ChannelBuffer.html). The Channel consists of a [Pipeline](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/channel/ChannelPipeline.html) with one or more [ChannelHandlers](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/channel/ChannelHandler.html) in a specified order. 
Incoming packets are sent *up* the pipeline. Outgoing messages are sent *down*. 

As seen below, the existing server first used the `JsonFrameDecoder` to decode the ChannelBuffer into a byte array containing JSON data. Then it passed the result down to `CustomPacketHandler`, an implementation of the [ChannelUpstreamHandler](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/channel/ChannelUpstreamHandler.html) interface. 

{% highlight java %}
Channels.pipeline(
    new JsonFrameDecoder(),
    new CustomPacketHandler()
);
{% endhighlight %}

The last packet handler is extremely simple. 
It sends the JSON byte array to a service communicating with a server that processes the *content* of the packet.

{% highlight java %}
public void messageReceived(
    ChannelHandlerContext ctx, MessageEvent e) throws IOException
{
    byte[] bytes = (byte[]) e.getMessage();
    int connectionId = ctx.getChannel().getId();
    ExternalServerCommunicator.handlePacket(connectionId, bytes);
}
{% endhighlight %}
  
###*So what had to be done differently using WebSockets?*

##Opening the WebSocket connection

When opening a WebSocket connection, you do *not* start by sending WebSocket frames right away! The connection is initialized with a HTTP handshake request, and needs to be upgraded to a WebSocket connection *on the server*. 

Therefore we needed to create our own server instance that would open customized *WebSocket channels*. In other words, we would have to create our own pipeline.

Let's open a connection to a randomly chosen domain using the JavaScript console in the browser.

{% highlight javascript %}
ws = new WebSocket('ws://comoyo.com/');
{% endhighlight %}

Below is the pipeline that the first WebSocket packet is sent through.

{% highlight java %}
ChannelPipeline pipeline = Channels.pipeline(); 
pipeline.addLast("decoder", new HttpRequestDecoder()); 
pipeline.addLast("aggregator", new HttpChunkAggregator(65536)); 
pipeline.addLast("encoder", new HttpResponseEncoder());
pipeline.addLast("handler", new WebSocketPacketHandler()); 
return pipeline;
{% endhighlight %}

Why are we *naming* the handlers, you say? Because replacing them later will be easy as a breeze.

The first three handlers are standard HTTP handlers, and require no further explaination. Do note that encoders do nothing with incoming packets and decoders leave outgoing packets alone. Now we've arrived at the `WebSocketPacketHandler` with our HTTP request.

As before, the incoming packet is passed to the `messageReceived` function, although this time we need to do a bit more work. Our first job is to check what kind of packet this is.

{% highlight java %}
public void messageReceived(
    ChannelHandlerContext ctx, MessageEvent e) throws IOException 
{
    Object msg = e.getMessage();
    if (msg instanceof HttpRequest) {
        handleHttpRequest(ctx, (HttpRequest) msg);
    } else if (msg instanceof WebSocketFrame) {
        handleWebSocketFrame(ctx, (WebSocketFrame) msg);
    }
}
{% endhighlight %}

Since we sent a handshake request over HTTP, we are sent to `handleHttpRequest`, which is show in snippets below. Here we need to perform the handshake. Thankfully we do not need to do this ourselves, Netty provides this for us. We send an error back if the client's handshake request yields an error.

{% highlight java %}
WebSocketServerHandshakerFactory wsFactory = new WebSocketServerHandshakerFactory(
    "ws://" + req.getHeader(HttpHeaders.Names.HOST) + req.getUri(), null, false);
WebSocketServerHandshaker handshaker = wsFactory.newHandshaker(req);

if (this.handshaker == null) {
    wsFactory.sendUnsupportedWebSocketVersionResponse(ctx.getChannel());
} else { 
    handshaker.handshake(ctx.getChannel(), req);
}
{% endhighlight %}

That's it. We've successfully performed the handshake. However, the pipeline of the channel will still handle packets as if they are HTTP. Let's fix that just below the handshake.

{% highlight java %}
ChannelPipeline p = ctx.getChannel().getPipeline(); 
p.remove("aggregator"); 
p.replace("decoder", "wsdecoder", new WebSocketFrameDecoder()); 
p.replace("encoder", "wsencoder", new WebSocketFrameEncoder());
{% endhighlight %}



##Handling WebSocket frames
As mentioned, WebSockets uses a custom frame specification. Is does not replace TCP, it runs *on* TCP, but it can still be concidered to belong at the transport level. A transport protocol on top of a transport protocol. 

For us, this means we have to treat incoming and outgoing packets as WebSockets frames.

So now you have established a link between the browser and the server, and upgraded it to a WebSocket connection. How do you handle the incoming and outcoming packets? Let's start by sending a message to the server from the javascript console.
    
{% highlight javascript %}
ws.send('Hello');
{% endhighlight %}

So the packet arrives at the server, and this time it is passed to the [WebSocketFrameDecoder](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/handler/codec/http/websocket/WebSocketFrameDecoder.html). Magic happens, and a [WebSocketFrame](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/handler/codec/http/websocket/WebSocketFrame.html) object is passed on to the `WebSocketPacketHandler`. This time, finally, it is sent to the "correct" function.
{% highlight java %}
private void handleWebSocketFrame(
    ChannelHandlerContext ctx, WebSocketFrame frame) throws IOException {
    
    if (frame instanceof CloseWebSocketFrame) {
        this.handshaker.close(ctx.getChannel(), (CloseWebSocketFrame) frame);
        return;
    } 
    else if (!(frame instanceof TextWebSocketFrame)) {
        // Preferably do something to handle unsupported frames
        return;
    }

    String request = ((TextWebSocketFrame) frame).getText();
    byte[] bytes = request.getBytes();
    int connectionId = ctx.getChannel().getId();
    ExternalServerCommunicator.handlePacket(connectionId, bytes);
}
{% endhighlight %}

For simplicity I have left out a couple of things, such as handling a `PingWebSocketFrame`. When we are sure that the current frame is a `TextWebSocketFrame`, we extract the text and pass it to the same server communicator as we used before.

So that was the incoming packet. How do we deal with an outgoing packet?
This is up to you. A Netty way of doing it is to implement a [ChannelDownstreamHandler](http://docs.jboss.org/netty/3.2/xref/org/jboss/netty/channel/ChannelDownstreamHandler.html), call it `WebSocketDownstreamHandler`, and add it at the end of the pipeline.
There you can do something like this.

{% highlight java %}
public void handleDownstream(
    ChannelHandlerContext ctx, WebSocketEvent evt) throws Exception {

    final byte[] packetBytes = JsonOutgoingPacketConverter.convert(evt.getMessage());        
    final Channel clientChannel = ctx.getChannel();

    if (clientChannel != null) {
        clientChannel.write(new TextWebSocketFrame(new String(packetBytes)));
    }
}
{% endhighlight %}

Here the `WebSocketEvent` is an implementation of the [ChannelEvent](http://docs.jboss.org/netty/3.2/xref/org/jboss/netty/channel/ChannelEvent.html) interface, and is triggered when we receive a message from the business logic part of our service. 

A JSON object is extracted from the message, converted to a byte array, and wrapped in a WebSocket frame. The `WebSocketFrameEncoder` encodes the message into a ChannelBuffer, and sends it out on the socket.

Hopefully, the packet arrives safely in our browser.
