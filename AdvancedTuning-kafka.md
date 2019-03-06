# Advanced Load Testing Kafka
Lets take our prior example and expand on it. We're going to try to change up some parameters and see what performance we get

## Prerequisites
To start, the specs of my cluster are as stated below:
- DC/OS 1.11
- 1 Master
- 5 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine

- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM [See here for more recommended instance types by Confluent](https://www.confluent.io/blog/design-and-deployment-considerations-for-deploying-apache-kafka-on-aws/)

### Default Kafka Framework Parameters
Note that the default Kafka package has these specifications for brokers:
- 3x Brokers
- 1 CPU
- 2048 MEM
- 5GB Disk
- 512 MB JVM Heap Size

**For our Advanced Guide we will use a larger Kafka cluster CPU/MEM to observe performance improvements:**

### Our Advanced Kafka Framework Parameters
- 3x Brokers
- 3 CPU
- 12GB MEM
- 5 GB Disk
- Log Retention Minutes - 1 min so we dont fill up the disk
- 512 MB JVM Heap Size

### If you have an Existing Kafka Deployment

If you were following the Quickstart guide before this, we deployed the default Kafka framework with the specs listed above.

Note that the Kafka framework does not support changing the volume requirements after initial deployment in order to prevent accidental data loss from reallocation. Instead we are just going to set the `log_retention_minutes` to a low number so that Kafka clears the logs before allocated disk fills up.

If you did not clone this repo, save below as `kafka-options.json`
```
{
	"service": {
		"name": "kafka"
	},
	"brokers": {
		"cpus": 3,
		"mem": 10000,
		"heap": {
			"size": 512
		},
		"disk": 25000,
		"count": 3
	},
	"kafka": {
		"log_retention_minutes": 2
	}
}
```

## Step 1: Install Kafka
```
dcos package install kafka --options=options-kafka.json --yes
```

If you have kafka already installed, update kafka by using:
```
dcos package update kafka --options=options-kafka.json --name=kafka --yes
```

Validate kafka Installation:
```
dcos kafka plan status deploy --name=kafka
```

Output should look like below when complete:
```
$ dcos kafka plan status deploy
deploy (serial strategy) (COMPLETE)
└─ broker (serial strategy) (COMPLETE)
   ├─ kafka-0:[broker] (COMPLETE)
   ├─ kafka-1:[broker] (COMPLETE)
   └─ kafka-2:[broker] (COMPLETE)
```

## Step 2: Add a test topic from the DC/OS CLI
```
dcos kafka topic create performancetest --partitions 10 --replication 3
```

Output should look similar to below:
```
$ dcos kafka topic create performancetest --partitions 10 --replication 3
{
  "message": "Output: Created topic \"performancetest\".\n"
}
```

## Step 3: Set up Proper Monitoring
DC/OS 1.12 now ships with Prometheus + Telegraf for improved metrics capabilities. Leveraging Grafana, you can test out building dashboards and monitoring your DC/OS cluster with the guide below

![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/kafka-dashboard1.png)
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/kafka-dashboard2.png)


### Install Prometheus and Grafana

Install the DC/OS Monitoring Package:
```
dcos package install beta-dcos-monitoring --yes
```

The DC/OS Monitoring package comprises of Prometheus, Alertmanager, PushGateway, and Grafana all in a single DC/OS Catalog framework.

To monitor the installation:
```
dcos beta-dcos-monitoring plan status deploy --name=dcos-monitoring
```

Once the deployment is complete, you should be able to access the grafana UI through Adminrouter by running the command below:
```
open `dcos config show core.dcos_url`/service/dcos-monitoring/grafana/
```

### Importing Dashboards

Select the + button --> import:
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/import2.png)

Paste the Grafana.com dashboard url or id

Reference Dashboard IDs:
- 1.12 DC/OS Kafka Dashboard - ID: 9018 - URL: https://grafana.com/dashboards/9018

Edit your Dashboard Name and Select Data Source and Import:
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/import3.png)

