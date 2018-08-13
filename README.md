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
- Results:
	- Highest Single Producer Throughput: 717360.114778 records/sec (171.03 MB/sec), 8.83 ms avg latency, 226.00 ms max latency
	- 6x Kafka Broker // 3 Producer Aggregate Throughput: 2191825.22 records/sec, 522.57 MB/sec, 9.34 ms avg latency, 225.33 ms avg max latency
	- 6x Kafka Broker // 5 Producer Aggregate Throughput: 3625119.68 records/sec, 864.3 MB/sec, 10.34 ms avg latency, 231.4 ms avg max latency
	- 6x Kafka Broker // 10 Producer Aggregate Throughput: 7212422.22 records/sec, 1719.57 MB/sec, 12.52 ms avg latency, 228.8 ms avg max latency
	- 6x Kafka Broker // 15 Producer Aggregate Throughput: 10556400.74 records/sec, 2516.83 MB/sec, 54.86 ms avg latency, 336.53 ms avg max latency
	- 9x Kafka Broker // 25 Producer Aggregate Throughput: 16254111.4 records/sec, 3875.3 MB/sec, 31.5768 ms avg latency, 437.24 ms avg max latency
