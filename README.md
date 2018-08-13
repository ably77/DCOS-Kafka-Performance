# Load Testing Kafka on DC/OS

The purpose of this guide is to teach users methods of how to tune the performance of Kafka running on DC/OS for goals of throughput, latency, availability, and durability.

This guide was made using the Confluent-Kafka DC/OS Certified Framework, but all commands should work if you replace `confluent-kafka` references with just `kafka`

[Load Testing Kafka Quickstart](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/Quickstart.md)
- A quick guide that will teach the basics of performance testing tools available in the Kafka distribution. 
	- Requires 5 Nodes minimum (1x master, 4x agents)

[Advanced Load Testing Kafka](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/AdvancedTuning.md)
- A deep-dive guide to performance testing and scaling your Kafka cluster on DC/OS. More focused on high throughput, but advice is offered for balancing goals of throughput, latency, availability, and durability
	- Requires 6 nodes minimum (1x master, 4x Kafka agents, 1x Producer Agent)
	- Optional: Test options scale up to 36 nodes maximum (1x master, 10x Kafka Agents, 25 Producer Agents)