### Run the Kafka Performance Test
Now lets run the same single producer Kafka performance test optimized for throughput as before on our 3 broker node Kafka cluster

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/1producer-lower-topic-performancetest.json
```

Example Output in Logs:
```
10000000 records sent, 347089.653257 records/sec (82.75 MB/sec), 33.86 ms avg latency, 589.00 ms max latency, 14 ms 50th, 93 ms 95th, 104 ms 99th, 192 ms 99.9th.
```

Navigate to the Grafana UI --> 1.12 DC/OS Kafka Dashboard to see real-time visualization:

![](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/resources/1producer.png)

Remove the Service:
```
dcos marathon app remove 1producer-lower-topic-performancetest
```

## Step 4: Run the Kafka performance tests

In this test we are using the following parameters:
- Topic: performancetest
- Number of Records: 10M
- Record Size: 250 bytes (representative of a typical log line)
- Throughput: 1M (Set arbitrarily high to "max out")
- Ack: 1 write
        - This allows Kafka to acknowledge 1 write only and let the remaining 2 replicas write in the background
- Buffer Memory: 67108864 (default)
    - Increasing buffer.memory allows Kafka to take longer before the producer starts blocking on additional sends, thereby increasing throughput. If you don't have a lot of partitions, you may not need to adjust this at all. However, if you have a lot of partitions you can tune this value taking into account the buffer size, linger time, and partition count.
- Batch Size: 8196 (default)
    - Producers can batch messages going to the same partition, tuning the producer batching to increase the batch size and time spent waiting for the batch to fill up with messages. Larger batch sizes result in fewer requests to the brokers, which reduces load on producers as well as broker CPU. Tradeoff is higher latency since messages are not sent as soon as they are ready to send
- linger.ms: 0 (default)
    - linger.ms set at 0 means that records will immediately be sent even if there is additional unused space in the buffer.
    - If the measure of performance you really care about is throughput, you can configure the linger.ms parameter to have the producer wait longer before sending. This allows the producer to wait for the batch to reach the configured batch.size
- Compression Type: none (default)
        - Can set to options: none, lz4, gzip, snappy

Description of Producer Service:
    - 1x Instance to start
    - 0.5 CPU
    - 2GB MEM

Here is the example application definition for our performance test service that we will call `250-baseline-kafka.json`
```
{
  "id": "/monitoring/loadgenerator/250-baseline",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "cmd": "kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 linger.ms=0 retries=0 bootstrap.servers=kafka-0-broker.kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.kafka.autoip.dcos.thisdcos.directory:1025",
  "container": {
    "type": "MESOS",
    "volumes": [],
    "docker": {
      "image": "confluentinc/cp-kafka",
      "forcePullImage": false,
      "parameters": []
    }
  },
  "cpus": 0.5,
  "disk": 0,
  "instances": 1,
  "maxLaunchDelaySeconds": 0,
  "mem": 2000,
  "gpus": 0,
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 0
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "healthChecks": [],
  "fetch": [],
  "constraints": []
}
```

Note: Note that running multiple producers from the same node is less effective in this situation because our bottleneck may start to come from other places, such as the NIC. Keeping the producers on separate nodes is more ideal for our current testing case as we can then remove the Producer as the throughput bottleneck.

Launch the producer service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/250-baseline.json
```

Navigate to the DC/OS UI --> Services --> 250-baseline --> logs --> Output (stdout) to view performance test results:
```
(AT BEGINNING OF FILE)
131328 records sent, 26265.6 records/sec (6.26 MB/sec), 551.6 ms avg latency, 1191.0 max latency.
344937 records sent, 68987.4 records/sec (16.45 MB/sec), 998.5 ms avg latency, 1790.0 max latency.
254200 records sent, 49960.7 records/sec (11.91 MB/sec), 1634.0 ms avg latency, 2393.0 max latency.
415975 records sent, 83161.7 records/sec (19.83 MB/sec), 543.2 ms avg latency, 1696.0 max latency.
548232 records sent, 109646.4 records/sec (26.14 MB/sec), 124.0 ms avg latency, 294.0 max latency.
977810 records sent, 195562.0 records/sec (46.63 MB/sec), 35.2 ms avg latency, 486.0 max latency.
1140992 records sent, 228198.4 records/sec (54.41 MB/sec), 47.7 ms avg latency, 374.0 max latency.
1157386 records sent, 231477.2 records/sec (55.19 MB/sec), 118.6 ms avg latency, 593.0 max latency.
1120032 records sent, 224006.4 records/sec (53.41 MB/sec), 40.6 ms avg latency, 288.0 max latency.
1193148 records sent, 238629.6 records/sec (56.89 MB/sec), 30.3 ms avg latency, 176.0 max latency.
1116922 records sent, 223384.4 records/sec (53.26 MB/sec), 159.2 ms avg latency, 683.0 max latency.
1114785 records sent, 222957.0 records/sec (53.16 MB/sec), 70.9 ms avg latency, 288.0 max latency.
10000000 records sent, 160986.525428 records/sec (38.38 MB/sec), 169.89 ms avg latency, 2393.00 ms max latency, 90 ms 50th, 907 ms 95th, 1591 ms 99th, 1905 ms 99.9th.
```

