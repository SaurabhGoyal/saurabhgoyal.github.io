---
layout: post
title:  "Scheduling a task - for dummies"
subtitle: A high level view of why and how of scheduling
date:   2017-07-29 23:18:07 +0530
categories: tech-blog
---
Nothing solidifies your learning about a topic more than explaining it to a dummy who is totally un-familiar with the topic. I tried learning about scheduling tasks in a web application that we have been using in a project but could not get far. I tried it again recently and it seems I have something that I can share.

`To begin with, the concept of scheduling a task is not limited to web applications. It can be used anywhere in any context. Basic concept is that there is a work at hand and there is some task related to that work. Few situations when scheduling the task may be needed instead of doing right away are -`
* When the task is not to be done now because of lower priority, un-availability of required resource, complexity of task, cost of running the task, ```priority of the main work at hand``` or various other reasons. Then you schedule that task to be done ```later```.

* When the task is to be distributed to one or more other people who'll do it. You need a common platform to communicate about the task to them. You schedule that task to be ```queued``` in their list of work.

* When the task is to be repeated at a certain frequency such as daily or weekly. You schedule that task to be run ```periodically```.

Now when we have decided to schedule a task, we need a protocol (a set of rules or a contract) to do that in efficient and inter-operable way in machines (read- computers). At it's core, a task is nothing but a set of instructions to be performed on a particular data set. That is the data you need to pass to schedule a task. The combined data can be called a `Message` which is passed from the scheduler to whoever will perform that task. The protocol used for all that is called `Messaging Protocol`. One of the most common messaging protocol that we use or rather did use in our daily lives is post office service, where we have to provide destination, source, add required stamps and enclose the contents in a wrapper for the messaging to be completed. In computers, one such protocol is SMTP (Simple Mail Transfer Protocol), which powers the email service.

``If you are wondering why a separate protocol, let me just clarify here, a protocol is nothing but a refined version of whole or parts of one or more other protocols customized for one specific purpose.``

Now that you have the message and a protocol to send it, what's missing? `Consistency` and `Reliability`. To explain better-
* Are we sure our message will not be damaged/altered in a ny way before the person gets it?

* Are we sure our message will reach the intended person?

For the above, one good approach is queuing the messages to prevent losses. Currently scheduling has become synonymous to __*Message Queuing*__ where there are-
* Messages to be delivered which consist of the task to be performed and the data on which it's to be performed.

* One or more publishers that produce messages and

* One or more consumers which consume messages.

There are two popular patterns for achieving above-

* ```Publisher Subscriber``` - Publishers directly publish messages to the pre-decided subscribers.

* ```Message Queue``` - Publishers add message to a queue and subscribers consume that queue.

The queue pattern is what's used mostly because of it's flexibility. Message queuing uses one popular protocol - AMQP. In above steps, all the interaction happens using AMQP so let's discuss that.

---

What's AMQP -
---
AMQP stands for Asynchronous Messaging Queue Protocol. It's just a kind of language(read - communication protocol) that machines or processes can use to interact with one another. Other such languages you might have heard of are -
* ```HTTP``` Mainly used for interaction among web-applications.

* ```SSH``` Mainly used for interaction among machines.

* ```FTP``` Mainly used for interaction among file servers/clients.

* ```IMAP``` Mainly used for interaction among email servers/clients.

Each of the above and other communication protocols have been designed and optimized for a specific kind of task and have become as good as to be accepted as the default standards for those specific tasks.


Why not HTTP or IMAP, Why AMQP-
---
Followings are the benefits of AMQP which also explains why such a protocol was even needed at the first place and how it is better than using HTTP and customizing that-
* HTTP is a synchronous protocol and works on a point to point connection which means whenever a message is sent from the sender, it'll be received by only one receiver and in synchronous mode, i.e. if there is any interruption before the data has been completely received by the receiver, data will be lost. This may be fine for web applications where client(receiver) will request again, but is not suitable for following cases-

  * Where delivery of a message is of utmost importance. Each sent message is crucial and can not be lost.

  * Where there may be multiple clients(receivers) which themselves are available at different times.

* HTTP is a simple server-client protocol which does not scale well for multiple senders and receivers such that they are working with same set of data/messages to be sent or received.

* HTTP has high dependency of sender and receiver on each other and each part of communication highly depends on proper understanding and knowledge of both. This prevents the consumers, publishers and the message queue from being independent of each other and anywhere and with any level of capacity.

* AMQP was created for messaging only and hence optimizes on connection establishment and request-response cycle in multiple ways, making it the solution for this purpose.

AMQP Solution-
---
What AMQP provides is a basic contract that ensures certain way of interaction among publishers and consumers. Another main part of solution is a common ground/a service/a server/a middleman that takes all the messages from publishers and manages them and let consumers consume them as and when they are available. This middleman is called ```Message Broker``` as mentioned earlier. This may sound simple but, the whole benefit of message reliability, interoperability between all kinds of resources because of common standard, scalability in terms of messages, publishers and consumers and the dependency removal between publishers and consumers is all that AMQP brings to the plate.

![AMQP Model-1]({{ site.url }}/assets/images/The-amqp-model-for-wikipedia.svg)
*<sub><sup>https://commons.wikimedia.org/wiki/File:The-amqp-model-for-wikipedia.svg</sup></sub>*

In a nutshell, Broker maintains one or more ```queues``` where all incoming messages are stored until they have been successfully consumed as expected. These queues may be in-memory or persistent. The messages can be added directly to the queues by broker or another layer called ```exchange``` be added which controls and manages which messages are to be added to which queue based on rules called ```binding```.

The consumers are called ```workers``` (Their work is to consume the queue). They may be increased or decreased based on load on the queue. They can be configured to consume only certain types of queues thus to distribute consumption of different kind of tasks to different workers.

![AMQP Model-2]({{ site.url }}/assets/images/exchanges-bidings-routing-keys.png)
*<sub><sup>https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html</sup></sub>*


Popular names -
---

* ```Celery -``` Just a client to be used as a producer to assign messages to broker or consume messages from broker. It basically is an interpreter between your application and broker and uses AMQP to interact with broker, so that your application does not have to do all that. Other similar solutions are - ```pika```, ```kafka``` etc.

* ```RabbitMQ -``` A broker that manages queues and interacts with publisher and consumer. Has highly reliable and comprehensive features to ensure message delivery and prevent any loss. Other similar solutions are - ```ActiveMQ```, ```Amazon SQS```, ```kafka``` etc.

* ```Redis-``` A data store that stores key-value pairs, supports certain types of data-structures as values and performs very well for high speed requirements. All data is stored in memory and hence is often used as cache. Other similar solutions are - ```memcached```, ```mongodb``` etc. __Why this is here__ - It has recently added support for publisher-consumer pattern through AMQP and hence is also getting popular as a broker.


---
References:-
* [Digital Ocean AMQP](https://www.digitalocean.com/community/tutorials/an-advanced-message-queuing-protocol-amqp-walkthrough){:target="_blank"}
* [RabbitMQ Tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-python.html){:target="_blank"}
