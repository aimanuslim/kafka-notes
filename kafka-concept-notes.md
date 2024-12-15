- [How many partitions should I have for my topic?](#how-many-partitions-should-i-have-for-my-topic)
- [How many replication factor?](#how-many-replication-factor)
- [How many brokers should I have](#how-many-brokers-should-i-have)
- [acks and their effect](#acks-and-their-effect)
- [Notes on serialization](#notes-on-serialization)
- [Kafka Properties tuning](#kafka-properties-tuning)
  - [flush.offset.interval.ms](#flushoffsetintervalms)
  - [vm.dirty\_ratio](#vmdirty_ratio)
  - [xfs instead of ext4](#xfs-instead-of-ext4)
- [Kafka Connect](#kafka-connect)
  - [Kafka Connect Task rebalancing](#kafka-connect-task-rebalancing)
  - [Kafka Sink and Source Connector Optimization notes](#kafka-sink-and-source-connector-optimization-notes)
    - [Source connectors](#source-connectors)
      - [manage offset dynamically](#manage-offset-dynamically)
      - [offset.flush.interval.ms](#offsetflushintervalms)
      - [acks.all](#acksall)
    - [Sink connectors](#sink-connectors)
      - ["delivery.guarantee": "exactly\_once",](#deliveryguarantee-exactly_once)
- [Producers](#producers)
  - [Notes on producers](#notes-on-producers)
  - [Relationship between `linger.ms`, `batch-size`, `record-queue-time-avg` and `request-latency-avg`](#relationship-between-lingerms-batch-size-record-queue-time-avg-and-request-latency-avg)
- [Consumer offsets](#consumer-offsets)
- [Metrics for GMS](#metrics-for-gms)
  - [DEV CC dashboard ](#dev-cc-dashboard-)
  - [Availability metrics -  from splunk](#availability-metrics----from-splunk)


# How many partitions should I have for my topic?

Dont start with 1000

If you have less than 6 brokers, takes something like Nx3
If you have more than 6 brokers, take something like Nx2

High number of partitions mean more consumers.

# How many replication factor?

at least 2 in production
dont go crazy, and test it out. 

# How many brokers should I have
If N is your replication factor for topics, then you can get N-1 nodes to faile for kafka to work.

# acks and their effect
- acks=0 - do not wait for replies from the brokers/server
- acks=1 - wait for the leader to ack
consquence: if leader is down is unelected yet, then data can be lost.
- acks=all - wait for all replicas to ack   - is the safest, but slowest since have to wait for all replicas to ack.

# Notes on serialization

- avro enables evolution of schema without breaking, but corresponding method to get the missing fields will return null
- avro requires that the deserializer has access to the schema during the time application wrote the message even if the consumer application is using a different schema

# Kafka Properties tuning
## flush.offset.interval.ms
## vm.dirty_ratio
## xfs instead of ext4

# Kafka Connect

## [Kafka Connect Task rebalancing](https://docs.confluent.io/platform/current/connect/index.html?_gl=1*1bdygni*_gcl_aw*R0NMLjE3MzE1NTk1MDQuQ2p3S0NBaUF1ZEc1QmhBUkVpd0FXTWxTalBZSmwxLV9ySXo1a0V6UGlObjZXUXJJUkZGQXlkSk5HSVVjRzVsZGZOZGt5VHZpUXZNc2RCb0N1T01RQXZEX0J3RQ..*_gcl_au*MTQ2ODYxMTgzLjE3Mjk3NDAwMzE.*_ga*OTM0MzMzOTE2LjE2OTAxODMwMTQ.*_ga_D2D3EGKSGD*MTczMTU1ODcxNy4yMjAuMS4xNzMxNTU5NTQwLjQuMC4w&_ga=2.38465699.1257346619.1731558717-934333916.1690183014&_gac=1.120311418.1731559503.CjwKCAiAudG5BhAREiwAWMlSjPYJl1-_rIz5kEzPiNn6WQrIRFFAydJNGIUcG5ldfNdkyTviQvMsdBoCuOMQAvD_BwE#task-rebalancing)


## [Kafka Sink and Source Connector Optimization notes](https://www.codefro.com/2024/08/28/mastering-kafka-connect-advanced-source-and-sink-configurations/)

### Source connectors
#### manage offset dynamically
#### offset.flush.interval.ms
#### acks.all

### Sink connectors
#### "delivery.guarantee": "exactly_once",

# Producers

## Notes on producers

- producer decides which partitions to write in advance, not brokers
- if key=null, then round robin style to partition



## Relationship between `linger.ms`, `batch-size`, `record-queue-time-avg` and `request-latency-avg`
- Low linger.ms Value:

When you set a low linger.ms value (e.g., 0 milliseconds or a very small value), it means the producer will try to send records to Kafka almost immediately after they are produced by the application.
This results in minimal waiting time for records in the producer's record queue before they are sent to Kafka.
As a result, the record-queue-time-avg metric will also be low because records spend very little time in the queue.

- Higher linger.ms Value:

When you set a higher linger.ms value (e.g., 100 milliseconds or more), the producer will accumulate records in its internal queue for a longer period before sending them to Kafka.
Records will spend more time waiting in the producer's record queue before being transmitted to Kafka.
Consequently, the record-queue-time-avg metric will increase because records now spend more time on average in the queue due to the longer batching interval.

Batching Behavior:

The batch.size setting determines the maximum number of messages or records that can be included in a single batch before the producer sends them to the Kafka broker.
When batch.size is set to a larger value, the producer accumulates more messages in each batch before sending them. This can lead to fewer network requests to Kafka and potentially reduce network overhead.
Conversely, when batch.size is set to a smaller value, the producer sends smaller batches more frequently.

Request Size and Latency:

Smaller batches (lower batch.size) typically result in smaller network requests sent to Kafka, which can have a lower impact on the network latency.
Larger batches (higher batch.size) can lead to larger network requests, which may take slightly longer to transmit over the network.
Overall Latency Impact:

The indirect impact of batch.size on request-latency-avg is related to how it affects the timing and size of producer requests to Kafka.
Smaller batches (lower batch.size) may lead to lower request latency on the producer side because individual batches are smaller and may be sent more frequently.
Larger batches (higher batch.size) may introduce slightly higher request latency on the producer side because it waits longer to accumulate a larger batch before sending.

# Consumer offsets
Consumer always send
- topic
- partition
- offset

to fetch data from a topic

# Metrics for GMS
## [DEV CC dashboard ](https://ors-obs.signalfx.com/#/dashboard/FhkVGnCAEAA?groupId=FhHlyJbAEAA&configId=FhkVGnHAAAA&startTime=-1h&endTime=Now)
## Availability metrics -  from splunk
- active connection count
- 
