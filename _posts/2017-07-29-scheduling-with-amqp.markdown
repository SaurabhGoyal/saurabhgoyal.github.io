---
layout: post
title:  "Scheduling a task - for dummies"
subtitle: A high level view of why and how of scheduling
date:   2017-07-29 23:18:07 +0530
categories: engineering
---
What -
===
`Basic concept is that there is a work at hand and there is some related task to be done, but not right away. Some situations when scheduling a task may be needed instead of doing right away are -`
* When the task is not to be done now because of lower priority, un-availability of required resource, complexity of task, cost of running the task, ```priority of the main work at hand``` or various other reasons. Then you schedule that task to be done ```later```.

* When the task is to be distributed to one or more other people who'll do it. You need a common platform to communicate about the task to them. You schedule that task to be ```queued``` in their list of work.

* When the task is to be repeated at a certain frequency such as daily or weekly. You schedule that task to be run ```periodically```.

Now how do we schedule? Let's look at the components involved. At it's core, a task is nothing but a set of instructions to be performed on a particular data set. That is the data you need to pass to schedule a task. The combined data can be called a `Message` which is passed from the scheduler to whoever will perform that task.

How -
===

Protocol
---
We need a protocol (a set of rules or a contract) to do that in efficient and inter-operable way in machines (read- computers). The protocol used for that is called `Messaging Protocol`. One of the most common messaging protocol that we use or rather did use in our daily lives is post office service, where we have to provide destination, source, add required stamps and enclose the contents in a wrapper for the messaging to be completed. In computers, one such protocol is SMTP (Simple Mail Transfer Protocol), which powers the email service.

``If you are wondering why a separate protocol, let's just think this way - a protocol is nothing but a refined version of whole or parts of one or more other protocols customized for one specific purpose.``

Followings are the benefits of using a specific messaging protocol over other such as HTTP/IMAP -
* HTTP is a synchronous protocol and works on a point to point connection which means whenever a message is sent from the sender, it'll be received by only one receiver and in synchronous mode, i.e. if there is any interruption before the data has been completely received by the receiver, data will be lost. This may be fine for web applications where client(receiver) will request again, but is not suitable for messaging which requires guarantee of message delivery to one or more clients which may or may not be available at the time of sending the message,

* HTTP has high dependency of sender and receiver on each other and each part of communication highly depends on proper understanding and knowledge of both. This prevents the consumers, publishers and other parts from being independent of each other and anywhere and with any level of capacity.

* Messaging protocols are created for messaging only and hence they optimize on connection establishment and request-response cycle in multiple ways, making it the solution for this purpose.

As most messaging protocol make use of asynchronous execution of sub-parts such as publishing, routing and consuming, they are named asynchronous messaging protocol. There are various such protocols - AMQP, MQTT etc.

Components & Process -
---
Now that you have the message and a protocol to send it, what's missing? `Consistency` and `Reliability`. In simpler terms, We need to ensure -
* The message does not get damaged/altered in any way before the person gets it.

* The message reaches the intended person.

* And most importantly, the task in the message gets executed successfully before discarding the message.

For the above, one good approach is having a `message storage` and a `storage-manager` to prevent losses and manage routing and message persistence in case of errors. So there are-
* Messages to be delivered which consist of the task to be performed and the data on which it's to be performed.

* One or more publishers that produce messages and

* One or more consumers which consume messages.

* Message storage - Queue is the data structure used mostly for this.

* Manager - A program/service/middleman that links all parts and facilitates the messaging.

This may sound simple but the whole benefit of message reliability, interoperability between all kinds of resources because of common standard, scalability in terms of messages, publishers and consumers and the dependency removal between publishers and consumers is all that async messaging protocols bring to the plate.


In general, There are two popular patterns for messaging -

* ```Publisher Subscriber``` - Publishers publish messages for certain topics. Consumers subscribe for certain topics. Whenever a new message is available for a topic, all subscribed consumers get that message.

* ```Message Queue``` - Publishers add message to a queue and consumers consume that queue. A message is consumed by only one consumer.

Let's Talk AMQP-
===
AMQP (Asynchronous Message Queuing Protocol) is a specific protocol that uses the components mentioned above and adds some more.

![AMQP Model-1]({{ site.url }}/assets/images/The-amqp-model-for-wikipedia.svg)
*<sub><sup>https://commons.wikimedia.org/wiki/File:The-amqp-model-for-wikipedia.svg</sup></sub>*

The store-manager/middleman is called ```Message Broker```.

In a nutshell, Broker maintains one or more ```queues``` where all incoming messages are stored until they have been successfully consumed as expected. These queues may be in-memory or persistent. The messages can be added directly to the queues by broker or another layer called ```exchange``` be added which controls and manages which messages are to be added to which queue based on rules called ```binding```.

The consumers are called ```workers``` (Their work is to consume the queue). They may be increased or decreased based on load on the queue. They can be configured to consume only certain types of queues thus to distribute consumption of different kind of tasks to different workers.

![AMQP Model-2]({{ site.url }}/assets/images/exchanges-bidings-routing-keys.png)
*<sub><sup>https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html</sup></sub>*


Popular names -
===

* ```Celery -``` Just a client to be used as a producer to assign messages to broker or consume messages from broker. It basically is an interpreter between your application and broker and uses AMQP to interact with broker, so that your application does not have to do all that. Other similar solutions are - ```pika```, ```kafka``` etc.

* ```RabbitMQ -``` A broker that manages queues and interacts with publisher and consumer. Has highly reliable and comprehensive features to ensure message delivery and prevent any loss. Other similar solutions are - ```ActiveMQ```, ```Amazon SQS```, ```kafka``` etc.

* ```Redis-``` A data store that stores key-value pairs, supports certain types of data-structures as values and performs very well for high speed requirements. All data is stored in memory and hence is often used as cache. Other similar solutions are - ```memcached```, ```mongodb``` etc. __Why this is here__ - It has recently added support for publisher-consumer pattern through AMQP and hence is also getting popular as a broker.


---
References:-
* [Digital Ocean AMQP](https://www.digitalocean.com/community/tutorials/an-advanced-message-queuing-protocol-amqp-walkthrough){:target="_blank"}
* [RabbitMQ Tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-python.html){:target="_blank"}
