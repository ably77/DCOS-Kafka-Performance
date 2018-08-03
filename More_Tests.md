# More Kafka Benchmark Examples
[Taken from this Github Repo here](https://gist.github.com/dongjinleekr/d24e3d0c7f92ac0f80c87218f1f5a02b)


## Create your topics that we will be using in our benchmark tests with the DC/OS CLI:

#### Producer Throughput: Single producer thread, no replication, no compression - (topic: benchmark-1-1-none)
```
dcos confluent-kafka topic create benchmark-1-1-none --partitions 1 --replication 1
```

#### Producer Throughput: Single producer thread, no replication, zstandard compression - (topic: benchmark-1-1-zstd)
```
dcos confluent-kafka topic create benchmark-1-1-zstd --partitions 1 --replication 1
```

#### Producer Throughput: Single producer thread, 3x asynchronous replication, no compression - (topic: benchmark-1-3a-none)
```
dcos confluent-kafka topic create benchmark-1-3a-none --partitions 1 --replication 3
```

#### Producer Throughput: Single producer thread, 3x asynchronous replication, zstandard compression - (topic: benchmark-1-3a-zstd)
```
dcos confluent-kafka topic create benchmark-1-3a-zstd --partitions 1 --replication 3
```

#### Producer Throughput: Single producer thread, 3x synchronous replication, no compression - (topic: benchmark-1-3-none)
```
dcos confluent-kafka topic create benchmark-1-3-none --partitions 1 --replication 3
```

#### Producer Throughput: Single producer thread, 3x synchronous replication, zstandard compression - (topic: benchmark-1-3-zstd)
```
dcos confluent-kafka topic create benchmark-1-3-zstd --partitions 1 --replication 3
```

#### Producer Throughput: 3 producer thread, no replication, no compression - (topic: benchmark-3-0-none)
```
dcos confluent-kafka topic create benchmark-3-0-none --partitions 1
```

#### Producer Throughput: 3 producer thread, no replication, zstandard compression - (topic: benchmark-3-0-zstd)
```
dcos confluent-kafka topic create benchmark-3-0-zstd --partitions 1
```

#### Producer Throughput: 3 producer thread, 3x asynchronous replication, no compression - (topic: benchmark-3-3a-none)
```
dcos confluent-kafka topic create benchmark-3-3a-none --partitions 1 --replication 3
```

#### Producer Throughput: 3 producer thread, 3x asynchronous replication, zstandard compression - (topic: benchmark-3-3a-zstd)
```
dcos confluent-kafka topic create benchmark-3-3a-zstd --partitions 1 --replication 3
```

#### Producer Throughput: 3 producer thread, 3x synchronous replication, no compression - (topic: benchmark-3-3-none)
```
dcos confluent-kafka topic create benchmark-3-3-none --partitions 1 --replication 3
```

#### Producer Throughput: 3 producer thread, 3x synchronous replication, zstandard compression - (topic: benchmark-3-3-zstd)
```
dcos confluent-kafka topic create benchmark-3-3-zstd --partitions 1 --replication 3
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

### Producer Throughput: Single producer thread, no replication, no compression
```
kafka-producer-perf-test --topic benchmark-1-1-none --num-records 15000000 --record-size 100 --throughput 15000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Output:
```
594449 records sent, 118889.8 records/sec (11.34 MB/sec), 1052.4 ms avg latency, 1804.0 max latency.
867428 records sent, 173485.6 records/sec (16.54 MB/sec), 2462.6 ms avg latency, 3133.0 max latency.
924260 records sent, 184852.0 records/sec (17.63 MB/sec), 3274.7 ms avg latency, 3354.0 max latency.
949864 records sent, 189972.8 records/sec (18.12 MB/sec), 3182.5 ms avg latency, 3268.0 max latency.
945794 records sent, 188405.2 records/sec (17.97 MB/sec), 3238.5 ms avg latency, 3309.0 max latency.
957708 records sent, 191541.6 records/sec (18.27 MB/sec), 3165.0 ms avg latency, 3248.0 max latency.
978206 records sent, 195641.2 records/sec (18.66 MB/sec), 3130.9 ms avg latency, 3242.0 max latency.
939356 records sent, 187533.6 records/sec (17.88 MB/sec), 3188.0 ms avg latency, 3270.0 max latency.
899988 records sent, 179459.2 records/sec (17.11 MB/sec), 3273.0 ms avg latency, 3380.0 max latency.
944461 records sent, 188892.2 records/sec (18.01 MB/sec), 3376.1 ms avg latency, 3614.0 max latency.
951936 records sent, 190387.2 records/sec (18.16 MB/sec), 3143.5 ms avg latency, 3220.0 max latency.
974950 records sent, 194990.0 records/sec (18.60 MB/sec), 3110.3 ms avg latency, 3195.0 max latency.
903244 records sent, 180648.8 records/sec (17.23 MB/sec), 3379.7 ms avg latency, 3564.0 max latency.
964039 records sent, 192807.8 records/sec (18.39 MB/sec), 3133.2 ms avg latency, 3225.0 max latency.
923923 records sent, 184784.6 records/sec (17.62 MB/sec), 3236.9 ms avg latency, 3362.0 max latency.
982054 records sent, 196410.8 records/sec (18.73 MB/sec), 3165.4 ms avg latency, 3257.0 max latency.
15000000 records sent, 183340.463240 records/sec (17.48 MB/sec), 3083.47 ms avg latency, 3614.00 ms max latency, 3182 ms 50th, 3497 ms 95th, 3586 ms 99th, 3610 ms 99.9th.
```

### Producer Throughput: Single producer thread, no replication, zstandard compression
```
