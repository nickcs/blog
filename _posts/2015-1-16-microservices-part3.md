---
layout: post
title: Reactive Microservice Architecture in Node - Part 3
---

This is the third article in the series on building Reactive Microservices in Node.  If you haven't had a chance to review the [initial](/2014/11/15/reactive-microservices) article, it should help provide a foundation for the code we will be reviewing in this article.

The services in this article are all based on a single architectural assumption that a message queue is available.  They  assume the queue is an AMQP broker which I discussed in detail in a previous post [AMQP](/2014/12/15/reactive-microservices-amqp).  As a result we will be using the [node-amqp](https://www.npmjs.com/package/amqp) library that supports most AMQP compatible queue.

## Log writer

Let's get started looking at a simple logging service that can write any logging message that is placed on the queue to a log file.  This service will offer centralized logging to our application architecture simple by listening for messages on a queue with the name of 'log-writer'.  

The entire source for this service can be found in the related github project [log-writer](https://github.com/nickcs/rabbitmq-proto/tree/master/log-writer).

The log writer depends on the following modules:

```javascript
var config = require('config');
var amqp = require('amqp');
var winston = require('winston');
```

* `config` - The node-config module is an easy way to externalize node configuration settings without having to concern myself with the node environment the application is running in.
* `amqp` - Provides standard client API for any AMQP compatible queue.  The API is extremely easy to use and flexible enough to meet most of the needs of any project.
* `winston` - A simple and universal logging library with support for multiple storage options.  Config and Winston make a great combination for externalizing the configuration of log writting.

Next we setup the configurable options of the service:

```javascript
var subscriptionName = process.env.SUBNAME || config.get('subscriptionName');
var exchangeOptions = config.get('exchangeOptions');
var host = process.env.HOST || config.get('host');
```

* `subscriptionName` -
* `exchangeOptions` - Exchange options are defined in the configuration file to allow for control over how the app create the exchange within the queue at deployment time.  You can find the details of the options in the [amqp docs](https://github.com/postwait/node-amqp#connectionexchange)
* `host` -

The last step of the application initialization is to setup the logger and connect to the queue:

```javascript
var logName = subscriptionName + '.log';
winston.add(winston.transports.File, {filename: logName});

var connection = amqp.createConnection({host: host});
connection.on('ready', function(){
  ...
});
```
