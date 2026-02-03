# Every Kafka Producer Feature (That I Know, .NET specificly)
<primary-label ref="draft"/>
<secondary-label ref="kafka"/>
<secondary-label ref="dotnet"/>

<img src="kafka-brainless-meme.jpg" alt="Wow, my project must use Kafka and Microservices"/>

## Overview // TODO
This guide will tightly use .net kafka client

## Produce Messages (LOL obviously)

By default, Kafka Producer sends the message to the lead partition of the topic, which is usually the 1st partition (If you don't have replications). 

Client can configure which partition to send the message to, see <a href="#how-kafka-decide-which-partition-to-send-the-message-to"/>

In .NET Kafka Producer, there are two methods to produce a message <code>Produce</code> and <code>ProduceAsync</code>.

### Here is a quick example:
<tabs>
    <tab title="Produce">
        <code-block lang="c#">
        // TODO code
        </code-block>
    </tab>
    <tab title="ProduceAsync">
        <code-block lang="c#">
        // TODO code
        </code-block>
    </tab>
</tabs>

///////////
In .NET Kafka producer, there are two methods to produce messages
// TODO NEED BETTER FORMAT: ProduceAsync and Produce

By default it send the message to the lead partition of the topic.
You can specify which partition to send the message to.

ProduceAsync

Produce
It runs async, but doesn't return a Task
a bit less overhead than ProduceAsync, because it map more directly to the underlying librdkafka producer API.

## How Kafka Decide Which Partition to Send the Message to? {id="how-kafka-decide-which-partition-to-send-the-message-to"}
you can specify the partition right in the produce message.

If you don't specify the partition, it based on the hashing configuration,

How it decide which partition to send the message to? => Based on Hashing key => need to explain

[//]: # (Partition.Any actully based on // Partitioner = config, just like how we produce message without specifying partition)

[//]: # (public enum Partitioner)

[//]: # ({)

[//]: # (/// <summary>Random</summary>)

[//]: # (Random,)

[//]: # (/// <summary>Consistent</summary>)

[//]: # (Consistent,)

[//]: # (/// <summary>ConsistentRandom</summary>)

[//]: # (ConsistentRandom,)

[//]: # (/// <summary>Murmur2</summary>)

[//]: # (Murmur2,)

[//]: # (/// <summary>Murmur2Random</summary>)

[//]: # (Murmur2Random,)

[//]: # (})

Custom hashing function

## Batching

* When use batching with await ProduceAsync, it waits for the batch to be sent. 
* e.g., if you set batch time is 10s, it will wait for 10s before sending the next batch. this mean each ProduceAsync is a batch

For memory buffer: This setting should correspond roughly to the total memory the producer will use, but is not a hard bound since not all memory the producer uses is used for buffering. Some additional memory will be used for compression (if compression is enabled) as well as for maintaining in-flight requests.

https://kafka.apache.org/41/configuration/producer-configs/#producerconfigs_batch.size

https://kafka.apache.org/41/configuration/producer-configs/#producerconfigs_linger.ms

Each Kafka topic contains one or more partitions. When a Kafka producer sends a record to a topic, it needs to decide which partition to send it to. If we send several records to the same partition at around the same time, they can be sent as a batch. Processing each batch requires a bit of overhead, with each of the records inside the batch contributing to that cost. Records in smaller batches have a higher effective cost per record. Generally, smaller batches lead to more requests and queuing, resulting in higher latency.

A batch is completed either when it reaches a certain size (batch.size) or after a period of time (linger.ms) is up. Both batch.size and linger.ms are configured in the producer. The default for batch.size is 16,384 bytes, and the default of linger.ms is 0 milliseconds. Once batch.size is reached or at least linger.ms time has passed, the system will send the batch as soon as it is able.

At first glance, it might seem like setting linger.ms to 0 would only lead to the production of single-record batches. However, this is usually not the case. Even when linger.ms is 0, the producer will group records into batches when they are produced to the same partition around the same time. This is because the system needs a bit of time to handle each request, and batches form when the system cannot attend to them all right away.

Part of what determines how batches form is the partitioning strategy; if records are not sent to the same partition, they cannot form a batch together. Fortunately, Kafka allows users to select a partitioning strategy by configuring a Partitioner class. The Partitioner assigns the partition for each record. The default behavior is to hash the key of a record to get the partition, but some records may have a key that is null. In this case, the old partitioning strategy before Apache Kafka 2.4 would be to cycle through the topic’s partitions and send a record to each one. Unfortunately, this method does not batch very well and may in fact add latency.

Due to the potential for increased latency with small batches, the original strategy for partitioning records with null keys can be inefficient. This changes with Apache Kafka version 2.4, which introduces sticky partitioning, a new strategy for assigning records to partitions with proven lower latency.

## End-to-end batch compression

## Retry 

## Transactional Producer
This should be another blog to write


## EnableIdempotence for exactly once delivery

## Guarantee Order delivery

## Compression