Remove the service:
```
dcos marathon app remove 250-baseline
```


### Kafka Consumer Performance Testing

Description of Producer Service:
- 1x Instance to start
- 0.5 CPU
- 2GB MEM

In this test we are using the following parameters:
- Topic: performancetest
- Number of Messages to Consume: 10M
- Threads: 1

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/1consumer-topic-performancetest.json
```

Navigate to the DC/OS UI --> Services --> 1consumer-topic-performancetest --> logs --> Output (stdout) to view performance test results. Example Output (Edited for readability):
```
(AT BEGINNING OF FILE)
start.time - 2018-11-20 18:29:16:590
end.time - 2018-11-20 18:29:44:568
data.consumed.in.MB - 2384.1858
MB.sec - 85.2164
data.consumed.in.nMsg - 10000000
nMsg.sec - 357423.6900
rebalance.time.ms - 3050
fetch.time.ms - 24928
fetch.MB.sec - 95.6429
fetch.nMsg.sec - 401155.3273
```

Remove the Service:
```
dcos marathon app remove 1consumer-performancetest
```

## Goal: Increase Throughput

#### Producers
For increasing throughput of Producers, Confluent recommends:
- batch.size: increase to 100000-200000 (default 16384)
- linger.ms: increase to 10-100 (default 0)
- compression.type = lz4 (default none)
- acks = 1 (default 1)
- buffer.memory: increase if there are a lot of partitions (default 6710884))

#### Consumers
For increasing throughput of Consumers, Confluent recommends:
- fetch.min.bytes: increase to ~1000000 (default 1)

### Producer Test

#### Lets try the lower end range parameters of the recommendations above:
- number of records - 10M
- Record Size: 250 bytes (representative of a typical log line)
- batch.size - 100000
- linger.ms - 10
- compression.type - lz4
- acks - 1
- buffer.memory - default

Deploy the service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/1producer-lower-topic-performancetest.json
```

Example Output in the logs:
```
10000000 records sent, 342935.528121 records/sec (81.76 MB/sec), 34.17 ms avg latency, 410.00 ms max latency, 10 ms 50th, 90 ms 95th, 99 ms 99th, 198 ms 99.9th.
```

Remove the Service:
```
dcos marathon app remove 1producer-lower-topic-performancetest
```

#### Lets try the upper end range parameters of the recommendations above:
- number of records - 10M
- batch.size - 200000
- linger.ms - 100
- compression.type - lz4
- acks - 1
- buffer.memory - default

Deploy Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/1producer-higher-topic-performancetest.json
```

Example Output in the logs:
```
10000000 records sent, 247586.036148 records/sec (59.03 MB/sec), 102.39 ms avg latency, 600.00 ms max latency, 101 ms 50th, 192 ms 95th, 199 ms 99th, 212 ms 99.9th.
```

Remove the service:
```
dcos marathon app remove 1producer-higher-topic-performancetest
```

### Consumer Test

#### Lets try the upper end range parameters of the recommendations above:
- fetch.min.bytes: increase to ~1000000 (default 1)

Run the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/1consumer-higher-topic-performancetest.json
```

Output:
```
start.time - 2018-11-21 18:02:51:305
end.time - 2018-11-21 18:03:15:289
data.consumed.in.MB - 2384.2063
MB.sec - 99.4082
data.consumed.in.nMsg - 10000086
nMsg.sec - 416948.2155
rebalance.time.ms - 3020
fetch.time.ms - 20964
fetch.MB.sec - 113.7286
fetch.nMsg.sec - 477012.3068
```

