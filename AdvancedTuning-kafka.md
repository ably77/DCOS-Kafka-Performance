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
- 5000 MB Disk
- 512 MB JVM Heap Size

**For our Advanced Guide we will use a larger Kafka cluster size to observe performance improvements:**

### Our Advanced Kafka Framework Parameters
- 3x Brokers
- 3 CPU
- 12GB MEM
- 25 GB Disk
- 512 MB JVM Heap Size

### If you have an Existing Kafka Deployment

If you were following the Quickstart guide before this, we deployed the default Kafka framework with the specs listed above.

The Kafka framework does not support changing the volume requirements after initial deployment in order to prevent accidental data loss from reallocation. This will require an uninstall, and reinstall of the Kafka deployment since we are testing a volume requirement 25GB Disk instead of the default 5GB.

**Optional:** If you are already using the service name `kafka` and cannot uninstall the deployment, you can follow this guide with a second Kafka cluster if you have the resources available. Otherwise, if you want to just continue to use the default 5GB storage volume requirement you can just scale using the commands below as well.

To Uninstall Kafka:
```
dcos package uninstall kafka --yes
```

If you do not have Kafka deployed yet, you can save the `options.json` configuration below and follow the instructions
```
{
  "service": {
    "name": "kafka"
  },
    "brokers": {
    "cpus": 3,
    "mem": 12000,
    "heap": {
      "size": 512
    },
        "disk": 25000,
    "count": 3
  }
}
```

## Step 1: Install Kafka
```
dcos package install kafka --options=options-kafka.json --yes
```

Validate kafka Installation:
```
dcos kafka plan status deploy
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

## Step 3: Run the Kafka performance tests

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
  "id": "/250-baseline",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "cmd": "kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 linger.ms=0 retries=0 bootstrap.servers=broker.kafka.l4lb.thisdcos.directory:9092",
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
  "maxLaunchDelaySeconds": 15,
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

Note: Note that running multiple producers from the same node is less effective in this situation because our bottleneck may start to come from other places, such as the NIC. Keeping the producers on seperate nodes is more ideal for our current testing case as we can then remove the Producer as the throughput bottleneck.

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
dcos marathon app remove 1consumer-topic-performancetest
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
10000000 records sent, 233579.370270 records/sec (55.69 MB/sec), 106.96 ms avg latency, 693.00 ms max latency, 100 ms 50th, 194 ms 95th, 204 ms 99th, 292 ms 99.9th.
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
start.time - 2018-11-20 18:49:06:835
end.time - 2018-11-20 18:49:34:059
data.consumed.in.MB - 2384.1858
MB.sec - 87.5766
data.consumed.in.nMsg - 10000000
nMsg.sec - 367322.9503
rebalance.time.ms - 3021
fetch.time.ms - 24203
fetch.MB.sec - 98.5079
fetch.nMsg.sec - 413171.9208
```

Remove the Service:
```
dcos marathon app remove 1consumer-higher-topic-performancetest
```

### Conclusions

#### Producers
Lower Range - 113% increase in Throughput
Higher Range - 45% increase in Throughput

By tuning for throughput and increasing the batch.size, linger.ms, and compression.type parameters we can see a significant increase in throughput performance as well as latency performance of our Kafka cluster. For a 250 byte record it seems as though the lower end ranges are more ideal, resulting in >300K records/sec. The upper end also saw improvements in performance, but may be more ideal for a situation where the record size is much larger.

For the rest of the testing, we will utilize the Lower Range parameters, but it would be advised to do more A/B testing within the range to optimize for your specific record-size

#### Consumers
Increasing fetch.min.bytes from 1 --> 1000000 resulted in a modest ~3% increase in performance of our Consumer.

## Horizontal Scale
Now that we have reached a "peak" in our current configuration (3CPU, 12GB MEM, 25GB DISK) lets horizontally scale our cluster to see what performance benefits we can gain. Begin so by adding some nodes to your DC/OS cluster. We started this guide with 5, and for the rest of this guide we will continue to scale test using up to 35 private agents

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
   └─ kafka-5:[broker] (COMPLETE)
```

## Set up Proper Monitoring
DC/OS 1.12 now ships with Prometheus + Telegraf for improved metrics capabilities. Leveraging Grafana, you can test out building dashboards and monitoring your DC/OS cluster with the guide below

![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/kafka-dashboard1.png)
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/kafka-dashboard2.png)


### Install Prometheus and Grafana

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
```

### Setting Up Grafana
Navigate to the Marathon-LB Public Agent serving the Grafana UI using the credentials `admin/admin`:
```
http://<public-agent-ip>:9094
```

