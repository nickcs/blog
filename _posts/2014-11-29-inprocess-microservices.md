---
layout: post
title: Single Process Microservices in Node
---

As part of my series on Reactive Microservices in Node I wanted to address the programming model vs the deployment model. Several ideas presented in this series are founded on a post named [Messaging as a programming model](http://eventuallyconsistent.net/2013/08/12/messaging-as-a-programming-model-part-1/).  In Messaging as a programming model, the author discusses the value of using messaging as away to separate concerns and decouple software interfaces. In Reactive Microservices in Node, I explain the use of this programming model in node and how it can be used to deploy software in a microservice architecture. However, it is important to understand that I will explore how this model can be deployed in a single executable as well.  This article explores the ideas behind the separation of the programming model from the deployment model.

Since the dawn of software architecture, we mostly knew that ‘Monolithic Architecture’ was more or less a synonym for [ball of mud architecture](http://en.m.wikipedia.org/wiki/Big_ball_of_mud). It mostly referred to code which was so entangled and coupled that it could not be separated out into components (or layers, or modules, or libraries, or equivalent). Examples include God classes, lack of information hiding, and failures of Modularity. Changing one thing should not require cascading changes in the rest of the codebase.

Recently the term Monolith has been taken to mean having a Single Deployment Target at runtime. This is a quite different meaning.

The point is that you are at liberty in pretty much any operating system, runtime or language devised in the last 30 years, to structure your code and components as carefully and modularly as you like, while choosing your runtime deployment scenario independently of that modularity. It’s okay for 2 uncoupled components to run on the same machine; this happens by design in every operating system.

The CTO at intilery.com showed me a couple of years ago how their server codebase can be deployed as either a single .WAR file for a single webserver or split as separate .WARs for separate machines by flicking a switch in the build config.

It’s not rocket science, it’s Separation of Concerns: the codebase is not the runtime.

To this point, the code we explore in this series will show you how to deploy it in a multi-process microservice architecture and a single application instance. Node requires a little more work than just flicking a build switch but I will show you how it is easily achievable.
