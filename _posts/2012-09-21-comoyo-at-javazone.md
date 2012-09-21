---
layout: post
author: markusbk
title: Comoyo at Javazone 2012
category: Test
---
[Javazone](http://jz12.java.no/) is one of the biggest conferences for software developers in Scandinavia, and took place in Oslo this September. Comoyo was naturally present both in the audience and on the stage, where we shared some of the experiences we have gathered after 1,5 years of delivering film & TV streaming services.

Below you will find videos of the presentations made by Comoyo employees. If you have any questions, feedback or any experience you want to share on these topics, please comment below. 

##Real-World Performance Testing in AWS##
By [Bjørn Remseth](https://github.com/la3lma), Senior Software Engineer (Comoyo) and [Kristian Klette](https://github.com/klette), Solutions Engineer (Iterate)

Comoyo provides infrastructure for moving pictures, both movies and TV, as well as live television. On March 23, 2012, the Norwegian national soccer league started up, and we were in no way certain that we would be able to handle the load we expected all the football fans in Norway to generate. The movie delivery subsystem is mostly, but not entirely, consisting of components hosted in Amazon's Elastic Compute Cloud (EC2), and mostly (but not entirely) written in Java. In this talk we describe how we used multiple tools to generate what we believed to be realistic loads against our systems, and how we used this to tune the system to actually perform.

This may seem like a simple task, but it wasn't: The system is made of components, but the component developers were not aware of the full system architecture.

Some components thought to "just work" (such as load balancers) were discovered to be a bottleneck. Initially, we didn't even think about load balancers, since they were effectively invisible for the developers. Writing test scripts for load testing is a skill separate from writing other types of tests, and the test tools themselves are much less well standardized or developed and much easier to use for some tasks than others.

We did all of this, we learned how to harness the tools, we delivered the Tippeliga, and we did it all in just a few weeks. In this talk we share some of the hard won lessons on how to write tests, interpret their results, and how to actually tune a system for high performance.

<iframe src="http://player.vimeo.com/video/49484335" width="500" height="281" frameborder="0">test</iframe> 

<p><a href="http://vimeo.com/49484335">Real-World Performance Testing in AWS</a> from <a href="http://vimeo.com/javazone">JavaZone</a> on <a href="http://vimeo.com">Vimeo</a>.</p>

##Reliable scalability with MongoDB##
By [Markus Krüger](https://github.com/markusbk), Senior Software Engineer

Many developers are discovering that traditional relational databases make it hard to scale to the large data volumes and user traffic required by Internet-scale applications. MongoDB is sailing up as one of the leading contenders in the NoSQL space. We at Comoyo are using MongoDB on Amazon Elastic Cloud Compute (EC2) for our payment subsystems, with a target of some hundred millions users regularly accessing the system. As we are handling financial transactions with high demands on reliability, we needed to make sure that MongoDB did not sacrifice our customers' safety on the altar of performance. This talk presents how we use MongoDB to gain higher availability and scalability than traditional databases, with simpler development and administration, without losing the required reliability and durability.

The talk describes how we configured replication, and our approach to work around the lack of transactions in MongoDB. There will be pointers on tuning MongoDB for availability and reliability. Also, the talk will describe how we used MongoDB to implement once-and-only-once messaging semantics on top of Amazon Simple Queuing Service (SQS).

<iframe src="http://player.vimeo.com/video/49368447" width="500" height="281" frameborder="0">test</iframe> 

<p><a href="http://vimeo.com/49368447">Reliable Scalability with MongoDB</a> from <a href="http://vimeo.com/javazone">JavaZone</a> on <a href="http://vimeo.com">Vimeo</a>.</p>

##Cloudname, a system for coordinating services##
By [Haakon Dybdahl](https://github.com/dybdahl), Senior Software Engineer

Cloudname is an open source Apache licensed system for running services in the cloud (available from github.com/cloudname). The system consists of two parts: Nodee, which is a very simple service for provisioning and managing software artifacts and running processes in a distributed system. The other part is the Cloudname library, which is used for coordination of services in a distributed system. (Usually when we refer to Cloudname, we mean the Cloudname library).

The Cloudname library aims to provide a minimal set of coordination and configuration services needed to run large distributed systems. It takes care of mapping and tracking services independently of physical their physical location. Services are assigned coordinates. When a service starts it will claim its coordinate. Once a coordinate has been claimed the service can use this to publish endpoints. Clients can resolve endpoints by way of resolver expressions.

It is an important point that Cloudname is implemented as a library that uses Apache ZooKeeper to perform its heavy lifting.

<iframe src="http://player.vimeo.com/video/49372218" width="500" height="281" frameborder="0">tekst</iframe> 

<p><a href="http://vimeo.com/49372218">Cloudname, a system for coordinating services</a> from <a href="http://vimeo.com/javazone">JavaZone</a> on <a href="http://vimeo.com">Vimeo</a>.</p>

We look forward to next year’s Javazone and other conferences, and hope to see you there! 

