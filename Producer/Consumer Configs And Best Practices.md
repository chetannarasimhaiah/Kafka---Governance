Overview :

Kafka is well known for its resiliency, fault-tolerance, and high throughput. But its performance doesn’t always meet everyone’s expectations. In some cases, we can improve it by scaling out or scaling up brokers. While in most cases, we have to play the game of configurations. This document is to come up with some of the best practices and setting of various important configurations on producer or consumer kafka client side. This will help application teams to fine tune their application to perform better and also utilize kafka resources better. Majority of configurations are already pre-defined in a way that they work well for most situations.


We’ll consider four goals which often involve tradeoffs with one another: Throughput, Latency, Durability, and Availability. We have to force teams to discuss the original business use cases and what the main goals are. There are two reasons this discussion is important.

The first reason is that you can’t maximize all goals at the same time. There are occasionally tradeoffs between throughput, latency, durability, and availability. This does not mean that optimizing one of these goals results in completely losing out on the others. It just means that they are all interconnected, and thus you can’t maximize all of them at the same time.

The second reason it is important to identify which service goal you want to optimize is that you can and should tune Kafka configuration parameters to achieve it.

Goals : 


There are mainly four goals or criteria we are going to consider. 

High Throughput :
Which is the rate that data is moved from producers to brokers or brokers to consumers? Some use cases have millions of writes per second. Because of Kafka’s design, writing large volumes of data into it is not a hard thing to do.

Low latency : 
Which is the elapsed time moving messages end to end (from producers to brokers to consumers)? One example of a low-latency use case is a chat application, where the recipient of a message needs to get the message with as little latency as possible.

High durability :
Durability is all about reducing the chance for a message to get lost. The most important feature that enables durability is replication, which ensures that messages are copied to multiple brokers. which guarantees that messages that have been committed will not be lost? One example use case for high durability is an event streaming microservices pipeline using Kafka as the event store. Another is for integration between an event streaming source and some permanent storage (e.g., AWS S3) for mission-critical business content.

High availability :
To optimize for high availability, you should tune Kafka to recover as quickly as possible from failure scenarios. which minimizes downtime in case of unexpected failures? Kafka is a distributed system, and it is designed to tolerate failures. In use cases demanding high availability, it’s important to configure Kafka such that it will recover from failures as quickly as possible.