Remove the Service:
```
dcos marathon app remove 1consumer-higher-performancetest
```

### Conclusions

#### Producers

Lower Range - 113% increase in Throughput

Higher Range - 54% increase in Throughput

By tuning for throughput and increasing the batch.size, linger.ms, and compression.type parameters we can see a significant increase in throughput performance as well as latency performance of our Kafka cluster. For a 250 byte record it seems as though the lower end ranges are more ideal, resulting in >300K records/sec. The upper end also saw improvements in performance, but may be more ideal for a situation where the record size is much larger.

For the rest of the testing, we will utilize the Lower Range parameters, but it would be advised to do more A/B testing within the range to optimize for your specific record-size, and hardware specs

#### Consumers
Increasing fetch.min.bytes from 1 --> 1000000 resulted in a ~16% increase in throughput performance of our Consumer.

## Horizontal Scale
Now that we have reached a "peak" in our current configuration (3CPU, 12GB MEM, 25GB DISK) lets horizontally scale our cluster to see what performance benefits we can gain. Begin so by adding some nodes to your DC/OS cluster. We started this guide with 5, and for the rest of this guide we will continue to scale test using up to 15 producers

### DC/OS Cluster Prerequisites
- 1 Master
- 10 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine
- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM See here for more recommended instance types by Confluent

### Kafka Cluster Parameters
- 5x Brokers
- 3 CPU
- 12GB MEM
- 25 GB Disk
- 512 MB JVM Heap Size

As you can see, nothing has changed above from our prior configuration except for scaling from 3 to 5 Kafka brokers. You can do so by passing an update command with an updated options.json file, or through the UI change Kafka broker count to 5.

To validate that our deployment is correct:
```
dcos kafka plan status deploy
```

Output should look similar to below:
```
$ dcos kafka plan status deploy
deploy (serial strategy) (COMPLETE)
└─ broker (serial strategy) (COMPLETE)
   ├─ kafka-0:[broker] (COMPLETE)
   ├─ kafka-1:[broker] (COMPLETE)
   ├─ kafka-2:[broker] (COMPLETE)
   ├─ kafka-3:[broker] (COMPLETE)
   ├─ kafka-4:[broker] (COMPLETE)
```

As we can see from above, our throughput for a single producer hasnt increased/decreased too much, however in order to gain the benefits of horizontal scaling we will also throw multiple producers at the same topic to see how much total throughput we can get out of the Kafka deployment.

## Running Multiple Producers in Parallel
In order to attack this throughput problem with multiple producers in parallel, we will run the performance test as a service in DC/OS and scale it  to run multiple producers. Note that running multiple producers from the same node is less effective in this situation because our bottleneck may start to come from other places, such as the NIC. Keeping the producers on seperate nodes is more ideal for our current testing case as we can then remove the Producer as the throughput bottleneck.

Here is the example application definition for our performance test service that we will call `3producer-topic-performancetest.json`
```
{
  "id": "/3producer-topic-performancetest",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "cmd": "kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=lz4 batch.size=100000 linger.ms=10 retries=0 bootstrap.servers=kafka-0-broker.kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.kafka.autoip.dcos.thisdcos.directory:1025",
  "container": {
    "type": "MESOS",
    "volumes": [],
    "docker": {
      "image": "confluentinc/cp-kafka",
      "forcePullImage": false,
      "parameters": []
    }
  },
  "cpus": 0.5,
  "disk": 0,
  "instances": 3,
  "maxLaunchDelaySeconds": 0,
  "mem": 2000,
  "gpus": 0,
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 0
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "healthChecks": [],
  "fetch": [],
  "constraints": [
    [
      "hostname",
      "UNIQUE"
    ]
  ]
}
```

Description of Producer Service:
- 3x Instances to start
- 0.5 CPU
- 2GB MEM
- Constraint: HOSTNAME / UNIQUE

