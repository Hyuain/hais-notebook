---
title: Kafka
date: 2023-03-16 15:12:44
categories:
  - - 前端
tags:
  - S1
---

Kafka is an open-source stream-processing software for collecting, processing, storing, and analyzing data at scale.

<!-- more -->

Kafka can be used in following cases:

- Messaging
- Website Activity Tracking
- Log aggregation
- Stream Processing

# Basic Concepts

**Producer**: **Client applications** that publish events to Kakfa.

**Broker**:

- **A server** which has Kafka running on it.
- Responsible for the communication between multiple services.
- Multiple brokers would form a **Kafka Cluster**.

**Consumer**:

- **Client application** which subscribe to (read and process) these events.
- Consumers can work together as a **Consumer Group**.

**Consumer Group**:

- Multiple consumer groups can read from the same log, each maintaining their position based on the offset they have read / processed.

**Event**:

- An event records the fact that "something happened" in the world or in your business.
- *Headers, Key, Timestamp, Value*.

**Topic**:

- Events are organized and durably stored in topics.
- E.g. "payments", "user details".
- A stream is considered to be a single topic of data (regardless of the number of partitions).

**Offset**:

- Keep a track of which events have already been consumed.
- **An index** pointing to the latest consumed message. 

**Horizontal Scaling**:

- Different partitions may spread over different Kafka brokers.
- Distributed placement of data achieves scalability.

# Partitions

A topic can be further divided into partitions in order to attain higher throughput.

Partitions are spread across the available brokers.

**1 Partition -> Only 1 Consumer in 1 Consumer Group** (ensure the message from one client will be processed in order).

**1 Consumer -> 1 / More Partitions**.

## Partition Key

Messages are written to a partition:

- Append-Only.
- Read in order from beginning to end.

**Events with the same partition key (e.g. a customer or vehicle ID) are written to the same partition.**

- E.g. `hash(key) = key % no_of_partitions`

Kafka guarantees: Any consumer of a given topic-partition will always **read that partition's event in exactly the same order as they were written**.

![Kafka Partition Key](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/KafkaPartitionKey.png)

## Partition Allocation

**2 consumers** in the same consumer group (with the same group ID) have **subscribed to the same topic**:

- None of these 2 consumers would receive the same messages.
- Helps attain consumption rate.

 ![Kafka Partition Allocation](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/KafkaPartitionAllocation-1.png)![Kafka Partition Allocation](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/KafkaPartitionAllocation-2.png)

# Fault-Tolerant

**Group Coordinator**:

- Group Coordinator is a broker responsible for receiving heartbeats from the consumers.
- Triggering a **rebalance** of work whenever consumer is marked as dead.

**Replication**:

- **Leader**: A partition is owned by a single broker in the cluster.
- **Followers**: A replicated partition is assigned to additional brokers. One of the followers can take over leadership if there is a broker failure.
- All producers must connect to the leader in order to publish messages.
- Consumers may fetch from either the leader or one of the followers.

