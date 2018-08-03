# Performance Testing Confluent-Kafka
Curious to see what the baseline benchmark performance of our Confluent Kafka framework? Here is a guide that will take you through basic performance testing, as well as expand into other areas of how to begin performance tuning your Kafka cluster based on Confluent Kafka best practices

## Prerequisites
For this guide, the specs of my cluster are as stated below:
- DC/OS 1.11
- 1 Master
- 8 Private Agents (Overprovisioned to scale)
- 1 Public Agent
- DC/OS CLI Installed and authenticated to your Local Machine

- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM, 80GiB SSD Storage, High Network Performance

## Step 1: Install Confluent Kafka
```
dcos package install confluent-kafka --yes
```

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
dcos confluent-kafka topic create performancetest --partitions 5 --replication 3
```

Output should look similar to below:
```
$ dcos confluent-kafka topic create performancetest --partitions 5 --replication 3
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
docker run -it confluentinc/cp-kafka /bin/bash
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
kafka-producer-perf-test --topic performancetest --num-records 2000000 --record-size 1000 --throughput 100000 --producer-props bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

In this test we are using the following parameters:
- Topic: performancetest
- Number of Records: 2000000
- Record Size: 1000 bytes
- Throughput: 100K messages/second
- Acknowledgements: 1
	- This parameter allows Kafka to acknowledge 1 write before allowing the remaining 2 replicas to write in the background
	- You can also tune to use acks=all to acknowledge all 3 writes before returning

Output of the test should look similar to below:
```
276363 records sent, 55272.6 records/sec (52.71 MB/sec), 437.3 ms avg latency, 1488.0 max latency.
328993 records sent, 65798.6 records/sec (62.75 MB/sec), 515.5 ms avg latency, 1696.0 max latency.
418642 records sent, 83728.4 records/sec (79.85 MB/sec), 405.3 ms avg latency, 1235.0 max latency.
547004 records sent, 109400.8 records/sec (104.33 MB/sec), 297.0 ms avg latency, 619.0 max latency.
2000000 records sent, 81846.456048 records/sec (78.05 MB/sec), 380.70 ms avg latency, 1696.00 ms max latency, 298 ms 50th, 1219 ms 95th, 1522 ms 99th, 1679 ms 99.9th.
```

You can also append the `--print-metrics` flag to the performance test to retrieve more descriptive metrics information:
```
kafka-producer-perf-test --topic performancetest --num-records 2000000 --record-size 1000 --throughput 100000 --print-metrics --producer-props bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Output should look similar to below:
```
378385 records sent, 75480.8 records/sec (71.98 MB/sec), 330.6 ms avg latency, 890.0 max latency.
364696 records sent, 72939.2 records/sec (69.56 MB/sec), 450.9 ms avg latency, 1397.0 max latency.
512443 records sent, 102488.6 records/sec (97.74 MB/sec), 323.7 ms avg latency, 817.0 max latency.
460536 records sent, 92107.2 records/sec (87.84 MB/sec), 360.1 ms avg latency, 1180.0 max latency.
2000000 records sent, 83381.972817 records/sec (79.52 MB/sec), 374.95 ms avg latency, 1805.00 ms max latency, 310 ms 50th, 1153 ms 95th, 1685 ms 99th, 1784 ms 99.9th.