Launch the marathon service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/3producer-topic-performancetest.json
```

Navigate to the Grafana UI --> 1.12 DC/OS Kafka Dashboard to view performance test results:

![](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/resources/3producer.png)


Remove the Service:
```
dcos marathon app remove 3producer-topic-performancetest
```

As you can see from above, running multiple Producers in parallel I was able to horizontally scale to ~900+K records/sec to my single `performancetest` topic. We could probably handle even more, which we will continue to test below

### Example total throughput from 5 Producers

Launch the marathon service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/5producer-topic-performancetest.json
```

Navigate to the Grafana UI --> 1.12 DC/OS Kafka Dashboard to view performance test results:

![](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/resources/5producer.png)

Remove the Service:
```
dcos marathon app remove 5producer-topic-performancetest
```

### Conclusions
As you can see from above, as we scale our Producers in parallel we can observe a linear relationship between adding more Producers and the Throughput increase. Now we will continue to scale our DC/OS cluster as well as our Kafka deployment to see if we can get even higher than 1.5 million records/sec with 5 producers.

## Optional: Scale your Cluster Again to test 10/15 Producers as well as adding more Partitions

### DC/OS Cluster Prerequisites
- 1 Master
- 10 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine
- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM See here for more recommended instance types by Confluent
    - EBS Backed Storage - 60 GB

### Test Setup:
- 5x Kafka Brokers
- 10/15 Producers

### Example output from 10 Producers

Deploy the service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/10producer-topic-performancetest.json
```

Navigate to the Grafana UI --> 1.12 DC/OS Kafka Dashboard to view performance test results:

![](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/resources/10producer.png)

Remove the Service:
```
dcos marathon app remove 10producer-topic-performancetest
```

### Example output from 15 Producers

Deploy the service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/15producer-topic-performancetest.json
```

Navigate to the Grafana UI --> 1.12 DC/OS Kafka Dashboard to view performance test results:

![](https://github.com/ably77/DCOS-Kafka-Performance/blob/master/resources/15producer.png)

Remove Service:
```
dcos marathon app remove 15producer-topic-performancetest
```

## Increasing Topic Partitions
As we increase the number of Kafka brokers in our cluster, we start to be able to tinker more with topic partitions. Partitions are a unit of parallelism in Kafka and can help with lowering latency.

### A standard formula for Partitions:
```
P = Throughput from producer to single partition
C = Throughput from a single partition to a consumer
T = Target throughput

Required # of Partitions = Max (T/P, T/C)
```

So for example if my target throughput (T) is 10 million messages, Required # of partitions would be 10M/330K which is 30 partitions

# Other Design Goals

## Goal: Optimize for Latency

#### Producers
For optimizing latency of Producers, Confluent recommends:
- linger.ms - 0
- compression.type - none
- acks - 1

#### Brokers
For optimizing latency of Brokers, Confluent recommends:
- num.replica.fetchers - increase if followers can't keep up with the leader (default = 1)

#### Consumers
For optimizing latency of Consumers, Confluent recommends:
- fetch.min.bytes - 1 (default 1)

## Goal: Optimize for Durability

#### Producers
For optimizing durability of Producers, Confluent recommends:
- replication.factor - 3, configure per topic
- acks - all
- retries - 1
- max.in.flight.requests.per.connection - 1 (default 5)
    - to prevent out of order messages

#### Brokers
For optimizing durability of Brokers, Confluent recommends:
- default.replication.factor - 3 (default 1)
- auto.create.topics.enable - false (default true)
- min.insync.replicas - 2 (default 1)
- unclean.leader.election.enable - false (default true)
- broker.rack - rack of the broker (default null)
- log.flush.interval.messages / log.flush.interval.ms - for topics with very low throughput, set message interval or time interval low as needed (default allows the OS to control flushing)

#### Consumers
For optimizing durability of Consumers, Confluent recommends:
- auto.commit.enable - false (default true)

## Goal: Optimize for Availability

#### Brokers
For optimizing availability of Brokers, Confluent recommends:
- unclean.leader.election.enable - true (default true)
- min.insync.replicas - 1 (default 1)
- num.recovery.threads.per.data.dir - number of directories in log.dirs (default 1)

#### Consumers
For optimizing availability of Consumers, Confluent recommends:
- session.timeout.ms - as low as feasible (default 10000)
