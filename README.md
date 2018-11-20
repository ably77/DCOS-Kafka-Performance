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

## [1.12] Monitoring Kafka with Prometheus/Grafana

DC/OS 1.12 now ships with Prometheus + Telegraf for improved metrics capabilities. Leveraging Grafana, you can test out building dashboards and monitoring your DC/OS cluster with the guide below

### [Tutorial: Prometheus Framework on DC/OS 1.12](https://github.com/ably77/dcos-se/tree/master/Prometheus/1.12_prometheus)

There are several example dashboards, but below are screenshots of the Kafka dashboard that is available:

**Note:** You do not need to configure the Mesos Telegraf plugin in order to have the Kafka dashboard working

![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/kafka-dashboard1.png)
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/kafka-dashboard2.png)

Save Prometheus options as `prometheus-options.json`:
```
{
  "service": {
    "name": "/monitoring/prometheus"
  }
}
```

Install Prometheus Framework:
```
dcos package install prometheus --package-version=0.1.1-2.3.2 --options=prometheus-options.json --yes
```

Save Grafana options as `grafana-options.json`:
```
{
  "service": {
    "name": "/monitoring/grafana"
  }
}
```

Install Grafana Service:
```
dcos package install grafana --package-version=5.5.0-5.1.3 --options=grafana-options.json --yes
```

Install Marathon-LB:
```
dcos package install marathon-lb --package-version=1.12.3 --yes
```

Install Prometheus MLB Proxy:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/dcos-se/master/Prometheus/1.12_prometheus/prometheus-mlb-proxy.json
```

Run the `findpublic_ips.sh` script:
```
./findpublic_ips.sh
```

Output should similar to below:
```
Public agent node found! public IP is:
52.27.213.225
172.12.3.121


Once all of the services are deployed:
open http://<PUBLIC_AGENT_IP>:9091-94 to access the Prometheus, Alertmanager, PushGateway, and Grafana UI
```


## Results:

Highest Single Producer Throughput:
```
717360.114778 records/sec (171.03 MB/sec), 8.83 ms avg latency, 226.00 ms max latency
```

6x Kafka Broker // 3 Producer Aggregate Throughput:
```
2191825.22 records/sec, 522.57 MB/sec, 9.34 ms avg latency, 225.33 ms avg max latency
```

6x Kafka Broker // 5 Producer Aggregate Throughput:
```
3625119.68 records/sec, 864.3 MB/sec, 10.34 ms avg latency, 231.4 ms avg max latency
```

6x Kafka Broker // 10 Producer Aggregate Throughput:
```
7212422.22 records/sec, 1719.57 MB/sec, 12.52 ms avg latency, 228.8 ms avg max latency
```

6x Kafka Broker // 15 Producer Aggregate Throughput:
```
10556400.74 records/sec, 2516.83 MB/sec, 54.86 ms avg latency, 336.53 ms avg max latency
```

9x Kafka Broker // 25 Producer Aggregate Throughput:
```
16254111.4 records/sec, 3875.3 MB/sec, 31.5768 ms avg latency, 437.24 ms avg max latency
```

## WIP:

A couple improvements can still be made to squeeze more overall performance from your Kafka deployment. Over time I will be adding more detailed information on how to:
- Use a dedicated Kafka-Zookeeper in conjunction with the Kafka Framework
- Isolate disk I/O using `MOUNT` disks
- Maxing out I/O of the attached storage
- Increase JVM heap size
- Testing more storage/networking optimized instances other than m3.xlarge (i.e. r3, h1, i3, and d2 instances in AWS)
- Balancing high throughput in exchange for a balanced profile of characteristics (durability, latency, availability)