Metric Name                                                                               Value
app-info:commit-id:{client-id=producer-1}                                               : 4b1dd33f255ddd2f
app-info:version:{client-id=producer-1}                                                 : 2.0.0-cp1
kafka-metrics-count:count:{client-id=producer-1}                                        : 142.000
producer-metrics:batch-size-avg:{client-id=producer-1}                                  : 14069.442
producer-metrics:batch-size-max:{client-id=producer-1}                                  : 16220.000
producer-metrics:batch-split-rate:{client-id=producer-1}                                : 0.000
producer-metrics:batch-split-total:{client-id=producer-1}                               : 0.000
producer-metrics:buffer-available-bytes:{client-id=producer-1}                          : 33554432.000
producer-metrics:buffer-exhausted-rate:{client-id=producer-1}                           : 0.000
producer-metrics:buffer-exhausted-total:{client-id=producer-1}                          : 0.000
producer-metrics:buffer-total-bytes:{client-id=producer-1}                              : 33554432.000
producer-metrics:bufferpool-wait-ratio:{client-id=producer-1}                           : 0.263
producer-metrics:bufferpool-wait-time-total:{client-id=producer-1}                      : 13927448187.000
producer-metrics:compression-rate-avg:{client-id=producer-1}                            : 1.000
producer-metrics:connection-close-rate:{client-id=producer-1}                           : 0.000
producer-metrics:connection-close-total:{client-id=producer-1}                          : 0.000
producer-metrics:connection-count:{client-id=producer-1}                                : 6.000
producer-metrics:connection-creation-rate:{client-id=producer-1}                        : 0.111
producer-metrics:connection-creation-total:{client-id=producer-1}                       : 6.000
producer-metrics:failed-authentication-rate:{client-id=producer-1}                      : 0.000
producer-metrics:failed-authentication-total:{client-id=producer-1}                     : 0.000
producer-metrics:incoming-byte-rate:{client-id=producer-1}                              : 135685.456
producer-metrics:incoming-byte-total:{client-id=producer-1}                             : 7310461.000
producer-metrics:io-ratio:{client-id=producer-1}                                        : 0.093
producer-metrics:io-time-ns-avg:{client-id=producer-1}                                  : 35017.561
producer-metrics:io-wait-ratio:{client-id=producer-1}                                   : 0.156
producer-metrics:io-wait-time-ns-avg:{client-id=producer-1}                             : 58567.036
producer-metrics:io-waittime-total:{client-id=producer-1}                               : 8432716085.000
producer-metrics:iotime-total:{client-id=producer-1}                                    : 5041968490.000
producer-metrics:metadata-age:{client-id=producer-1}                                    : 23.868
producer-metrics:network-io-rate:{client-id=producer-1}                                 : 3360.095
producer-metrics:network-io-total:{client-id=producer-1}                                : 2040409990.000
producer-metrics:outgoing-byte-rate:{client-id=producer-1}                              : 37733844.265
producer-metrics:outgoing-byte-total:{client-id=producer-1}                             : 2033099529.000
producer-metrics:produce-throttle-time-avg:{client-id=producer-1}                       : 0.000
producer-metrics:produce-throttle-time-max:{client-id=producer-1}                       : 0.000
producer-metrics:record-error-rate:{client-id=producer-1}                               : 0.000
producer-metrics:record-error-total:{client-id=producer-1}                              : 0.000
producer-metrics:record-queue-time-avg:{client-id=producer-1}                           : 322.219
producer-metrics:record-queue-time-max:{client-id=producer-1}                           : 1800.000
producer-metrics:record-retry-rate:{client-id=producer-1}                               : 0.000
producer-metrics:record-retry-total:{client-id=producer-1}                              : 0.000
producer-metrics:record-send-rate:{client-id=producer-1}                                : 37156.074
producer-metrics:record-send-total:{client-id=producer-1}                               : 2000000.000
producer-metrics:record-size-avg:{client-id=producer-1}                                 : 1086.000
producer-metrics:record-size-max:{client-id=producer-1}                                 : 1086.000
producer-metrics:records-per-request-avg:{client-id=producer-1}                         : 22.096
producer-metrics:request-latency-avg:{client-id=producer-1}                             : 3.520
producer-metrics:request-latency-max:{client-id=producer-1}                             : 845.000
producer-metrics:request-rate:{client-id=producer-1}                                    : 1680.079
producer-metrics:request-size-avg:{client-id=producer-1}                                : 22458.736
producer-metrics:request-size-max:{client-id=producer-1}                                : 32513.000
producer-metrics:request-total:{client-id=producer-1}                                   : 2033099529.000
producer-metrics:requests-in-flight:{client-id=producer-1}                              : 0.000
producer-metrics:response-rate:{client-id=producer-1}                                   : 1680.203
producer-metrics:response-total:{client-id=producer-1}                                  : 7310461.000
producer-metrics:select-rate:{client-id=producer-1}                                     : 2664.989
producer-metrics:select-total:{client-id=producer-1}                                    : 8432716085.000
producer-metrics:successful-authentication-rate:{client-id=producer-1}                  : 0.000
producer-metrics:successful-authentication-total:{client-id=producer-1}                 : 0.000
producer-metrics:waiting-threads:{client-id=producer-1}                                 : 0.000
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--1}        : 4.604
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--2}        : 4.603
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--3}        : 11.359
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-0}         : 54174.459
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-1}         : 43213.116
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-2}         : 38405.654
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--1}       : 248.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--2}       : 248.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--3}       : 612.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-0}        : 2916482.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-1}        : 2325341.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-2}        : 2067530.000
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--1}        : 0.891
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--2}        : 0.891
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--3}        : 1.745
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-0}         : 15106794.598
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-1}         : 15084803.897
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-2}         : 7573923.766
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--1}       : 48.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--2}       : 48.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--3}       : 94.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-0}        : 813289394.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-1}        : 812075333.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-2}        : 407734612.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--1}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--2}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--3}       : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-0}        : 3.264
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-1}        : 4.672
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-2}        : 2.899
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--1}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--2}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--3}       : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-0}        : 471.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-1}        : 845.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-2}        : 89.000
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--1}              : 0.037
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--2}              : 0.037
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--3}              : 0.056
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-0}               : 607.389
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-1}               : 464.438
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-2}               : 609.566
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--1}          : 24.000
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--2}          : 24.000
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--3}          : 31.333
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-0}           : 24871.235
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-1}           : 32479.116
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-2}           : 12424.872
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--1}          : 24.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--2}          : 24.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--3}          : 46.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-0}           : 32483.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-1}           : 32513.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-2}           : 16270.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--1}             : 48.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--2}             : 48.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--3}             : 94.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-0}              : 813289394.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-1}              : 812075333.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-2}              : 407734612.000
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--1}             : 0.037
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--2}             : 0.037
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--3}             : 0.056
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-0}              : 607.412
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-1}              : 464.645
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-2}              : 609.578
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--1}            : 248.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--2}            : 248.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--3}            : 612.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-0}             : 2916482.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-1}             : 2325341.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-2}             : 2067530.000
producer-topic-metrics:byte-rate:{client-id=producer-1, topic=performancetest}          : 37655132.485
producer-topic-metrics:byte-total:{client-id=producer-1, topic=performancetest}         : 2026787506.000
producer-topic-metrics:compression-rate:{client-id=producer-1, topic=performancetest}   : 1.000
producer-topic-metrics:record-error-rate:{client-id=producer-1, topic=performancetest}  : 0.000
producer-topic-metrics:record-error-total:{client-id=producer-1, topic=performancetest} : 0.000
producer-topic-metrics:record-retry-rate:{client-id=producer-1, topic=performancetest}  : 0.000
producer-topic-metrics:record-retry-total:{client-id=producer-1, topic=performancetest} : 0.000
producer-topic-metrics:record-send-rate:{client-id=producer-1, topic=performancetest}   : 37157.455
producer-topic-metrics:record-send-total:{client-id=producer-1, topic=performancetest}  : 2000000.000
```

## Conclusion
In this example I have tested the base 3 Kafka broker node deploy on a DC/OS Cluster running on AWS m3.xlarge instances. From my test observations with these fixed parameters set:
- Number of Records: 20M
- Throughput: 5M (Set arbitrarily high to "max out")

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
