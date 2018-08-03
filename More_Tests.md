# More Kafka Benchmark Examples
[Taken from this Github Repo here](https://gist.github.com/dongjinleekr/d24e3d0c7f92ac0f80c87218f1f5a02b)


## Create your topics that we will be using in our benchmark tests with the DC/OS CLI:

#### Producer Throughput: Single producer thread, no replication, no compression - (topic: benchmark-1-1-none)
```
dcos confluent-kafka topic create benchmark-1-1-none --partitions 1 --replication 1
```

#### Producer Throughput: Single producer thread, 3x asynchronous replication, no compression - (topic: benchmark-1-3a-none)
```
dcos confluent-kafka topic create benchmark-1-3a-none --partitions 1 --replication 3
```

#### Producer Throughput: Single producer thread, 3x synchronous replication, no compression - (topic: benchmark-1-3-none)
```
dcos confluent-kafka topic create benchmark-1-3-none --partitions 1 --replication 3
```

#### Producer Throughput: 3 producer thread, no replication, no compression - (topic: benchmark-3-0-none)
```
dcos confluent-kafka topic create benchmark-3-0-none --partitions 1
```

#### Producer Throughput: 3 producer thread, 3x asynchronous replication, no compression - (topic: benchmark-3-3a-none)
```
dcos confluent-kafka topic create benchmark-3-3a-none --partitions 1 --replication 3
```

#### Producer Throughput: 3 producer thread, 3x synchronous replication, no compression - (topic: benchmark-3-3-none)
```
dcos confluent-kafka topic create benchmark-3-3-none --partitions 1 --replication 3
```

## In a new terminal window SSH into a node and run the Confluent Kafka Docker Image

SSH Options:
```
ssh -i <private_key> <user>@<node>

dcos node ssh --master-proxy --mesos-id=<MESOS_ID> --user=<OS_USER>
```

The command below will run the Confluent Kafka docker image which contains multiple tools that we can use to produce, consume, and performance test our Kafka deployment
```
docker run -it confluentinc/cp-kafka /bin/bash
```

## Run Tests

### Standard Compression Tests

#### Producer Throughput: Single producer thread, no replication, no compression - (topic: benchmark-1-1-none) 
```
kafka-producer-perf-test --topic benchmark-1-1-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Example output:
```
15000000 records sent, 183340.463240 records/sec (17.48 MB/sec), 3083.47 ms avg latency, 3614.00 ms max latency, 3182 ms 50th, 3497 ms 95th, 3586 ms 99th, 3610 ms 99.9th.
```

#### Producer Throughput: Single producer thread, 3x asynchronous replication, no compression - (topic: benchmark-1-3a-none)
```
kafka-producer-perf-test --topic benchmark-1-3a-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Example Output:
```
15000000 records sent, 122944.773208 records/sec (11.72 MB/sec), 4716.28 ms avg latency, 5404.00 ms max latency, 4834 ms 50th, 5243 ms 95th, 5369 ms 99th, 5392 ms 99.9th.
```

#### Producer Throughput: Single producer thread, 3x synchronous replication, no compression - (topic: benchmark-1-3-none)
```
kafka-producer-perf-test --topic benchmark-1-3-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Example Output:
```
15000000 records sent, 123582.915898 records/sec (11.79 MB/sec), 4693.13 ms avg latency, 6097.00 ms max latency, 4732 ms 50th, 5546 ms 95th, 5989 ms 99th, 6051 ms 99.9th.
```

#### Producer Throughput: 3 producer thread, no replication, no compression - (topic: benchmark-3-0-none)
```
kafka-producer-perf-test --topic benchmark-3-0-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Example Output:
```
15000000 records sent, 125746.093488 records/sec (11.99 MB/sec), 4600.36 ms avg latency, 5393.00 ms max latency, 4719 ms 50th, 5156 ms 95th, 5279 ms 99th, 5359 ms 99.9th.
```

#### Producer Throughput: 3 producer thread, 3x asynchronous replication, no compression - (topic: benchmark-3-3a-none)
```
kafka-producer-perf-test --topic benchmark-3-3a-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Example Output:
```
15000000 records sent, 117416.829746 records/sec (11.20 MB/sec), 4956.66 ms avg latency, 6395.00 ms max latency, 4958 ms 50th, 5766 ms 95th, 6164 ms 99th, 6300 ms 99.9th.
```

#### Producer Throughput: 3 producer thread, 3x synchronous replication, no compression - (topic: benchmark-3-3-none)
```
kafka-producer-perf-test --topic benchmark-3-3-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Example Output:
```
15000000 records sent, 131762.721691 records/sec (12.57 MB/sec), 4374.54 ms avg latency, 5272.00 ms max latency, 4491 ms 50th, 5007 ms 95th, 5111 ms 99th, 5210 ms 99.9th.
```
