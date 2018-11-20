# Load Testing and Tuning Kafka on DC/OS

The purpose of this guide is to teach users methods of how to tune the performance of Kafka running on DC/OS for goals of throughput, latency, availability, and durability using guidelines and best-practices provided by Confluent

See [Optimizing Your Apache Kafka Deployment](https://www.confluent.io/white-paper/optimizing-your-apache-kafka-deployment/) for the full whitepaper discussing how to optimize your Apache Kafka deployment for various service goals.

## Apache Kafka

### [Load Testing Apache Kafka Quickstart](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/Quickstart-kafka.md)
- A quick guide that will teach the basics of performance testing tools available in the Kafka distribution.
        - Requires 5 Nodes minimum (1x master, 4x agents)

### [Advanced Load Testing Apache Kafka](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/AdvancedTuning-kafka.md)
- A deep-dive guide to performance testing and scaling your Kafka cluster on DC/OS. More focused on high throughput, but advice is offered for balancing goals of t$
        - Requires 6 nodes minimum (1x master, 4x Kafka agents, 1x Producer Agent)
        - Optional: Test options scale up to 36 nodes maximum (1x master, 10x Kafka Agents, 25 Producer Agents)

## Confluent Kafka

### [Load Testing Confluent Kafka Quickstart](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/Quickstart-cpkafka.md)
- A quick guide that will teach the basics of performance testing tools available in the Kafka distribution.
	- Requires 5 Nodes minimum (1x master, 4x agents)

### [Advanced Load Testing Confluent Kafka Framework](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/AdvancedTuning-cpkafka.md)
- A deep-dive guide to performance testing and scaling your Kafka cluster on DC/OS. More focused on high throughput, but advice is offered for balancing goals of throughput, latency, availability, and durability
	- Requires 6 nodes minimum (1x master, 4x Kafka agents, 1x Producer Agent)
	- Optional: Test options scale up to 36 nodes maximum (1x master, 10x Kafka Agents, 25 Producer Agents)

## WIP:

A couple improvements can still be made to squeeze more overall performance from your Kafka deployment. Over time I will be adding more detailed information on how to:
- Use a dedicated Kafka-Zookeeper in conjunction with the Kafka Framework
- Isolate disk I/O using `MOUNT` disks
- Maxing out I/O of the attached storage
- Increase JVM heap size
- Testing more storage/networking optimized instances other than m3.xlarge (i.e. r3, h1, i3, and d2 instances in AWS)
- Balancing high throughput in exchange for a balanced profile of characteristics (durability, latency, availability)
