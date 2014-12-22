---
layout: post
title: Single Process Microservices in Node
---

As part of my series on Reactive Microservices in Node I wanted to address the programming model vs the deployment model.  Several ideas presented in this series are founded on a post named [Messaging as a programming model](http://eventuallyconsistent.net/2013/08/12/messaging-as-a-programming-model-part-1/).  In Messaging as a programming model, the author discusses the value of using messaging as away to separate concerns and decouple software interfaces.  In Reactive Microserives in Node, I explain the use of this programming model in node and how it can be used to deploy software in a microservice architecture. However, it is important to understand that I will explore how this model can be deployed in a single executable as well.  A single executable being defined as the entire code base runs in a single process.  This article explores the ideas behind the separation of the programming model from the deployment model.

Since the dawn of software architecture, we mostly knew that ‘Monolithic Architecture’ was more or less a synonym for [ball of mud architecture](http://en.m.wikipedia.org/wiki/Big_ball_of_mud). It mostly referred to code which was so entangled and coupled that it could not be separated out into components (or layers, or modules, or libraries, or equivalent). God classes, lack of information hiding, a change in one place means 20 other changes. That kind of thing. A failure of Modularity.

Recently the term Monolith has been taken to mean having a Single Deployment Target at runtime. This is a quite different meaning.

The point is that you are at liberty in pretty much any operating system, runtime or language devised in the last 30 years, to structure your code and components as carefully and modularly as you like, whilst choosing your runtime deployment scenario independently of that modularity: it’s okay for 2 uncoupled components to run on the same machine. Honestly. *nix does it all the time. Ooh, so does Windows. And .Net and Java and Android and iOS and ….

The CTO at intilery.com showed me a couple of years ago how their server codebase can be deployed as either a single .war for a single webserver or split as separate .wars for separate machines by flicking a switch in the build config.

It’s not rocket science, it’s Separation of Concerns: the codebase is not the runtime.

To this point, the code we explore in this series will show you how to deploy it in a multi-process microservice architecture and a single application instance.  Node requires a little more work than just flicking a build switch but I will show you how it is easily achievable.
