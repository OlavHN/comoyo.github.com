---
layout: post
author: jan
title: Building your first HTML5 app for Firefox OS in 5 minutes!
category: Firefox OS
---
Let'ssssssss get ready to rumble! In this 5 minute introduction we'll get from zero to hero in building a Firefox OS application. Why is this cool? Well, not only is it fun and easy to get started; you can re-use your existing skills as a HTML/CSS/JS wizard, but it's also an opportunity to witness first hand what the [future will look like](https://hacks.mozilla.org/2012/10/creating-the-future-of-mobile-with-firefox-os/).

## Basic principles

An app for Firefox OS is a mobile website. Nothing more, nothing less. However, to build quality apps we need to add features like offline support, and access to certain phone APIs. This is where Firefox OS comes in. A bunch of [new APIs](https://wiki.mozilla.org/WebAPI) have been introduced that address the incapabilities of the current mobile web. The wonderful thing is that it's still 'just a website'. A website that you can also visit on your iPhone or Android device. Just for features where you need access to one of the new APIs Firefox OS gives you an edge. Just like you can create rounded corners in CSS3 that will graceful degrade in IE7.

## Getting the developer tools

To get started on Firefox OS apps you don't need a multi-gigabyte toolchain (I'm looking at you XCode!), but rather the [Firefox](http://www.mozilla.org/en-US/firefox/new/) browser, and the [Firefox OS Simulator](https://addons.mozilla.org/en-US/firefox/addon/firefox-os-simulator/) that runs as a browser plugin.

After installation go to 'Tools' -> 'Web Developer' -> 'Firefox OS Simulator' to get into the simulator dashboard. One little toggle of the switch here will spawn up simulator and you can experience Firefox OS first hand.

![Simulator location in menu](/assets/img/posts/ffos-in-5min/img02.png)

## And now getting that first app up!

So let's get rolling! We can either start of with one of your existing mobile websites, but we can also (kinda) start from scratch by building upon an existing template. For now we'll be using [ffos-list-detail](https://github.com/comoyo/ffos-list-detail); a Comoyo built application template that shows a super simple mobile app powered by AngularJS and bundled with the same [UI library](http://buildingfirefoxos.com/) that powers all system applications.

First things first. We need [node.js](http://nodejs.org/).

Now execute the following commands in the terminal (or cmd.exe on Windows):

```bash
# Clone the template repository
git clone https://github.com/comoyo/ffos-list-detail
# Grab the UI library
git submodule update --init --recursive
# Install server side dependencies
npm install

# AND START...
node server.js
```

We now have built a bare bone application that is excellent at showing a list, and showing a detail view! Go in any modern browser to [http://localhost:8081](http://localhost:8081) to see the app galore!

![The app running in Firefox on the desktop](/assets/img/posts/ffos-in-5min/img01.png)

## Running the app in the simulator

The '/www/manifest.webapp' file is one of the major things introduced by Firefox OS. It's a file with information (icons, description) about your application. This file is being used to publish your app to the Firefox OS Marketplace or when someone installs your app directly on their phones. We can emulate this process in the simulator. In the right column you can specify the location of the manifest file and then install your app right away. Add the following line in the textbox and press 'Add'. 

> http://localhost:8081/manifest.webapp

![Loading the app in the simulator](/assets/img/posts/ffos-in-5min/img03.png)

Your app has now been installed on the device. It will show up immediately in the simulator, or you can start it via the homescreen (swipe to the right). If you have updated your app, press the 'Update' button in the simulator dashboard to reinstall the app and see the changes.

## Debugging! It's as easy as normal websites!

There will be a moment that you want to debug your app. This is easy peasy via the simulator dashboard. When the simulator is running with your app in it, press 'Connect...', and immediately press 'Connect' again. You now have the normal Firefox developer tools available to do javascript debugging on the actual simulator.

![Debugging an app running in the simulator from desktop Firefox](/assets/img/posts/ffos-in-5min/img04.png)

Of course you can always use a desktop browser to debug as long as you don't use any of the WebAPIs.

## And now: go and build!

We have an app set up, it runs in the simulator, and we even have a debugger attached. Time to add some awesomness to the soup! 

1. All available UI elements can be found on [buildingfirefoxos.com](http://buildingfirefoxos.com/) (and they will also run on other devices)
2. You can reach phone capabilites via the WebAPIs that have been described on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/WebAPI), or for a showcase view the [Firefox OS Boilerplate app](https://github.com/robnyman/Firefox-OS-Boilerplate-App).
3. Also visit MDN to learn how to add [offline capabilities](https://developer.mozilla.org/en/docs/HTML/Using_the_application_cache) to your app.

Happy coding!