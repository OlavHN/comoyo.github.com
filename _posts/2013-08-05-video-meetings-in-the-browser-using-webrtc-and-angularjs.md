---
layout: post
author: [johan, vegard, johnedvard]
title: Video meetings in the browser made simple using WebRTC and AngularJS
category: WebRTC
---

TLDR; Test our new video meeting service in your browser at appear.in - built on WebRTC and AngularJS. No installs or login.

<object width="500" height="281"><param name="movie" value="//www.youtube.com/v/FWPNQAoatvg?hl=en_US&amp;version=3"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="//www.youtube.com/v/FWPNQAoatvg?hl=en_US&amp;version=3" type="application/x-shockwave-flash" width="500" height="281" allowscriptaccess="always" allowfullscreen="true"></embed></object>

We at Comoyo have been interested in [WebRTC](http://www.w3.org) for a while now, reading about it and experimenting with what is actually possible to do with it. Since our employees work in five different locations (Oslo, Trondheim, Stockholm, Amsterdam, Munich), the need for efficient communication tools is omnipresent. We have tested a lot of video conferencing solutions, who all have their advantages and disadvantages. 

So, we thought, why not combine our interest in WebRTC with our need for an easier way to set up video meetings? As summer interns in Comoyo, we were assigned with the task of building an easy-to-use web conference system using WebRTC and HTML5. We were not really sure what would come out of this, but the result turned out to be so good that we decided to open it up to public testing in a beta version.

appear.in was created to achieve one goal: **to make it really easy to start a video meeting.** To achieve this, we allow users to create a new video room in one click from the frontpage. You add people to the conversation by sharing the URL with them. When they open the URL, they will be taken straight into the room and asked to turn on their webcam and microphone. 

##Features in the beta version##
Like most developers, we do a lot of whiteboard drawing. But to our horror, none of the communication channels we have used to date, have an easy way to draw together right then and there. So to be able to share drawings, while still being able to share screens at the same time (because no one has the time to switch back and forth between tabs), we have often ended up doing this:

<img src="/assets/img/posts/appear-in/whiteboard.JPG" alt="Manual whiteboard drawing in video conference">

This is of course less than ideal, so one of the first conclusion from our planning was that we wanted to make a video meeting software with a shared whiteboard feature.

Other features that we have included in this initial basic feature set include: 
- Name your own room (or use a really cool predefined one, generated from a list of 2000 adjectives and animal types)
- Muting of local video and audio
- Fullscreen support
- Several different layouts for different types of conferences
- Drag and drop video windows to move them around
- Screen sharing

##How did we build it? (The technical stuff)## 
Since WebRTC builds on peer-to-peer connections, we all agreed that the service should have as little of a reliance of a central server as possible. But at the same time, we knew we’d have to have some signaling functionality to/from said server, as specified in the [W3C specifications](http://www.w3.org/TR/webrtc/): 

<cite>“Communications are coordinated via a signaling channel which is provided by unspecified means, …”</cite>

So we ended up with this:
<img src="/assets/img/posts/appear-in/simple.png" alt="signaling drawing appear.in">

When working with WebRTC a lot of the functionality is hidden from the developer, given that, the only part in the above image that needed work was the signaling. For example, creating a stream is:

	getUserMedia({video : true , audio : true}, successCallback, errorCallback);

and then waiting for the callback.

Peer Connections are also supposed to be almost as simple, after adding streams and doing the signaling, it is just supposed to work. And when it’s fully implemented in the browsers, it might. But as of right now there are more than a few bugs in different implementations that require workarounds.

##What goes into making appear.in work##
The architecture of appear.in has two different parts to it, a frontend which is the app that resides within your browser, and a backend.

The frontend of appear.in is built using AngularJS, Socket.io, WebRTC, Twitter Bootstrap 3 and HTML5, while the backend is built on Node.js using Express and Socket.io. In the end we get a technology stack looking something like this:

<img src="/assets/img/posts/appear-in/techstack.png" alt="architecture drawing appear.in">

We are using Angular for a multitude of things including routing, writing custom directives, as well as of course data binding. Using Angular with WebRTC is then easy peasy.

	function addNewStream(newStream){
	$scope.streams.push(URL.createObjectURL(newStream));
	}

in your javascript to push the objectURL to a variable named streams in your scope. Then in your html you just have to do:

	<div ng-repeat=”stream in streams”>
			<video ng-src=stream autoplay=”true”>
			</video>
	</div>

And just like that, you get a video element with a stream for every video you add using our javascript function. If you have created your layouts with the help of Angular directives, you can just change it to:

	<div ng-repeat=”stream in streams” splitscreen=”true”>
			<video ng-src=stream autoplay=”true”>
			</video>
	</div>

and get it to be a part of your split screen layout. But we will go more into this in another blog post. 

##Moving forward with appear.in##
We will be working constantly with adding more features and fixing bugs. We know that there are lots of bugs and that it does not work in all scenarios and with various network limitations, but we still want to release it to see if this is something people want us to continue working on. 

With this beta release we want people to test the service by using it as they normally would use a video conference or video calling solution. We would love to hear about the situations you are using it in, and how it is working for you! 

Feedback regarding call quality can be given in the form that appears when you click leave room and will be sent directly to us. 

For any feedback, feature requests and/or bug reports, please contact us at [feedback@appear.in](mailto:feedback@appear.in), [Twitter](https://twitter.com/appear_in) or [Facebook](https://www.facebook.com/appear.in.video). 
 