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


Configuration details for goals :


Best Practices for High Throughput :


Producer : 

  Batching of messages. Increase Batch size and batching time. Larger batches requires lesser calls to brokers. Reduces CPU overhead.
      Batch.size
      Linger.ms
      
  Messages to different partitions can be sent in parallel by producers, written in parallel by different brokers, and read in parallel by different consumers. higher number of topic partitions results in higher throughput.
  
  Enable compression.
      compression.type For performance, we generally recommend lz4 and avoid gzip because it can be a CPU hog.
      
  Setting acks=1 makes the leader broker write the record to its local log and then acknowledge the request without awaiting acknowledgment from all followers.
  
  If number of partitions is higher, fine tune buffer.memory. If that memory limit is reached, then the producer will block on additional sends until memory frees up or until max.block.ms time passes.
  
  
Consumer :

Increase fetch.min.bytes, this parameter sets the minimum number of bytes expected for a fetch response from a consumer.  Increasing this will also reduce the number of fetch requests made to the broker, reducing the broker CPU overhead to  process each fetch, thereby also improving throughput.

Use Consumer groups with multiple consumers to parallelize consumption. Parallelizing consumption may improve throughput because multiple consumers can balance the load, processing multiple partitions simultaneously. The upper limit on this parallelization is the number of partitions in the topic.




Best Practices for Low Latency: 

Producer : 

 Trade off for number of partitions. An increased number of partitions may increase throughput. However, there is a tradeoff in that an increased number of partitions may also increase latency. A broker by default uses a single thread to replicate data from another broker, so it may take longer to replicate a lot of partitions shared between each pair of brokers and consequently take longer for messages to be considered committed. No message can be consumed until it is committed, so this can ultimately increase end-toend latency.
 
By default, the producer is tuned for low latency and the configuration parameter linger.ms is set to 0, which means the producer will send as soon as it has data to send.
Disabling compression typically spares the CPU cycles but increases network bandwidth utilization, compression.type=none to spare the CPU cycles.

Acks configuration parameter. By default, acks=1, which means the leader broker will respond sooner to the producer before all replicas have received the message. Depending on your application requirements, you can even set acks=0 so that the producer will not wait for a response for a producer request from the broker, but then messages can potentially be lost without the producer even knowing.

Consumer :

 Two configuration parameters together lets you reason through the size of fetch request, i.e., fetch.min.bytes, or the age of a fetch request, i.e., fetch.max.wait.ms
 
 
Best Practices for High Durability :

Producer : 

Acks is primarily used in the context of durability. To optimize for high durability, we recommend setting it to acks=all (equivalent to acks=-1), which means the leader will wait for the full set of in-sync replicas to acknowledge the message and to consider it committed. This provides the strongest available guarantees that the record will not be lost as long as at least one in-sync replica remains alive.

Try to resend messages if any sends fail to ensure that data is not lost. The producer automatically tries to resend messages up to the number of times specified by the configuration parameter retries (default MAX_INT) and up to the time duration specified by the configuration parameter delivery.timeout.ms (default 120000).

There are two things to take into consideration with these automatic producer retries: duplication and message ordering.

1. Duplication: If there are transient failures in the cluster that cause a producer retry, the producer may send duplicate messages to the broker
2. Ordering: Multiple send attempts may be “in flight” at the same time, and a retry of a previously failed message send may occur after a newer message send succeeded.

Exactly-once semantics - There are two ways to handle this :

             To address both of these, we generally recommend that you configure the producer for idempotency, i.e., enable.idempotence=true, for which brokers track messages using incrementing sequence numbers,  Idempotent producers can handle duplicate messages and preserve message order even with request pipeline there is no message duplication because the broker ignores duplicate sequence numbers, and message ordering is preserved because when there are failures, the producer temporarily constrains to a single message in flight until sequencing is restored. In case the idempotence guarantees can’t be satisfied, the producer will raise a fatal error and reject any further sends, so when configuring the producer for idempotency, the application developer needs to catch the fatal error and handle it appropriately. And with retires > 1.

                                    OR

To handle possible message duplication if there are transient failures in the cluster, be sure to build your consumer application logic to process duplicate messages. To preserve message order while also allowing resending failed messages, set the configuration parameter max.in.flight.requests.per.connection=1 to ensure that only one request can be sent to the broker at a time. To preserve message order while allowing request pipelining, set the configuration parameter retries=0 if the application is able to tolerate some message loss.

3.  When a producer sets acks=all (or acks=-1), then the configuration parameter min.insync.replicas specifies the minimum threshold for the replica count in the ISR list. If this minimum count cannot be met, then the producer will raise an exception. When used together, min.insync.replicas and acks allow you to enforce greater durability guarantees.

