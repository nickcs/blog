---
layout: post
title: Reactive Microservice Architecture in Node - Part 2
---

This is the second article in a series on building Reactive Microservices in Node.  If you haven't had a chance to review the [introduction](/2014/11/15/reactive-microservices) article, it should help provide a foundation for the code we will be reviewing in this article.

The services in this article are all based on a single architectual assumption that a message queue is available.  They assume the queue is [AMQP](http://en.m.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) compatible queues like Rabbitmq, Zeromq, Activemq.  As a result we will be using the [node-amqp](https://www.npmjs.com/package/amqp) library that supports most AMQP compatible queue.

