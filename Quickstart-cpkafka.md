# Load Testing Confluent-Kafka Quickstart
Want to load test our Confluent Kafka framework? Here is a guide that will take you through basic performance testing, as well as expand into other areas of how to begin performance tuning your Kafka cluster based on Confluent Kafka best practices and existing tools already provided

## Prerequisites
For this guide, the specs of my cluster are as stated below:
- DC/OS 1.11
- 1 Master
- 4 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine

- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM [See here for more recommended instance types by Confluent](https://www.confluent.io/blog/design-and-deployment-considerations-for-deploying-apache-kafka-on-aws/)
	- EBS Backed Storage - 60 GB

## Step 1: Install Confluent Kafka
```
dcos package install confluent-kafka --yes
```

### Default Kafka Framework Parameters:
The default Kafka package has these specifications for brokers:
- 3x Brokers
- 1 CPU
- 2048 MEM
- 5000 MB Disk
- 512 MB JVM Heap Size

Note that the defaults make up a rather small Kafka deployment, later we will explore tuning further inside Kafka as well as scaling out  when we go into more Advanced Tuning Tutorials. But for now we will just be load testing the default install of our framework.

Validate Confluent-Kafka Installation:
```
dcos confluent-kafka plan status deploy --name=confluent-kafka
```

Output should look like below when complete:
```
$ dcos confluent-kafka plan status deploy
deploy (serial strategy) (COMPLETE)
└─ broker (serial strategy) (COMPLETE)
   ├─ kafka-0:[broker] (COMPLETE)
   ├─ kafka-1:[broker] (COMPLETE)
   └─ kafka-2:[broker] (COMPLETE)
```

## Step 2: Add a test topic from the DC/OS cli
```
dcos confluent-kafka topic create performancetest --partitions 10 --replication 3
```

Output should look similar to below:
```
$ dcos confluent-kafka topic create performancetest --partitions 10 --replication 3
{
  "message": "Output: Created topic \"performancetest\".\n"
}
```

## Step 3: Get the List of Kafka Brokers

From the UI:
Either from the UI > Services > Kafka > Endpoints

From the CLI:
```
dcos confluent-kafka endpoint broker | jq -r .dns[] | paste -sd, -
```
**Note:** jq might need to be installed if not already. See [jq installation](https://github.com/stedolan/jq/wiki/Installation)

Output should look similar to below:
```
$ dcos confluent-kafka endpoint broker | jq -r .dns[] | paste -sd, -
kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Save this as we will need this output later for our Performance test.

## Step 4: SSH onto any private agent

If your organization already has SSH tooling set-up, feel free to use that. Otherwise, below is some instructions for SSH using the DC/OS CLI.

With the DC/OS CLI:
```
ssh-add </PATH/TO/SSH_PRIVATE_KEY>
```

Find a Node:
```
dcos node
```

Output:
```
$ dcos node
   HOSTNAME        IP                         ID                    TYPE                 REGION          ZONE
  10.0.2.254   10.0.2.254  306242b2-7a64-48b0-a140-5418c5a880e1-S3  agent            aws/us-west-2  aws/us-west-2c
  10.0.5.167   10.0.5.167  306242b2-7a64-48b0-a140-5418c5a880e1-S1  agent            aws/us-west-2  aws/us-west-2a
  10.0.5.40    10.0.5.40   306242b2-7a64-48b0-a140-5418c5a880e1-S2  agent            aws/us-west-2  aws/us-west-2a
  10.0.6.188   10.0.6.188  306242b2-7a64-48b0-a140-5418c5a880e1-S0  agent            aws/us-west-2  aws/us-west-2a
  10.0.6.224   10.0.6.224  306242b2-7a64-48b0-a140-5418c5a880e1-S8  agent            aws/us-west-2  aws/us-west-2a
  10.0.6.229   10.0.6.229  306242b2-7a64-48b0-a140-5418c5a880e1-S4  agent            aws/us-west-2  aws/us-west-2a
  10.0.7.109   10.0.7.109  306242b2-7a64-48b0-a140-5418c5a880e1-S5  agent            aws/us-west-2  aws/us-west-2a
  10.0.7.232   10.0.7.232  306242b2-7a64-48b0-a140-5418c5a880e1-S7  agent            aws/us-west-2  aws/us-west-2a
  10.0.7.236   10.0.7.236  306242b2-7a64-48b0-a140-5418c5a880e1-S6  agent            aws/us-west-2  aws/us-west-2a
master.mesos.  10.0.2.204    306242b2-7a64-48b0-a140-5418c5a880e1   master (leader)  aws/us-west-2  aws/us-west-2c
```

Select an agent and run the below command to SSH into a DC/OS Private Agent
```
dcos node ssh --master-proxy --mesos-id=<MESOS_ID> --user=<OS_USER>
```

Output should look similar to below:
```
$ dcos node ssh --master-proxy --mesos-id=306242b2-7a64-48b0-a140-5418c5a880e1-S1 --user=core
Running `ssh -A -t  -l core 52.34.83.22 -- ssh -A -t  -l core 10.0.5.167 -- `
The authenticity of host '10.0.5.167 (10.0.5.167)' can't be established.
ECDSA key fingerprint is SHA256:kyIEvP4WI75QxzW1NyAf6gHgPF9fk/xRb5lH2jS5ETs.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.5.167' (ECDSA) to the list of known hosts.
Last login: Fri Aug  3 16:40:21 UTC 2018 from 24.23.253.216 on pts/0
Container Linux by CoreOS stable (1235.9.0)
Update Strategy: No Reboots
Failed Units: 1
  update-engine.service
core@ip-10-0-5-167 ~ $
```

## Step 5: Run the Confluent Kafka Docker Image

The command below will run the Confluent Kafka docker image which contains multiple tools that we can use to produce, consume, and performance test our Kafka deployment
```
sudo docker run -it confluentinc/cp-kafka /bin/bash
```

### Test producing a message
```
echo “This is a test at $(date)” | kafka-console-producer --broker-list kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025 --topic performancetest
```

### Test consuming a message
```
kafka-console-consumer --bootstrap-server kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025 --topic performancetest --from-beginning
```

Output should look similar to below:
```
root@ba372c143b80:/# kafka-console-consumer --bootstrap-server kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025 --topic performancetest --from-beginning
“This is a test at Fri Aug 3 17:24:54 UTC 2018”
```

## Step 6: Run the Kafka Performance Test:
```
kafka-producer-perf-test --topic performancetest --num-records 5000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

In this test we are using the following parameters:
- Topic: performancetest
- Number of Records: 5M
- Record Size: 250 bytes (representative of a typical log line)
- Throughput: 1M (Set arbitrarily high to "max out")
- Ack: 1 write
	- This allows Kafka to acknowledge 1 write only and let the remaining 2 replicas write in the background
- Buffer Memory: 67108864 (default)
- Batch Size: 8196 (default)
- Compression Type: none
	- Can set to options: none, lz4, gzip, snappy

Output of the test should look similar to below:
```
906606 records sent, 181321.2 records/sec (43.23 MB/sec), 55.6 ms avg latency, 446.0 max latency.
1345573 records sent, 269114.6 records/sec (64.16 MB/sec), 54.1 ms avg latency, 354.0 max latency.
1090606 records sent, 218121.2 records/sec (52.00 MB/sec), 65.9 ms avg latency, 643.0 max latency.
1317132 records sent, 263215.8 records/sec (62.76 MB/sec), 122.9 ms avg latency, 786.0 max latency.
5000000 records sent, 235249.835325 records/sec (56.09 MB/sec), 72.84 ms avg latency, 786.00 ms max latency, 25 ms 50th, 481 ms 95th, 743 ms 99th, 777 ms 99.9th.
```

### Initial Thoughts:
As you can see from above, our first test with the default framework parameters resulted in a throughput of 235K records/sec with a record size of 250 bytes. Not a bad start, but we will continue to performance tune in the Advanced Tuning section of this guide

You can also append the `--print-metrics` flag to the performance test to retrieve more descriptive metrics information:
```
kafka-producer-perf-test --topic performancetest --num-records 5000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025 --print-metrics
```

Output should look similar to below:
```
887137 records sent, 177427.4 records/sec (42.30 MB/sec), 52.1 ms avg latency, 283.0 max latency.
1261667 records sent, 252333.4 records/sec (60.16 MB/sec), 71.2 ms avg latency, 357.0 max latency.
1480457 records sent, 296091.4 records/sec (70.59 MB/sec), 112.5 ms avg latency, 477.0 max latency.
1336610 records sent, 267322.0 records/sec (63.73 MB/sec), 61.1 ms avg latency, 481.0 max latency.
5000000 records sent, 246414.666601 records/sec (58.75 MB/sec), 78.64 ms avg latency, 481.00 ms max latency, 28 ms 50th, 244 ms 95th, 355 ms 99th, 476 ms 99.9th.

Metric Name                                                                               Value
app-info:commit-id:{client-id=producer-1}                                               : 4b1dd33f255ddd2f
app-info:version:{client-id=producer-1}                                                 : 2.0.0-cp1
kafka-metrics-count:count:{client-id=producer-1}                                        : 142.000
producer-metrics:batch-size-avg:{client-id=producer-1}                                  : 5923.779
producer-metrics:batch-size-max:{client-id=producer-1}                                  : 8090.000
producer-metrics:batch-split-rate:{client-id=producer-1}                                : 0.000
producer-metrics:batch-split-total:{client-id=producer-1}                               : 0.000
producer-metrics:buffer-available-bytes:{client-id=producer-1}                          : 67108864.000
producer-metrics:buffer-exhausted-rate:{client-id=producer-1}                           : 0.000
producer-metrics:buffer-exhausted-total:{client-id=producer-1}                          : 0.000
producer-metrics:buffer-total-bytes:{client-id=producer-1}                              : 67108864.000
producer-metrics:bufferpool-wait-ratio:{client-id=producer-1}                           : 0.000
producer-metrics:bufferpool-wait-time-total:{client-id=producer-1}                      : 0.000
producer-metrics:compression-rate-avg:{client-id=producer-1}                            : 1.000
producer-metrics:connection-close-rate:{client-id=producer-1}                           : 0.000
producer-metrics:connection-close-total:{client-id=producer-1}                          : 0.000
producer-metrics:connection-count:{client-id=producer-1}                                : 6.000
producer-metrics:connection-creation-rate:{client-id=producer-1}                        : 0.119
producer-metrics:connection-creation-total:{client-id=producer-1}                       : 6.000
producer-metrics:failed-authentication-rate:{client-id=producer-1}                      : 0.000
producer-metrics:failed-authentication-total:{client-id=producer-1}                     : 0.000
producer-metrics:incoming-byte-rate:{client-id=producer-1}                              : 175483.828
producer-metrics:incoming-byte-total:{client-id=producer-1}                             : 8800163.000
producer-metrics:io-ratio:{client-id=producer-1}                                        : 0.081
producer-metrics:io-time-ns-avg:{client-id=producer-1}                                  : 19494.152
producer-metrics:io-wait-ratio:{client-id=producer-1}                                   : 0.041
producer-metrics:io-wait-time-ns-avg:{client-id=producer-1}                             : 9973.528
producer-metrics:io-waittime-total:{client-id=producer-1}                               : 2082322992.000
producer-metrics:iotime-total:{client-id=producer-1}                                    : 4070086586.000
producer-metrics:metadata-age:{client-id=producer-1}                                    : 20.140
producer-metrics:network-io-rate:{client-id=producer-1}                                 : 2624.492
producer-metrics:network-io-total:{client-id=producer-1}                                : 1322792390.000
producer-metrics:outgoing-byte-rate:{client-id=producer-1}                              : 26199673.539
producer-metrics:outgoing-byte-total:{client-id=producer-1}                             : 1313992227.000
producer-metrics:produce-throttle-time-avg:{client-id=producer-1}                       : 0.000
producer-metrics:produce-throttle-time-max:{client-id=producer-1}                       : 0.000
producer-metrics:record-error-rate:{client-id=producer-1}                               : 0.000
producer-metrics:record-error-total:{client-id=producer-1}                              : 0.000
producer-metrics:record-queue-time-avg:{client-id=producer-1}                           : 54.470
producer-metrics:record-queue-time-max:{client-id=producer-1}                           : 478.000
producer-metrics:record-retry-rate:{client-id=producer-1}                               : 0.000
producer-metrics:record-retry-total:{client-id=producer-1}                              : 0.000
producer-metrics:record-send-rate:{client-id=producer-1}                                : 99804.383
producer-metrics:record-send-total:{client-id=producer-1}                               : 5000000.000
producer-metrics:record-size-avg:{client-id=producer-1}                                 : 336.000
producer-metrics:record-size-max:{client-id=producer-1}                                 : 336.000
producer-metrics:records-per-request-avg:{client-id=producer-1}                         : 75.983
producer-metrics:request-latency-avg:{client-id=producer-1}                             : 4.274
producer-metrics:request-latency-max:{client-id=producer-1}                             : 480.000
producer-metrics:request-rate:{client-id=producer-1}                                    : 1312.272
producer-metrics:request-size-avg:{client-id=producer-1}                                : 19964.329
producer-metrics:request-size-max:{client-id=producer-1}                                : 32449.000
producer-metrics:request-total:{client-id=producer-1}                                   : 1313992227.000
producer-metrics:requests-in-flight:{client-id=producer-1}                              : 0.000
producer-metrics:response-rate:{client-id=producer-1}                                   : 1312.455
producer-metrics:response-total:{client-id=producer-1}                                  : 8800163.000
producer-metrics:select-rate:{client-id=producer-1}                                     : 4149.723
producer-metrics:select-total:{client-id=producer-1}                                    : 2082322992.000
producer-metrics:successful-authentication-rate:{client-id=producer-1}                  : 0.000
producer-metrics:successful-authentication-total:{client-id=producer-1}                 : 0.000
producer-metrics:waiting-threads:{client-id=producer-1}                                 : 0.000
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--1}        : 16.770
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--2}        : 4.945
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--3}        : 4.946
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-0}         : 45096.499
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-1}         : 71713.775
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-2}         : 58799.800
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--1}       : 841.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--2}       : 248.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--3}       : 248.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-0}        : 2259515.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-1}        : 3593147.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-2}        : 2946164.000
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--1}        : 1.874
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--2}        : 0.957
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--3}        : 0.957
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-0}         : 7850811.089
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-1}         : 10494778.496
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-2}         : 7879911.426
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--1}       : 94.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--2}       : 48.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--3}       : 48.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-0}        : 393349188.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-1}        : 525819887.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-2}        : 394822962.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--1}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--2}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--3}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-0}        : 5.119
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-1}        : 3.916
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-2}        : 3.976
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--1}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--2}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--3}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-0}        : 95.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-1}        : 70.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-2}        : 480.000
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--1}              : 0.060
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--2}              : 0.040
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--3}              : 0.040
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-0}               : 366.650
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-1}               : 468.716
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-2}               : 478.057
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--1}          : 31.333
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--2}          : 24.000
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--3}          : 24.000
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-0}           : 21411.420
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-1}           : 22389.606
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-2}           : 16482.548
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--1}          : 46.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--2}          : 24.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--3}          : 24.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-0}           : 24351.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-1}           : 32449.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-2}           : 24351.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--1}             : 94.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--2}             : 48.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--3}             : 48.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-0}              : 393349188.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-1}              : 525819887.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-2}              : 394822962.000
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--1}             : 0.060
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--2}             : 0.040
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--3}             : 0.040
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-0}              : 366.657
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-1}              : 468.725
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-2}              : 478.076
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--1}            : 841.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--2}            : 248.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--3}            : 248.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-0}             : 2259515.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-1}             : 3593147.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-2}             : 2946164.000
producer-topic-metrics:byte-rate:{client-id=producer-1, topic=performancetest}          : 26119330.585
producer-topic-metrics:byte-total:{client-id=producer-1, topic=performancetest}         : 1308473985.000
producer-topic-metrics:compression-rate:{client-id=producer-1, topic=performancetest}   : 1.000
producer-topic-metrics:record-error-rate:{client-id=producer-1, topic=performancetest}  : 0.000
producer-topic-metrics:record-error-total:{client-id=producer-1, topic=performancetest} : 0.000
producer-topic-metrics:record-retry-rate:{client-id=producer-1, topic=performancetest}  : 0.000
producer-topic-metrics:record-retry-total:{client-id=producer-1, topic=performancetest} : 0.000
producer-topic-metrics:record-send-rate:{client-id=producer-1, topic=performancetest}   : 99808.368
producer-topic-metrics:record-send-total:{client-id=producer-1, topic=performancetest}  : 5000000.000
```

### Kafka Consumer Performance Testing
```
kafka-consumer-perf-test --topic performancetest --messages 15000000 --threads 1 --broker-list=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```
- Topic: performancetest
- Number of Messages to Consume: 1.5M
- Threads: 1 (default)
	- set according to how many disks are in the cluster

Example Output (Edited for readability):
```
start.time - 2018-08-09 17:04:16:955
end.time - 2018-08-09 17:04:54:128
data.consumed.in.MB - 3576.3285
MB.sec - 96.2077
data.consumed.in.nMsg - 15000209
nMsg.sec - 403524.3053
rebalance.time.ms - 3048
fetch.time.ms - 34125
fetch.MB.sec - 104.8008
fetch.nMsg.sec - 439566.5641
```

### Initial Thoughts:
As you can see from above, our first test with the default framework parameters resulted in a consumer throughput of 403524 msg/sec at a rate of about 96.2 MB/sec. Again we we will continue to performance tune in the Advanced Tuning section of this guide.

## More Tests
In this example I have tested the base 3 Kafka broker node deploy on a DC/OS Cluster running on AWS m3.xlarge instances. From my test observations with these fixed parameters set:
- Topic: performancetest
- Number of Records: 20M
- Throughput: 5M (Set arbitrarily high to "max out")
- Ack: 1 write
        - This allows Kafka to acknowledge 1 write only and let the remaining 2 replicas write in the background
- Buffer Memory: 67108864 (default)
- Batch Size: 8196 (default)
- Compression Type: none

My Variable parameter was `record-size` in bytes which I averaged across 5 runs:

**record-size: 1**
```
5000000 records sent, 291494.199265 records/sec (0.28 MB/sec), 11.24 ms avg latency, 234.00 ms max latency, 4 ms 50th, 35 ms 95th, 47 ms 99th, 62 ms 99.9th.
5000000 records sent, 269570.843218 records/sec (0.26 MB/sec), 10.18 ms avg latency, 220.00 ms max latency, 4 ms 50th, 36 ms 95th, 49 ms 99th, 74 ms 99.9th.
5000000 records sent, 244714.173845 records/sec (0.23 MB/sec), 10.30 ms avg latency, 219.00 ms max latency, 4 ms 50th, 37 ms 95th, 53 ms 99th, 63 ms 99.9th.
5000000 records sent, 272628.135224 records/sec (0.26 MB/sec), 11.48 ms avg latency, 229.00 ms max latency, 4 ms 50th, 41 ms 95th, 57 ms 99th, 73 ms 99.9th.
5000000 records sent, 271076.172404 records/sec (0.26 MB/sec), 10.98 ms avg latency, 222.00 ms max latency, 5 ms 50th, 40 ms 95th, 54 ms 99th, 72 ms 99.9th.

Average: 269896.7 records/sec, 0.258 MB/sec, 10.84 ms avg latency, 224.8 ms max latency
```

**record-size: 10**
```
5000000 records sent, 267608.649112 records/sec (2.55 MB/sec), 13.98 ms avg latency, 230.00 ms max latency, 8 ms 50th, 52 ms 95th, 73 ms 99th, 105 ms 99.9th.
5000000 records sent, 258598.396690 records/sec (2.47 MB/sec), 12.68 ms avg latency, 236.00 ms max latency, 6 ms 50th, 46 ms 95th, 62 ms 99th, 104 ms 99.9th.
5000000 records sent, 283286.118980 records/sec (2.70 MB/sec), 11.46 ms avg latency, 235.00 ms max latency, 6 ms 50th, 42 ms 95th, 58 ms 99th, 73 ms 99.9th.
5000000 records sent, 278613.618634 records/sec (2.66 MB/sec), 11.69 ms avg latency, 237.00 ms max latency, 4 ms 50th, 31 ms 95th, 45 ms 99th, 64 ms 99.9th.
5000000 records sent, 282023.802809 records/sec (2.69 MB/sec), 12.76 ms avg latency, 228.00 ms max latency, 4 ms 50th, 28 ms 95th, 38 ms 99th, 56 ms 99.9th.

Average: 274026.1172 records/sec, 2.61 MB/sec, 12.51 ms avg latency, 233.2 ms max latency
```

**record-size: 50**
```
5000000 records sent, 261547.313909 records/sec (12.47 MB/sec), 13.34 ms avg latency, 227.00 ms max latency, 6 ms 50th, 41 ms 95th, 60 ms 99th, 86 ms 99.9th.
5000000 records sent, 250012.500625 records/sec (11.92 MB/sec), 20.99 ms avg latency, 782.00 ms max latency, 8 ms 50th, 45 ms 95th, 64 ms 99th, 85 ms 99.9th.
5000000 records sent, 266325.769681 records/sec (12.70 MB/sec), 12.87 ms avg latency, 239.00 ms max latency, 7 ms 50th, 45 ms 95th, 62 ms 99th, 81 ms 99.9th.
5000000 records sent, 295980.583674 records/sec (14.11 MB/sec), 13.18 ms avg latency, 232.00 ms max latency, 7 ms 50th, 43 ms 95th, 57 ms 99th, 84 ms 99.9th.
5000000 records sent, 274679.997803 records/sec (13.10 MB/sec), 16.40 ms avg latency, 239.00 ms max latency, 12 ms 50th, 57 ms 95th, 83 ms 99th, 106 ms 99.9th.

Average: 269709.23 records/sec, 12.86 MB/sec, 15.36 ms avg latency, 343.8 ms max latency
```

**record-size: 100**
```
5000000 records sent, 255506.157698 records/sec (24.37 MB/sec), 15.88 ms avg latency, 217.00 ms max latency, 4 ms 50th, 31 ms 95th, 47 ms 99th, 63 ms 99.9th.
5000000 records sent, 266922.912663 records/sec (25.46 MB/sec), 16.39 ms avg latency, 232.00 ms max latency, 5 ms 50th, 32 ms 95th, 52 ms 99th, 68 ms 99.9th.
5000000 records sent, 262922.648157 records/sec (25.07 MB/sec), 15.19 ms avg latency, 242.00 ms max latency, 12 ms 50th, 53 ms 95th, 92 ms 99th, 115 ms 99.9th.
5000000 records sent, 259470.679813 records/sec (24.75 MB/sec), 15.09 ms avg latency, 235.00 ms max latency, 12 ms 50th, 46 ms 95th, 59 ms 99th, 72 ms 99.9th.
5000000 records sent, 241370.987207 records/sec (23.02 MB/sec), 14.67 ms avg latency, 235.00 ms max latency, 9 ms 50th, 43 ms 95th, 61 ms 99th, 102 ms 99.9th.

Average: 257238.68 records/sec, 24.53 MB/sec, 15.44 ms avg latency, 232.2 ms max latency
```

**record-size: 250**
```
5000000 records sent, 247011.164905 records/sec (58.89 MB/sec), 80.89 ms avg latency, 539.00 ms max latency, 18 ms 50th, 247 ms 95th, 405 ms 99th, 450 ms 99.9th.
5000000 records sent, 229811.095280 records/sec (54.79 MB/sec), 59.21 ms avg latency, 525.00 ms max latency, 8 ms 50th, 91 ms 95th, 395 ms 99th, 514 ms 99.9th.
5000000 records sent, 242918.913667 records/sec (57.92 MB/sec), 116.52 ms avg latency, 945.00 ms max latency, 24 ms 50th, 673 ms 95th, 882 ms 99th, 934 ms 99.9th.
5000000 records sent, 219355.970870 records/sec (52.30 MB/sec), 248.81 ms avg latency, 1628.00 ms max latency, 14 ms 50th, 480 ms 95th, 767 ms 99th, 815 ms 99.9th.
5000000 records sent, 256160.663968 records/sec (61.07 MB/sec), 85.26 ms avg latency, 921.00 ms max latency, 26 ms 50th, 371 ms 95th, 428 ms 99th, 450 ms 99.9th.

Average: 239051.56 records/sec, 60 MB/sec, 118.14 ms avg latency, 911.6 ms max latency
```

**record-size: 500**
```
5000000 records sent, 142653.352354 records/sec (68.02 MB/sec), 765.89 ms avg latency, 2800.00 ms max latency, 357 ms 50th, 1208 ms 95th, 1301 ms 99th, 1395 ms 99.9th.
5000000 records sent, 123152.709360 records/sec (58.72 MB/sec), 923.38 ms avg latency, 4906.00 ms max latency, 69 ms 50th, 1410 ms 95th, 1591 ms 99th, 1891 ms 99.9th.
5000000 records sent, 150802.268066 records/sec (71.91 MB/sec), 686.42 ms avg latency, 2842.00 ms max latency, 368 ms 50th, 1002 ms 95th, 1234 ms 99th, 1276 ms 99.9th.
5000000 records sent, 127016.385114 records/sec (60.57 MB/sec), 873.63 ms avg latency, 4888.00 ms max latency, 68 ms 50th, 636 ms 95th, 695 ms 99th, 715 ms 99.9th.
5000000 records sent, 142885.720001 records/sec (68.13 MB/sec), 757.63 ms avg latency, 3224.00 ms max latency, 481 ms 50th, 2545 ms 95th, 3150 ms 99th, 3216 ms 99.9th.

Average: 137302.09 records/sec, 65.47 MB/sec, 801.39 ms avg latency, 3732 ms max latency
```

## Conclusion
At this point we have shown how to use DC/OS to spin up our default Kafka framework, as well as using the tools within the package to perform some basic load testing. As you can see from above, the 3x broker (1 CPU, 2048 MEM, 5GB DISK) default configuration hovers around ~250K msg/sec throughput on a 4 node (m3.xlarge) DC/OS Cluster. As the record-size increases, we notice that latency and throughput start to suffer - this is potentially due to not having enough horsepower. In our Advanced tutorial we will move forward to tune parameters within Kafka itself, as well as test horizontal scaling of the cluster.
