---
layout: post
title: Reactive Microservice Architecture in Node - AMQP
---

This is the second article in the series on building Reactive Microservices in Node.  If you haven't had a chance to review the [initial](/2014/11/15/reactive-microservices) article, it should help provide a foundation for the code we will be reviewing in this article.

This series is based on using the [AMQP](http://en.m.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) protocol for message queueing.  Now is a good time to provide an overview of the protocol and a couple code examples in node.

## AMQP

The AMQP core architectural components are: publisher, exchange, queue, and consumer. As you may have guessed, the publisher is the data producer which pushes messages to an exchange. The exchange is a routing engine which is responsible for delivering the messages to the right queues (exchanges never store messages). For example, a message may need to be routed to just a single queue (direct exchange), maybe the message should be forwarded to every queue (pubsub) in the list (fanout exchange), or perhaps the message should be routed based on a key (topic exchange).

![AMQP Image](/public/amqp-diagram.png)

## Introduction to RabbitMQ

RabbitMQ is an AMQP message broker. It simply accepts messages from one or more endpoints `Producers` and sends it to one or more endpoints `Consumers`.

RabbitMQ can be configured to figure out what needs to be done when a consumer crashes(store and re-deliver message), when consumer is slow (queue messages), when there are multiple consumers (distribute work load), or even when RabbitMQ itself crashes (durable). For more please see: [RabbitMQ tutorials](http://www.rabbitmq.com/getstarted.html).

### Fundamental pieces of RabbitMQ

We will be using the publish/subscriber pattern.

![RabbitMQ Pub/Sub Image](/public/rabbitmq-pubsub.png)

* Producer (`P`) – Sends messages to an exchange along with `Routing key` indicating how to route the message.
* Exchange (`X`) – Receives message and Routing key from Producers and figures out what to do with the message.
* Queues(`Q`) – A temporary place where the messages are stored based on Queue’s `binding key` until a consumer is ready to receive the message. Note: While a Queue physically resides inside RabbitMQ, a consumer (`C`) is the one that actually creates it by providing a `Binding Key`.
* Consumer(`C`) – Subscribes to a Queue to receive messages.

### Routing Key, Binding Key and types of Exchanges

To allow various work-flows like pub-sub, work queues, topics, RPC etc., RabbitMQ allows us to independently configure the type of the Exchange, Routing Key and Binding Key.

#### Routing Key:

A string/constraint from Producer instructing Exchange how to route the message. A Routing key looks like: `logs`, `errors.logs`, `warnings.logs` `tweets` etc.

#### Binding Key:

Another string/constraint added by a Consumer to a queue to which it is binding/listening to. A Binding key looks like: `logs`, `*.logs`, `#.logs` etc.

Note: In RabbitMQ, Binding keys can have `patterns` (but not Routing keys).

#### Types of Exchange:

Exchanges can be of 4 types:

* Direct – Sends messages from producer to consumer if Routing Key and Binding key match exactly.
Fanout – Sends any message from a producer to ALL consumers (i.e ignores both routing key & binding key)
* Topic – Sends a message from producer to consumer based on pattern-matching.
* Headers – If more complicated routing is required beyond simple Routing key string, you can use headers exchange.
* In RabbitMQ the combination of the type of Exchange, Routing Key and Binding Key make it behave completely differently. For example: A fanout Exchange ignores Routing Key and Binding Key and sends messages to all queues. A Topic Exchange sends a copy of a message to zero, one or more consumers based on RabbitMQ patterns (`#`, `*`).

Going into more details is beyond the scope of this blog, but here is another good blog that goes into more details: [AMQP 0-9-1 Model](http://www.rabbitmq.com/tutorials/amqp-concepts.html) Explained

## Publishing & Consuming AMQP in Node

The type of exchange, message parameters, and the name of the attached queue can all contribute to the delivery and routing behavior of the message. However, for the sake of example, let's create a simple pubsub fanout exchange in Node with the  [node-amqp module](https://github.com/postwait/node-amqp) to talk to a RabbitMQ broker.

```javascript
//Connect to broker and get reference to the connection.
var amqp = require('amqp');
var amqpConn = amqp.createConnection({host: 'localhost'});

//create a fanout exchange on the broker
amqpConn.on('ready', function () {
    amqpConn.exchange(
      'multicast',
      {'type': 'fanout'},
      function(exchange) {
        // publish multiple messages to fanout
        exchange.publish('hello');
        exchange.publish('world');
      }
    );
});
```

In order to consume the messages from an exchange the consumer needs to create a queue and then bind it to an exchange. A queue can be durable (survive between server restarts), or auto-deletable for cases when the queue should disappear if the consumer goes down. Best of all, once the queue is bound to an exchange, the messages are streamed to the client in real-time via a persistent connection (no polling!):

```javascript
var amqp = require('amqp');
var amqpConn = amqp.createConnection({host: 'localhost'});

amqpConn.on('ready', function () {
  // bind 'listener' queue to 'multicast' exchange
  amqpConn.queue('listener',function(queue){
    queue.bind('multicast','');
    queue.subscribe(function(msg){
      // process your message here
    }
  }
}
```

## Advanced AMQP Recipes

The flexibility of the message and the exchange model is what makes AMQP such a powerful tool. Whenever a publisher generates a message, he can mark it as `persistent` which will guarantee delivery through the broker - if there is an attached queue, it will accumulate messages until the consumer requests them. However, if you're streaming transient data (access logs, for example), you can also disable message persistence and not worry about overwhelming your broker. That's how you achieve `exactly-once` vs `at least once` semantics.

* Pub/Sub - Create a fanout exchange and attach as many queues as you want, each consumer will receive a copy of the message.
* Load balancing - Bind two workers to the same queue and the broker will automatically round-robin the messages (there is no upper limit on the number of workers).
* Failover - By default the AMQP broker does not require a message to be ACKed by a consumer, but with a simple configuration flag the messages will be kept on the server until the ACK is received. If the consumer goes down without ACKing a message, they will be automatically put back on the queue.
* Key based routing - Topic exchange allows partial matching based on a message key that is set by the producer.
* Notify the producer of no subscribers - Set the immediate flag on the message and the broker will do all the work. Best of all, you can also compose these patterns to cover virtually any delivery use case!
