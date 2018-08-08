# Load Testing Confluent-Kafka
Want to load test our Confluent Kafka framework? Here is a guide that will take you through basic performance testing, as well as expand into other areas of how to begin performance tuning your Kafka cluster based on Confluent Kafka best practices and existing tools already provided

## Prerequisites
For this guide, the specs of my cluster are as stated below:
- DC/OS 1.11
- 1 Master
- 3 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine

- AWS Instance Type: r3.xlarge - 4vCPU, 30.5GB RAM [See here for more recommended instance types by Confluent](https://www.confluent.io/blog/design-and-deployment-considerations-for-deploying-apache-kafka-on-aws/) 
	- EBS Backed Storage - 120 GB

## Step 1: Install Confluent Kafka
```
dcos package install confluent-kafka --yes
```

Note that the default Kafka package has these specifications for brokers:
- 3x Brokers
- 1 CPU
- 2048 MEM
- 5000 MB Disk
- 512 MB JVM Heap Size

We will explore this later when we go into more Advanced Tuning Tutorials. But for now we will just be load testing the default install of our framework.

Validate Confluent-Kafka Installation:
```
dcos confluent-kafka plan status deploy
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

### Step 2: Add a test topic from the DC/OS cli
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
kafka-producer-perf-test --topic performancetest --num-records 5000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=snappy batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
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
- Compression Type: snappy (recommended)
	- Can set to options: none, lz4, gzip, snappy

Output of the test should look similar to below:
```
630004 records sent, 125724.2 records/sec (29.97 MB/sec), 166.0 ms avg latency, 1506.0 max latency.
831054 records sent, 166210.8 records/sec (39.63 MB/sec), 445.2 ms avg latency, 3076.0 max latency.
1186935 records sent, 237102.5 records/sec (56.53 MB/sec), 135.5 ms avg latency, 1166.0 max latency.
1132588 records sent, 226517.6 records/sec (54.01 MB/sec), 242.2 ms avg latency, 1700.0 max latency.
1161926 records sent, 232060.3 records/sec (55.33 MB/sec), 88.4 ms avg latency, 433.0 max latency.
5000000 records sent, 195091.497913 records/sec (46.51 MB/sec), 203.55 ms avg latency, 3076.00 ms max latency, 19 ms 50th, 216 ms 95th, 548 ms 99th, 663 ms 99.9th.
```

You can also append the `--print-metrics` flag to the performance test to retrieve more descriptive metrics information:
```
kafka-producer-perf-test --topic performancetest --num-records 20000000 --record-size 10 --throughput 5000000 --print-metrics --producer-props bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Output should look similar to below:
```
681897 records sent, 136215.9 records/sec (32.48 MB/sec), 113.3 ms avg latency, 1724.0 max latency.
859206 records sent, 171841.2 records/sec (40.97 MB/sec), 362.3 ms avg latency, 3022.0 max latency.
1034156 records sent, 206831.2 records/sec (49.31 MB/sec), 618.9 ms avg latency, 3610.0 max latency.
1044415 records sent, 208883.0 records/sec (49.80 MB/sec), 595.3 ms avg latency, 3264.0 max latency.
1129617 records sent, 225923.4 records/sec (53.86 MB/sec), 678.5 ms avg latency, 3855.0 max latency.
5000000 records sent, 188253.012048 records/sec (44.88 MB/sec), 505.41 ms avg latency, 3855.00 ms max latency, 13 ms 50th, 176 ms 95th, 302 ms 99th, 370 ms 99.9th.

Metric Name                                                                               Value
app-info:commit-id:{client-id=producer-1}                                               : 4b1dd33f255ddd2f
app-info:version:{client-id=producer-1}                                                 : 2.0.0-cp1
kafka-metrics-count:count:{client-id=producer-1}                                        : 142.000
producer-metrics:batch-size-avg:{client-id=producer-1}                                  : 844.688
producer-metrics:batch-size-max:{client-id=producer-1}                                  : 8107.000
producer-metrics:batch-split-rate:{client-id=producer-1}                                : 0.000
producer-metrics:batch-split-total:{client-id=producer-1}                               : 0.000
producer-metrics:buffer-available-bytes:{client-id=producer-1}                          : 67108864.000
producer-metrics:buffer-exhausted-rate:{client-id=producer-1}                           : 0.000
producer-metrics:buffer-exhausted-total:{client-id=producer-1}                          : 0.000
producer-metrics:buffer-total-bytes:{client-id=producer-1}                              : 67108864.000
producer-metrics:bufferpool-wait-ratio:{client-id=producer-1}                           : 0.000
producer-metrics:bufferpool-wait-time-total:{client-id=producer-1}                      : 0.000
producer-metrics:compression-rate-avg:{client-id=producer-1}                            : 0.183
producer-metrics:connection-close-rate:{client-id=producer-1}                           : 0.000
producer-metrics:connection-close-total:{client-id=producer-1}                          : 0.000
producer-metrics:connection-count:{client-id=producer-1}                                : 6.000
producer-metrics:connection-creation-rate:{client-id=producer-1}                        : 0.106
producer-metrics:connection-creation-total:{client-id=producer-1}                       : 6.000
producer-metrics:failed-authentication-rate:{client-id=producer-1}                      : 0.000
producer-metrics:failed-authentication-total:{client-id=producer-1}                     : 0.000
producer-metrics:incoming-byte-rate:{client-id=producer-1}                              : 137109.034
producer-metrics:incoming-byte-total:{client-id=producer-1}                             : 7738571.000
producer-metrics:io-ratio:{client-id=producer-1}                                        : 0.066
producer-metrics:io-time-ns-avg:{client-id=producer-1}                                  : 17845.079
producer-metrics:io-wait-ratio:{client-id=producer-1}                                   : 0.090
producer-metrics:io-wait-time-ns-avg:{client-id=producer-1}                             : 24324.668
producer-metrics:io-waittime-total:{client-id=producer-1}                               : 5074198810.000
producer-metrics:iotime-total:{client-id=producer-1}                                    : 3722537018.000
producer-metrics:metadata-age:{client-id=producer-1}                                    : 26.433
producer-metrics:network-io-rate:{client-id=producer-1}                                 : 3073.080
producer-metrics:network-io-total:{client-id=producer-1}                                : 151245318.000
producer-metrics:outgoing-byte-rate:{client-id=producer-1}                              : 2542507.432
producer-metrics:outgoing-byte-total:{client-id=producer-1}                             : 143506747.000
producer-metrics:produce-throttle-time-avg:{client-id=producer-1}                       : 0.000
producer-metrics:produce-throttle-time-max:{client-id=producer-1}                       : 0.000
producer-metrics:record-error-rate:{client-id=producer-1}                               : 0.000
producer-metrics:record-error-total:{client-id=producer-1}                              : 0.000
producer-metrics:record-queue-time-avg:{client-id=producer-1}                           : 171.319
producer-metrics:record-queue-time-max:{client-id=producer-1}                           : 3841.000
producer-metrics:record-retry-rate:{client-id=producer-1}                               : 0.000
producer-metrics:record-retry-total:{client-id=producer-1}                              : 0.000
producer-metrics:record-send-rate:{client-id=producer-1}                                : 88709.105
producer-metrics:record-send-total:{client-id=producer-1}                               : 5000000.000
producer-metrics:record-size-avg:{client-id=producer-1}                                 : 336.000
producer-metrics:record-size-max:{client-id=producer-1}                                 : 336.000
producer-metrics:records-per-request-avg:{client-id=producer-1}                         : 57.659
producer-metrics:request-latency-avg:{client-id=producer-1}                             : 4.100
producer-metrics:request-latency-max:{client-id=producer-1}                             : 175.000
producer-metrics:request-rate:{client-id=producer-1}                                    : 1536.567
producer-metrics:request-size-avg:{client-id=producer-1}                                : 1654.638
producer-metrics:request-size-max:{client-id=producer-1}                                : 16061.000
producer-metrics:request-total:{client-id=producer-1}                                   : 143506747.000
producer-metrics:requests-in-flight:{client-id=producer-1}                              : 0.000
producer-metrics:response-rate:{client-id=producer-1}                                   : 1536.649
producer-metrics:response-total:{client-id=producer-1}                                  : 7738571.000
producer-metrics:select-rate:{client-id=producer-1}                                     : 3686.477
producer-metrics:select-total:{client-id=producer-1}                                    : 5074198810.000
producer-metrics:successful-authentication-rate:{client-id=producer-1}                  : 0.000
producer-metrics:successful-authentication-total:{client-id=producer-1}                 : 0.000
producer-metrics:waiting-threads:{client-id=producer-1}                                 : 0.000
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--1}        : 4.394
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--2}        : 10.808
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--3}        : 4.394
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-0}         : 71787.795
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-1}         : 53318.851
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-2}         : 12157.785
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--1}       : 248.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--2}       : 610.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--3}       : 248.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-0}        : 4046678.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-1}        : 3005477.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-2}        : 685310.000
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--1}        : 0.851
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--2}        : 1.665
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--3}        : 0.850
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-0}         : 1158191.876
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-1}         : 1013162.604
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-2}         : 374339.436
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--1}       : 48.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--2}       : 94.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--3}       : 48.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-0}        : 65293067.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-1}        : 57111976.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-2}        : 21101514.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--1}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--2}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--3}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-0}        : 2.597
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-1}        : 3.509
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-2}        : 11.877
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--1}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--2}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--3}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-0}        : 133.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-1}        : 175.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-2}        : 169.000
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--1}              : 0.035
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--2}              : 0.053
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--3}              : 0.035
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-0}               : 772.009
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-1}               : 573.443
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-2}               : 192.940
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--1}          : 24.000
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--2}          : 31.333
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--3}          : 24.000
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-0}           : 1500.231
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-1}           : 1766.805
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-2}           : 1940.191
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--1}          : 24.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--2}          : 46.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--3}          : 24.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-0}           : 16030.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-1}           : 16061.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-2}           : 8172.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--1}             : 48.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--2}             : 94.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--3}             : 48.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-0}              : 65293067.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-1}              : 57111976.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-2}              : 21101514.000
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--1}             : 0.035
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--2}             : 0.053
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--3}             : 0.035
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-0}              : 772.077
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-1}              : 573.464
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-2}              : 192.946
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--1}            : 248.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--2}            : 610.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--3}            : 248.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-0}             : 4046678.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-1}             : 3005477.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-2}             : 685310.000
producer-topic-metrics:byte-rate:{client-id=producer-1, topic=performancetest}          : 2435391.363
producer-topic-metrics:byte-total:{client-id=producer-1, topic=performancetest}         : 137263528.000
producer-topic-metrics:compression-rate:{client-id=producer-1, topic=performancetest}   : 0.183
producer-topic-metrics:record-error-rate:{client-id=producer-1, topic=performancetest}  : 0.000
producer-topic-metrics:record-error-total:{client-id=producer-1, topic=performancetest} : 0.000
producer-topic-metrics:record-retry-rate:{client-id=producer-1, topic=performancetest}  : 0.000
producer-topic-metrics:record-retry-total:{client-id=producer-1, topic=performancetest} : 0.000
producer-topic-metrics:record-send-rate:{client-id=producer-1, topic=performancetest}   : 88712.253
producer-topic-metrics:record-send-total:{client-id=producer-1, topic=performancetest}  : 5000000.000
```

### Kafka Consumer Performance Testing
```
kafka-consumer-perf-test --topic performancetest --messages 15000000 --threads 1 --broker-list=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```
- Topic: performancetest
- Number of Messages to Consume: 1.5M
- Threads: 1

Example Output (Edited for readability):
```
start.time = 2018-08-03 23:20:23:718
end.time = 2018-08-03 23:21:36:460
data.consumed.in.MB = 3915.3982
MB.sec = 53.8258
data.consumed.in.nMsg = 15000316
nMsg.sec = 206212.5870
rebalance.time.ms = 3088
fetch.time.ms = 69654
fetch.MB.sec = 56.2121
fetch.nMsg.sec = 215354.6961
```

## Conclusion
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
20000000 records sent, 540088.034350 records/sec (0.52 MB/sec), 10.84 ms avg latency, 213.00 ms max latency, 3 ms 50th, 39 ms 95th, 56 ms 99th, 80 ms 99.9th.
20000000 records sent, 527927.357196 records/sec (0.50 MB/sec), 11.80 ms avg latency, 372.00 ms max latency, 4 ms 50th, 36 ms 95th, 50 ms 99th, 63 ms 99.9th.
20000000 records sent, 494743.351886 records/sec (0.47 MB/sec), 10.65 ms avg latency, 224.00 ms max latency, 3 ms 50th, 33 ms 95th, 52 ms 99th, 80 ms 99.9th.
20000000 records sent, 523450.586265 records/sec (0.50 MB/sec), 11.30 ms avg latency, 295.00 ms max latency, 4 ms 50th, 38 ms 95th, 78 ms 99th, 262 ms 99.9th.
20000000 records sent, 488448.200068 records/sec (0.47 MB/sec), 10.64 ms avg latency, 208.00 ms max latency, 3 ms 50th, 34 ms 95th, 49 ms 99th, 80 ms 99.9th.

Average: 514931.5 records/sec (0.492 MB/sec), 11.05 ms avg latency, 262.4 ms avg max latency
```

**record-size: 10**
```
20000000 records sent, 469032.152154 records/sec (4.47 MB/sec), 10.76 ms avg latency, 260.00 ms max latency, 4 ms 50th, 40 ms 95th, 56 ms 99th, 88 ms 99.9th.
20000000 records sent, 528485.360956 records/sec (5.04 MB/sec), 15.29 ms avg latency, 729.00 ms max latency, 3 ms 50th, 42 ms 95th, 61 ms 99th, 80 ms 99.9th.
20000000 records sent, 473283.165318 records/sec (4.51 MB/sec), 12.00 ms avg latency, 549.00 ms max latency, 3 ms 50th, 40 ms 95th, 217 ms 99th, 510 ms 99.9th.
20000000 records sent, 512150.777189 records/sec (4.88 MB/sec), 10.93 ms avg latency, 235.00 ms max latency, 4 ms 50th, 41 ms 95th, 59 ms 99th, 120 ms 99.9th.
20000000 records sent, 523916.802012 records/sec (5.00 MB/sec), 10.58 ms avg latency, 219.00 ms max latency, 4 ms 50th, 38 ms 95th, 53 ms 99th, 73 ms 99.9th.

Average: 501,373.6 records/sec (4.78 MB/sec), 11.91 ms avg latency, 398.4 ms avg max latency
```

**record-size: 50**
```
20000000 records sent, 439241.868535 records/sec (20.94 MB/sec), 11.36 ms avg latency, 203.00 ms max latency, 4 ms 50th, 42 ms 95th, 58 ms 99th, 80 ms 99.9th.
20000000 records sent, 446558.153035 records/sec (21.29 MB/sec), 12.56 ms avg latency, 250.00 ms max latency, 4 ms 50th, 40 ms 95th, 71 ms 99th, 212 ms 99.9th.
20000000 records sent, 457948.847114 records/sec (21.84 MB/sec), 12.41 ms avg latency, 305.00 ms max latency, 4 ms 50th, 45 ms 95th, 89 ms 99th, 136 ms 99.9th.
20000000 records sent, 448239.539210 records/sec (21.37 MB/sec), 12.02 ms avg latency, 243.00 ms max latency, 3 ms 50th, 39 ms 95th, 74 ms 99th, 154 ms 99.9th.
20000000 records sent, 508517.670989 records/sec (24.25 MB/sec), 10.81 ms avg latency, 214.00 ms max latency, 4 ms 50th, 39 ms 95th, 60 ms 99th, 112 ms 99.9th.

Average: 460,101.2 records/sec (21.94 MB/sec), 11.83 ms avg latency, 243 ms avg max latency
```

**record-size: 100**
```
20000000 records sent, 454710.803929 records/sec (43.36 MB/sec), 26.16 ms avg latency, 628.00 ms max latency, 9 ms 50th, 391 ms 95th, 573 ms 99th, 619 ms 99.9th.
20000000 records sent, 421008.314914 records/sec (40.15 MB/sec), 23.00 ms avg latency, 649.00 ms max latency, 7 ms 50th, 336 ms 95th, 518 ms 99th, 630 ms 99.9th.
20000000 records sent, 432591.439015 records/sec (41.26 MB/sec), 20.11 ms avg latency, 575.00 ms max latency, 5 ms 50th, 51 ms 95th, 109 ms 99th, 173 ms 99.9th.
20000000 records sent, 387987.894778 records/sec (37.00 MB/sec), 23.93 ms avg latency, 760.00 ms max latency, 4 ms 50th, 55 ms 95th, 108 ms 99th, 159 ms 99.9th.
20000000 records sent, 415006.640106 records/sec (39.58 MB/sec), 21.72 ms avg latency, 616.00 ms max latency, 5 ms 50th, 46 ms 95th, 86 ms 99th, 123 ms 99.9th.

Average: 422261.0 records/sec (40.27 MB/sec), 22.98 ms avg latency, 645.6 ms avg max latency 
```
**WIP - record-size: 1000**
```
20000000 records sent, 45113.019392 records/sec (43.02 MB/sec), 709.53 ms avg latency, 30154.00 ms max latency, 30 ms 50th, 698 ms 95th, 874 ms 99th, 948 ms 99.9th.
```