This takes you to the Grafana console
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/grafana1.png)

Select `Add a Data Source` and add Prometheus as a data source

![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/grafana2.png)

Input the fields:

`Name`: Prometheus

`Type`: Prometheus

In this demo, because the Prometheus service is nested in the `/monitoring` group folder in DC/OS, the VIP hostname syntax for this demo is shown below:

`HTTP URL`: `http://prometheus-0-server.monitoringprometheus.autoip.dcos.thisdcos.directory:1025`

**Note:** your data source will not register without http:// in front of the URL

![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/grafana3.png)

Select Save and Test. Now you are ready to use Prometheus as a data source in Grafana.

### Importing Dashboards
Once you have correctly set up your data source in the steps above you can import the dashboard.json files

Select the + button --> import:
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/import2.png)

Paste the Grafana.com dashboard url or id
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/import3.png)

Reference Dashboard IDs:
- 1.12 DC/OS Kafka Dashboard - ID: 9018 - URL: https://grafana.com/dashboards/9018

Edit your Dashboard Name and Select Data Source and Import:
![](https://github.com/ably77/dcos-se/blob/master/Prometheus/resources/import3.png)


### Run the Kafka Performance Test
Now lets run the same single producer Kafka performance test optimized for throughput as before on our 5 broker node Kafka cluster

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/1producer-lower-topic-performancetest.json
```

Example Output in Logs:
```
10000000 records sent, 315437.511829 records/sec (75.21 MB/sec), 36.03 ms avg latency, 500.00 ms max latency, 13 ms 50th, 94 ms 95th, 102 ms 99th, 190 ms 99.9th.
```

Remove the Service:
```
dcos marathon app remove 1producer-lower-topic-performancetest
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
  "cmd": "kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=lz4 batch.size=100000 linger.ms=10 retries=0 bootstrap.servers=broker.kafka.l4lb.thisdcos.directory:9092",
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
  "maxLaunchDelaySeconds": 15,
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

As you can see from above, running multiple Producers in parallel I was able to horizontally scale to ~700K records/sec to my single `performancetest` topic. We could probably handle even more, which we will continue to test below

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
As you can see from above, as we scale our Producers in parallel we can observe a linear relationship between adding more Producers and the Throughput increase. Now we will continue to scale our DC/OS cluster as well as our Kafka deployment to see if we can get even higher than 1.2 million records/sec with 5 brokers.

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

### Create new topics/partitions

Using the DC/OS CLI:
```
dcos kafka topic create performancetest2 --partitions 20 --replication 3
dcos kafka topic create performancetest3 --partitions 30 --replication 3
```

### Re-test 15 Producers on more partitions to see if Latency decreases

**15 Producers - 20 Partitions - Topic: performancetest2**

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/15producer-topic-performancetest2.json
```

Output from Logs:
```
10000000 records sent, 678426.051560 records/sec (161.75 MB/sec), 18.95 ms avg latency, 427.00 ms max latency, 9 ms 50th, 27 ms 95th, 93 ms 99th, 137 ms 99.9th.
10000000 records sent, 654535.934023 records/sec (156.05 MB/sec), 18.11 ms avg latency, 440.00 ms max latency, 9 ms 50th, 75 ms 95th, 182 ms 99th, 247 ms 99.9th.
10000000 records sent, 672268.907563 records/sec (160.28 MB/sec), 18.39 ms avg latency, 433.00 ms max latency, 10 ms 50th, 40 ms 95th, 108 ms 99th, 132 ms 99.9th.
10000000 records sent, 682454.104961 records/sec (162.71 MB/sec), 19.19 ms avg latency, 446.00 ms max latency, 9 ms 50th, 29 ms 95th, 78 ms 99th, 99 ms 99.9th.
10000000 records sent, 720772.668300 records/sec (171.85 MB/sec), 19.66 ms avg latency, 426.00 ms max latency, 10 ms 50th, 35 ms 95th, 134 ms 99th, 160 ms 99.9th.
10000000 records sent, 698860.856803 records/sec (166.62 MB/sec), 19.44 ms avg latency, 458.00 ms max latency, 10 ms 50th, 99 ms 95th, 365 ms 99th, 443 ms 99.9th.
10000000 records sent, 694685.654741 records/sec (165.63 MB/sec), 19.69 ms avg latency, 447.00 ms max latency, 11 ms 50th, 124 ms 95th, 362 ms 99th, 433 ms 99.9th.
10000000 records sent, 699349.604867 records/sec (166.74 MB/sec), 19.33 ms avg latency, 434.00 ms max latency, 15 ms 50th, 98 ms 95th, 289 ms 99th, 335 ms 99.9th.
10000000 records sent, 691467.293597 records/sec (164.86 MB/sec), 18.23 ms avg latency, 367.00 ms max latency, 10 ms 50th, 41 ms 95th, 152 ms 99th, 193 ms 99.9th.
10000000 records sent, 701065.619742 records/sec (167.15 MB/sec), 18.67 ms avg latency, 382.00 ms max latency, 10 ms 50th, 35 ms 95th, 145 ms 99th, 179 ms 99.9th.
10000000 records sent, 722386.765874 records/sec (172.23 MB/sec), 19.07 ms avg latency, 412.00 ms max latency, 10 ms 50th, 34 ms 95th, 188 ms 99th, 246 ms 99.9th.
10000000 records sent, 708867.937903 records/sec (169.01 MB/sec), 18.99 ms avg latency, 429.00 ms max latency, 9 ms 50th, 27 ms 95th, 137 ms 99th, 185 ms 99.9th.
10000000 records sent, 677828.238324 records/sec (161.61 MB/sec), 17.76 ms avg latency, 361.00 ms max latency, 8 ms 50th, 25 ms 95th, 187 ms 99th, 213 ms 99.9th.
10000000 records sent, 709723.207949 records/sec (169.21 MB/sec), 19.33 ms avg latency, 408.00 ms max latency, 10 ms 50th, 44 ms 95th, 142 ms 99th, 176 ms 99.9th.
10000000 records sent, 722386.765874 records/sec (172.23 MB/sec), 18.95 ms avg latency, 402.00 ms max latency, 15 ms 50th, 85 ms 95th, 300 ms 99th, 370 ms 99.9th.

Total Throughput: 10435079.61 records/sec, 2487.93 MB/sec, 18.92 ms avg latency, 418.13 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 15producer-topic-performancetest2
```

**15 Producers - 30 Partitions - Topic: performancetest3**

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/15producer-topic-performancetest3.json
```

Output from Logs:
```
10000000 records sent, 682920.166633 records/sec (162.82 MB/sec), 19.03 ms avg latency, 309.00 ms max latency, 11 ms 50th, 75 ms 95th, 160 ms 99th, 245 ms 99.9th.
10000000 records sent, 644662.197009 records/sec (153.70 MB/sec), 18.14 ms avg latency, 279.00 ms max latency, 11 ms 50th, 67 ms 95th, 139 ms 99th, 237 ms 99.9th.
10000000 records sent, 628219.625581 records/sec (149.78 MB/sec), 18.66 ms avg latency, 359.00 ms max latency, 10 ms 50th, 67 ms 95th, 170 ms 99th, 335 ms 99.9th.
10000000 records sent, 622975.330177 records/sec (148.53 MB/sec), 17.64 ms avg latency, 314.00 ms max latency, 10 ms 50th, 57 ms 95th, 145 ms 99th, 289 ms 99.9th.
10000000 records sent, 682454.104961 records/sec (162.71 MB/sec), 18.82 ms avg latency, 404.00 ms max latency, 12 ms 50th, 79 ms 95th, 188 ms 99th, 366 ms 99.9th.
10000000 records sent, 694010.687765 records/sec (165.47 MB/sec), 19.53 ms avg latency, 353.00 ms max latency, 12 ms 50th, 81 ms 95th, 246 ms 99th, 323 ms 99.9th.
10000000 records sent, 685119.210743 records/sec (163.35 MB/sec), 18.02 ms avg latency, 307.00 ms max latency, 10 ms 50th, 60 ms 95th, 127 ms 99th, 270 ms 99.9th.
10000000 records sent, 650914.534922 records/sec (155.19 MB/sec), 18.67 ms avg latency, 309.00 ms max latency, 10 ms 50th, 49 ms 95th, 92 ms 99th, 188 ms 99.9th.
10000000 records sent, 680549.884307 records/sec (162.26 MB/sec), 18.60 ms avg latency, 275.00 ms max latency, 11 ms 50th, 70 ms 95th, 138 ms 99th, 218 ms 99.9th.
10000000 records sent, 685588.920883 records/sec (163.46 MB/sec), 19.09 ms avg latency, 278.00 ms max latency, 11 ms 50th, 62 ms 95th, 163 ms 99th, 257 ms 99.9th.
10000000 records sent, 683386.865304 records/sec (162.93 MB/sec), 18.71 ms avg latency, 297.00 ms max latency, 11 ms 50th, 77 ms 95th, 145 ms 99th, 261 ms 99.9th.
10000000 records sent, 698177.756057 records/sec (166.46 MB/sec), 19.30 ms avg latency, 372.00 ms max latency, 10 ms 50th, 44 ms 95th, 109 ms 99th, 155 ms 99.9th.
10000000 records sent, 664187.035069 records/sec (158.35 MB/sec), 18.67 ms avg latency, 379.00 ms max latency, 11 ms 50th, 70 ms 95th, 141 ms 99th, 208 ms 99.9th.
10000000 records sent, 661638.216223 records/sec (157.75 MB/sec), 18.82 ms avg latency, 362.00 ms max latency, 11 ms 50th, 76 ms 95th, 164 ms 99th, 254 ms 99.9th.
10000000 records sent, 675721.332522 records/sec (161.10 MB/sec), 18.89 ms avg latency, 300.00 ms max latency, 10 ms 50th, 70 ms 95th, 177 ms 99th, 256 ms 99.9th.

Total Throughput: 10040525.87 records/sec, 2393.86 MB/sec, 18.71 ms avg latency, 326.46 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 15producer-topic-performancetest3
```

### Conclusions
Increasing the topic partitions to both 20 and 30 resulted in a latency decrease back to an acceptable range. While both tests resulted in a similar avg latency, the 20 partition resulted in a higher max throughput but higher max latency, while the 30 partition test resulted in lower throughput but also lower avg max latency. From here on out we will choose to use the 30 partition topic, as the throughput performance loss was not super significant compared to the max avg latency decrease.

## Optional: Performance Testing using 9x Kafka Brokers and 25 Producers

### Scale your Kafka Cluster to 9 Brokers using CLI or GUI:
- 9x Brokers
- 3 CPU
- 12GB MEM
- 25 GB Disk
- 512 MB JVM Heap Size

### Test Setup:
- 9x Kafka Brokers
- 25 Producers
Total: 35 Nodes

To validate that the deployment is correct:
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
   ├─ kafka-5:[broker] (COMPLETE)
   ├─ kafka-6:[broker] (COMPLETE)
   ├─ kafka-7:[broker] (COMPLETE)
   └─ kafka-8:[broker] (COMPLETE)
```

### Example Output from 25 Producers

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/kafka_tests/25producer-topic-performancetest3.json
```

Example Output from Logs:
```
10000000 records sent, 649814.802781 records/sec (154.93 MB/sec), 37.86 ms avg latency, 504.00 ms max latency, 15 ms 50th, 133 ms 95th, 361 ms 99th, 465 ms 99.9th.
10000000 records sent, 644412.939812 records/sec (153.64 MB/sec), 35.81 ms avg latency, 456.00 ms max latency, 17 ms 50th, 198 ms 95th, 287 ms 99th, 440 ms 99.9th.
10000000 records sent, 511901.714871 records/sec (122.05 MB/sec), 16.66 ms avg latency, 368.00 ms max latency, 9 ms 50th, 82 ms 95th, 223 ms 99th, 312 ms 99.9th.
10000000 records sent, 672314.105150 records/sec (160.29 MB/sec), 37.60 ms avg latency, 413.00 ms max latency, 14 ms 50th, 187 ms 95th, 272 ms 99th, 314 ms 99.9th.
10000000 records sent, 624141.805018 records/sec (148.81 MB/sec), 36.81 ms avg latency, 427.00 ms max latency, 17 ms 50th, 202 ms 95th, 314 ms 99th, 376 ms 99.9th.
10000000 records sent, 624882.834469 records/sec (148.98 MB/sec), 36.13 ms avg latency, 462.00 ms max latency, 15 ms 50th, 117 ms 95th, 352 ms 99th, 437 ms 99.9th.
10000000 records sent, 619693.871228 records/sec (147.75 MB/sec), 35.61 ms avg latency, 534.00 ms max latency, 23 ms 50th, 169 ms 95th, 322 ms 99th, 509 ms 99.9th.
10000000 records sent, 633191.920471 records/sec (150.96 MB/sec), 36.47 ms avg latency, 459.00 ms max latency, 13 ms 50th, 130 ms 95th, 316 ms 99th, 439 ms 99.9th.
10000000 records sent, 672675.904749 records/sec (160.38 MB/sec), 24.49 ms avg latency, 347.00 ms max latency, 10 ms 50th, 69 ms 95th, 90 ms 99th, 178 ms 99.9th.
10000000 records sent, 652954.619654 records/sec (155.68 MB/sec), 37.56 ms avg latency, 501.00 ms max latency, 13 ms 50th, 141 ms 95th, 319 ms 99th, 459 ms 99.9th.
10000000 records sent, 641272.284212 records/sec (152.89 MB/sec), 36.62 ms avg latency, 415.00 ms max latency, 13 ms 50th, 90 ms 95th, 249 ms 99th, 356 ms 99.9th.
10000000 records sent, 671817.265704 records/sec (160.17 MB/sec), 36.69 ms avg latency, 433.00 ms max latency, 13 ms 50th, 96 ms 95th, 240 ms 99th, 408 ms 99.9th.
10000000 records sent, 688183.882733 records/sec (164.08 MB/sec), 36.39 ms avg latency, 464.00 ms max latency, 13 ms 50th, 183 ms 95th, 281 ms 99th, 327 ms 99.9th.
10000000 records sent, 686294.694942 records/sec (163.63 MB/sec), 20.89 ms avg latency, 400.00 ms max latency, 10 ms 50th, 59 ms 95th, 83 ms 99th, 109 ms 99.9th.
10000000 records sent, 687805.213564 records/sec (163.99 MB/sec), 28.17 ms avg latency, 497.00 ms max latency, 14 ms 50th, 126 ms 95th, 275 ms 99th, 452 ms 99.9th.
10000000 records sent, 651338.500619 records/sec (155.29 MB/sec), 37.53 ms avg latency, 488.00 ms max latency, 13 ms 50th, 95 ms 95th, 254 ms 99th, 415 ms 99.9th.
10000000 records sent, 677002.234107 records/sec (161.41 MB/sec), 36.14 ms avg latency, 438.00 ms max latency, 12 ms 50th, 88 ms 95th, 210 ms 99th, 374 ms 99.9th.
10000000 records sent, 698080.279232 records/sec (166.44 MB/sec), 24.29 ms avg latency, 344.00 ms max latency, 10 ms 50th, 67 ms 95th, 87 ms 99th, 130 ms 99.9th.
10000000 records sent, 621929.224454 records/sec (148.28 MB/sec), 21.78 ms avg latency, 326.00 ms max latency, 10 ms 50th, 75 ms 95th, 99 ms 99th, 174 ms 99.9th.
10000000 records sent, 679393.980569 records/sec (161.98 MB/sec), 37.22 ms avg latency, 448.00 ms max latency, 13 ms 50th, 93 ms 95th, 237 ms 99th, 391 ms 99.9th.
10000000 records sent, 649139.889646 records/sec (154.77 MB/sec), 22.80 ms avg latency, 476.00 ms max latency, 10 ms 50th, 64 ms 95th, 88 ms 99th, 129 ms 99.9th.
10000000 records sent, 677048.070413 records/sec (161.42 MB/sec), 26.08 ms avg latency, 398.00 ms max latency, 11 ms 50th, 72 ms 95th, 110 ms 99th, 330 ms 99.9th.
10000000 records sent, 620347.394541 records/sec (147.90 MB/sec), 36.48 ms avg latency, 472.00 ms max latency, 13 ms 50th, 107 ms 95th, 237 ms 99th, 357 ms 99.9th.
10000000 records sent, 620231.966756 records/sec (147.87 MB/sec), 26.45 ms avg latency, 415.00 ms max latency, 13 ms 50th, 113 ms 95th, 270 ms 99th, 375 ms 99.9th.
10000000 records sent, 678241.996744 records/sec (161.71 MB/sec), 26.89 ms avg latency, 446.00 ms max latency, 11 ms 50th, 78 ms 95th, 205 ms 99th, 420 ms 99.9th.

Total Throughput: 16254111.4 records/sec, 3875.3 MB/sec, 31.5768 ms avg latency, 437.24 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 25producer-topic-performancetest3
```

### Conclusions
By horizontally scaling our Kafka cluster as well as increasing the parallelism of our Producers, we can use the increased throughput parameters to achieve an aggregate >16.2 million messages per second on our single 30 partition performancetest3 topic. This was all tested on a 9 broker node Kafka cluster running on DC/OS on AWS m3.xlarge instances, which is pretty good. AWS instances optimized for storage and networking may result in even better performance since Kafka is so heavily dependent on I/O and fast network performance over anything else.

As we continue to scale, it is important to continue testing multiple parameters to achieve the best balance between throughput, latency, durability, and availability. Depending on your design goals, you can use the information below to help tweak your Kafka performance

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
