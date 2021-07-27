Overview :
Kafka is well known for its resiliency, fault-tolerance, and high throughput. But its performance doesn’t always meet everyone’s expectations. In some cases, we can improve it by scaling out or scaling up brokers. While in most cases, we have to play the game of configurations. This document is to come up with some of the best practices and setting of various important configurations on producer or consumer kafka client side. This will help application teams to fine tune their application to perform better and also utilize kafka resources better. Majority of configurations are already pre-defined in a way that they work well for most situations.


We’ll consider four goals which often involve tradeoffs with one another: Throughput, Latency, Durability, and Availability. We have to force teams to discuss the original business use cases and what the main goals are. There are two reasons this discussion is important.

The first reason is that you can’t maximize all goals at the same time. There are occasionally tradeoffs between throughput, latency, durability, and availability. This does not mean that optimizing one of these goals results in completely losing out on the others. It just means that they are all interconnected, and thus you can’t maximize all of them at the same time.

The second reason it is important to identify which service goal you want to optimize is that you can and should tune Kafka configuration parameters to achieve it.
