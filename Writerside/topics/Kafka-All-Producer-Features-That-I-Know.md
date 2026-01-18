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

## End-to-end batch compression

## Retry 

## Transactional Producer
This should be another blog to write


## EnableIdempotence for exactly once delivery

## Guarantee Order delivery

## Compression