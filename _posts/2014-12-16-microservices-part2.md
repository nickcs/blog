---
layout: post
title: Reactive Microservice Architecture in Node - Part 2
---

This is the second article in a series on building Reactive Microservices in Node.  If you haven't had a chance to review the [introduction](/2014/11/15/reactive-microservices) article, it should help provide a foundation for the code we will be reviewing in this article.

The services in this article are all based on a single architectual assumption that a message queue is available.  They assume the queue is [AMQP](http://en.m.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) compatible platform like Rabbitmq, Zeromq, Activemq.  As a result we will be using the [node-amqp](https://www.npmjs.com/package/amqp) library that supports most AMQP compatible queue.

Let's get started looking at a simple logging service that can write any logging message that is placed on the queue to a log file.  This service will offer centralized logging to our application architecture simple by listening for messages on a queue with the name of 'log-writer'.  

The log writer depends on the following modules:

```
var config = require('config'),
    amqp = require('amqp'),
    winston = require('winston')
```

* config - The node-config module is an easy way to externalize node configuration settings without having to concern myself with the node environment the application is running in.
* amqp - Provides standard client API for any AMQP compatible queue.  The API is extremely easy to use and flexible enough to meet most of the needs of any project.
* winston - A simple and universal logging library with support for multiple storage options.  Config and Winston make a great combination for externalizing the configuration of log writting.
