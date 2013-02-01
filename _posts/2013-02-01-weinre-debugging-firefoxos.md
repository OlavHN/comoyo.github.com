---
layout: post
author: jan
title: Get your weinre out, remote debugging for Firefox OS
category: FirefoxOS
---

As all mobile developers should know
[Weinre](http://people.apache.org/~pmuellr/weinre/docs/latest/) is the best
thing to have happened to mankind since American Idol got cancelled. It allows
you to connect the Chrome Debugger Tools to a remote mobile device and use
awesomeness like Element inspection, CSS manipulation and javascript execution
to do debugging on steroids. No more `alert` style debugging your mobile
websites. Speed++.

On the other hand Firefox OS, the best thing to have happened since the
invention of in-plane WiFi (guess where I'm writing this blog post), lacks a
proper way to debug applications after deploying them to your shiny new phone.

By combining these two facts, plus given that the Firefox OS UI layer is
completely written in web standards, it's only natural that we at Telenor
Comoyo (ok, ok, Kevin Grandon came up with the original idea) started hacking
on superawesomeincrediblegreat integration of Weinre and Firefox OS. That
means: live viewing and manipulating your app markup; live editing of styles
and live code injection. All from your desktop and directly accessible on your
developer phone. Now try that on any other mobile platform. But Jan, how do we
know that it actually works? Well dear folks, here is a video:

<!--
<iframe width="420" height="315" src="http://www.youtube.com/embed/UiZSEkdAKAA" frameborder="0"></iframe>
-->

<object width="420" height="315"><param name="movie" value="http://www.youtube.com/v/UiZSEkdAKAA?version=3&amp;hl=en_US"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="http://www.youtube.com/v/UiZSEkdAKAA?version=3&amp;hl=en_US" type="application/x-shockwave-flash" width="420" height="315" allowscriptaccess="always" allowfullscreen="true"></embed></object>

##I want it##

Sure, here's how you'll get started:

1. Install [node.js](http://nodejs.org)
2. Install Weinre via `npm install -g weinre`
3. Start Weinre via `weinre --boundHost -all- --httpPort 9090`
4. Check out the Comoyo build Gaia, the UI library for Firefox OS (git clone git//github.com/comoyo/gaia.git)
5. Build Gaia with the Weinre extension: `WEINRE=your.internal.ip:9090 make`
6. Grab the [Firefox OS system](http://ftp.mozilla.org/pub/mozilla.org/b2g/nightly/latest-mozilla-b2g18/)
7. Now start the emulator from your Gaia directory: f.e. on OS/X: `/Applications/B2G.app/Contents/MacOS/b2g-bin -profile $PWD/profile`
8. Go to the [Weinre debug interface](http://localhost:9090/client/#anonymous) on your workstation and see the magic happening.

When you're ready to debug an application, add the following line to your index.html and restart the emulator:

`<script src="shared/js/weinre_app.js"></script>`

**Lessons learned**

For 15 years we built software to develop, design and debug on the web. It's
incredible to see that we can leverage all this existing technology and
integrate them into our Firefox OS build chain with hardly any effort. I you
had any doubt whether Firefox OS was the number one choice for developers, this
is the moment for you to reconsider.

**Our debugging process has also been written up by [Mozilla](https://hacks.mozilla.org/2013/01/remote-debugging-firefox-os-with-weinre/)**
