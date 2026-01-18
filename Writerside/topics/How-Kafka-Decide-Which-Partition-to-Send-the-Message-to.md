# How Kafka Determines Which Partition to Send the Message to?
<primary-label ref="research"/>
<secondary-label ref="kafka"/>
<secondary-label ref="dotnet"/>

<img src="microservicehell-meme.jpg" alt="Wow, my project must use Kafka and Microservices"/>

## Overview

In a distributed system like Kafka, partitions are the unit of parallelism. 
Understanding how a message finds its way to a specific partition is 
crucial for ensuring data ordering and balancing the load across your cluster.

- Kafka version: <code>4.1.X</code>
- Confluent.Kafka Nuget version: <code>2.13.0</code>

https://www.confluent.io/learn/kafka-partition-strategy/

https://www.confluent.io/blog/apache-kafka-producer-improvements-sticky-partitioner/

## Choose a specific partition

In Kafka, choosing which partition to send a message to is decided by the Producer Client.

In .NET, you can manually define a destination using the `TopicPartition` class. 
The example below targets partition `1` of `topic_0`;

<code-block lang="c#">
var topic = "topic_0";
var partition = 1;

producer.Produce(
    new TopicPartition(topic, partition),
    new Message&lt;string, Event&gt;
    {
        Value = new Event { Message = "Hello World!" }
    }
);
</code-block>

<warning>
Sending messages to a specific partition is generally not recommended. 
It bypasses Kafka's load-balancing logic and can lead to "hot partitions," 
where one partition/broker is overwhelmed while others remain idle.
</warning>

## Hashing key

When you provide a Key with your message, Kafka uses a partitioner to map that key to a specific partition. 
This ensures that all messages with the same key always land in the same partition, preserving semantic ordering.

## Random
