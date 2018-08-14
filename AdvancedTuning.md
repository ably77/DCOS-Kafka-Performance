# Advanced Load Testing Kafka
Lets take our prior example and expand on it. We're going to try to change up some parameters and see what performance we get

## Prerequisites
To start, the specs of my cluster are as stated below:
- DC/OS 1.11
- 1 Master
- 5 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine

- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM [See here for more recommended instance types by Confluent](https://www.confluent.io/blog/design-and-deployment-considerations-for-deploying-apache-kafka-on-aws/) 
	- EBS Backed Storage - 60 GB

### Default Kafka Framework Parameters
Note that the default Kafka package has these specifications for brokers:
- 3x Brokers
- 1 CPU
- 2048 MEM
- 5000 MB Disk
- 512 MB JVM Heap Size

For our Advanced Guide we will use a larger Kafka cluster size to observe performance improvements:

### Our Advanced Kafka Framework Parameters
- 3x Brokers
- 3 CPU
- 12GB MEM
- 25 GB Disk
- 512 MB JVM Heap Size

### If you have an Existing Kafka Deployment

If you were following the Quickstart guide before this, we deployed the default Kafka framework with the specs listed above.

The Kafka framework does not support changing the volume requirements after initial deployment in order to prevent accidental data loss from reallocation. This will require an uninstall, and reinstall of the Kafka deployment since we are testing a volume requirement 25GB Disk instead of the default 5GB.

**Optional:** If you are already using the service name `confluent-kafka` and cannot uninstall the deployment, you can follow this guide with a second Kafka cluster if you have the resources available. Otherwise, if you want to just continue to use the default 5GB storage volume requirement you can just scale using the commands below as well.

To Uninstall Kafka:
```
dcos package uninstall confluent-kafka --yes
```

To Update Kafka, grab the configuration `options.json`:
```
dcos confluent-kafka describe > options.json
```

Make edits and submit Service update using:
```
dcos confluent-kafka update start --options=options.json
```

Update fields below:
```
"count": 3,
"cpus": 3,
    },
    "mem": 12000,
```

**Note:** Cannot change the storage requirements for updates, unless it is a fresh cluster install.

If you do not have Kafka deployed yet, you can save the `options.json` configuration below and follow the instructions below, as you can see there are many parameters in Kafka that we can tune:
```
{
  "brokers": {
    "count": 3,
    "cpus": 3,
    "disk": 25000,
    "disk_path": "kafka-broker-data",
    "disk_type": "ROOT",
    "heap": {
      "size": 512
    },
    "kill_grace_period": 30,
    "mem": 12000,
    "port": 0,
    "port_tls": 0
  },
  "kafka": {
    "auto_create_topics_enable": true,
    "auto_leader_rebalance_enable": true,
    "background_threads": 10,
    "compression_type": "producer",
    "confluent_metrics_reporter_topic": "_confluent-metrics",
    "confluent_metrics_reporter_topic_replicas": 3,
    "confluent_support_customer_id": "anonymous",
    "confluent_support_metrics_enable": false,
    "connections_max_idle_ms": 600000,
    "controlled_shutdown_enable": true,
    "controlled_shutdown_max_retries": 3,
    "controlled_shutdown_retry_backoff_ms": 5000,
    "controller_socket_timeout_ms": 30000,
    "default_replication_factor": 1,
    "delete_records_purgatory_purge_interval_requests": 1,
    "delete_topic_enable": false,
    "fetch_purgatory_purge_interval_requests": 1000,
    "group_initial_rebalance_delay_ms": 3000,
    "group_max_session_timeout_ms": 300000,
    "group_min_session_timeout_ms": 6000,
    "inter_broker_protocol_version": "1.0",
    "kafka_advertise_host_ip": true,
    "kafka_metrics_reporters": "com.airbnb.kafka.kafka08.StatsdMetricsReporter",
    "kafka_zookeeper_uri": "",
    "leader_imbalance_check_interval_seconds": 300,
    "leader_imbalance_per_broker_percentage": 10,
    "log_cleaner_backoff_ms": 15000,
    "log_cleaner_dedupe_buffer_size": 134217728,
    "log_cleaner_delete_retention_ms": 86400000,
    "log_cleaner_enable": true,
    "log_cleaner_io_buffer_load_factor": 0.9,
    "log_cleaner_io_buffer_size": 524288,
    "log_cleaner_io_max_bytes_per_second": 1.7976931348623157e+308,
    "log_cleaner_min_cleanable_ratio": 0.5,
    "log_cleaner_min_compaction_lag_ms": 0,
    "log_cleaner_threads": 1,
    "log_cleanup_policy": "delete",
    "log_flush_interval_messages": "9223372036854775807",
    "log_flush_offset_checkpoint_interval_ms": 60000,
    "log_flush_scheduler_interval_ms": "9223372036854775807",
    "log_flush_start_offset_checkpoint_interval_ms": 60000,
    "log_index_interval_bytes": 4096,
    "log_index_size_max_bytes": 10485760,
    "log_message_format_version": "1.0",
    "log_preallocate": false,
    "log_retention_bytes": "-1",
    "log_retention_check_interval_ms": 300000,
    "log_retention_hours": 168,
    "log_retention_minutes": 10,
    "log_roll_hours": 168,
    "log_roll_jitter_hours": 0,
    "log_segment_bytes": 1073741824,
    "log_segment_delete_delay_ms": 60000,
    "max_connections_per_ip": 2147483647,
    "max_connections_per_ip_overrides": "",
    "message_max_bytes": 1000012,
    "metric_reporters": "com.airbnb.kafka.kafka09.StatsdMetricsReporter,io.confluent.metrics.reporter.ConfluentMetricsReporter",
    "metrics_num_samples": 2,
    "metrics_sample_window_ms": 30000,
    "min_insync_replicas": 1,
    "num_io_threads": 8,
    "num_network_threads": 3,
    "num_partitions": 1,
    "num_recovery_threads_per_data_dir": 1,
    "num_replica_fetchers": 1,
    "offset_metadata_max_bytes": 4096,
    "offsets_commit_required_acks": -1,
    "offsets_commit_timeout_ms": 5000,
    "offsets_load_buffer_size": 5242880,
    "offsets_retention_check_interval_ms": 600000,
    "offsets_retention_minutes": 1440,
    "offsets_topic_compression_codec": 0,
    "offsets_topic_num_partitions": 50,
    "offsets_topic_replication_factor": 3,
    "offsets_topic_segment_bytes": 104857600,
    "producer_purgatory_purge_interval_requests": 1000,
    "queued_max_request_bytes": -1,
    "queued_max_requests": 500,
    "quota_consumer_default": "9223372036854775807",
    "quota_producer_default": "9223372036854775807",
    "quota_window_num": 11,
    "quota_window_size_seconds": 1,
    "replica_fetch_backoff_ms": 1000,
    "replica_fetch_max_bytes": 1048576,
    "replica_fetch_min_bytes": 1,
    "replica_fetch_response_max_bytes": 10485760,
    "replica_fetch_wait_max_ms": 500,
    "replica_high_watermark_checkpoint_interval_ms": 5000,
    "replica_lag_time_max_ms": 10000,
    "replica_socket_receive_buffer_bytes": 65536,
    "replica_socket_timeout_ms": 30000,
    "replication_quota_window_num": 11,
    "replication_quota_window_size_seconds": 1,
    "request_timeout_ms": 30000,
    "reserved_broker_max_id": 1000,
    "socket_receive_buffer_bytes": 102400,
    "socket_request_max_bytes": 104857600,
    "socket_send_buffer_bytes": 102400,
    "transaction_abort_timed_out_transaction_cleanup_interval_ms": 60000,
    "transaction_max_timeout_ms": 900000,
    "transaction_remove_expired_transaction_cleanup_interval_ms": 3600000,
    "transaction_state_log_load_buffer_size": 5242880,
    "transaction_state_log_min_isr": 2,
    "transaction_state_log_num_partitions": 50,
    "transaction_state_log_replication_factor": 3,
    "transaction_state_log_segment_bytes": 104857600,
    "transactional_id_expiration_ms": 604800000,
    "unclean_leader_election_enable": false,
    "zookeeper_session_timeout_ms": 6000,
    "zookeeper_sync_time_ms": 2000
  },
  "service": {
    "deploy_strategy": "serial",
    "log_level": "INFO",
    "mesos_api_version": "V1",
    "name": "confluent-kafka",
    "placement_constraint": "[[\"hostname\", \"MAX_PER\", \"1\"]]",
    "region": "",
    "security": {
      "authorization": {
        "allow_everyone_if_no_acl_found": false,
        "enabled": false,
        "super_users": ""
      },
      "kerberos": {
        "debug": false,
        "enabled": false,
        "enabled_for_zookeeper": false,
        "kdc": {},
        "primary": "kafka"
      },
      "ssl_authentication": {
        "enabled": false
      },
      "transport_encryption": {
        "allow_plaintext": false,
        "ciphers": "TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_CBC_SHA256,TLS_DHE_RSA_WITH_AES_128_GCM_SHA256,TLS_DHE_RSA_WITH_AES_128_CBC_SHA256,TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_DHE_RSA_WITH_AES_256_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384",
        "enabled": false
      }
    },
    "service_account": "",
    "service_account_secret": "",
    "user": "nobody",
    "virtual_network_enabled": false,
    "virtual_network_name": "dcos",
    "virtual_network_plugin_labels": ""
  }
}
```

## Step 1: Install Confluent Kafka
```
dcos package install confluent-kafka --options=options.json --yes
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

## Step 2: Add a test topic from the DC/OS CLI
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

Notice that the broker DNS hostnames follow a common pattern `kafka-0-broker.<service-name>.autoip.dcos.thisdcos.directory:1025` provided by Mesos-DNS. This allows for DC/OS to manage service discovery when issues arise such as a broker failure. Using the DNS hostname we can abstract the need to know and reconfigure a static IP:port pairing.

## Step 4: Run the Confluent Kafka performance tests

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
- Compression Type: none
        - Can set to options: none, lz4, gzip, snappy

Here is the example application definition for our performance test service that we will call `250-baseline.json`
```
{
  "id": "/250-baseline",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "cmd": "kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=none batch.size=8196 linger.ms=0 retries=0 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025 && sleep 60",
  "constraints": [
    [
      "hostname",
      "UNIQUE"
    ]
  ],
  "container": {
    "type": "MESOS",
    "volumes": [],
    "docker": {
      "image": "confluentinc/cp-kafka",
      "forcePullImage": false,
      "parameters": []
    }
  },
  "cpus": 4,
  "disk": 0,
  "instances": 1,
  "maxLaunchDelaySeconds": 3600,
  "mem": 13000,
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
  "fetch": []
}
```

Description of Producer Service:
- 1x Instance to start
- 4 CPU
- 13GB MEM
- Constraint: HOSTNAME / UNIQUE
- Sleep 120 seconds and restart

Note: Note that running multiple producers from the same node is less effective in this situation because our bottleneck may start to come from other places, such as the NIC. Keeping the producers on seperate nodes is more ideal for our current testing case as we can then remove the Producer as the throughput bottleneck. In our case when we built our cluster we deployed an extra agent node, which the Producer will utilize.

Launch the marathon service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/250-baseline.json
```

Navigate to the DC/OS UI --> Services --> 250-baseline --> logs --> Output (stdout) to view performance test results:
```
(AT BEGINNING OF FILE)
Marked '/' as rslave
Prepared mount '{"flags":20480,"source":"\/var\/lib\/mesos\/slave\/slaves\/e77195fd-0c64-4b78-8d51-1223609f3b73-S0\/frameworks\/e77195fd-0c64-4b78-8d51-1223609f3b73-0001\/executors\/1producer-topic-performancetest.json.3ed2a367-9f28-11e8-982a-3a84e2ff092b\/runs\/0c5cf368-1735-4489-aa6f-edcb8c8b68c8","target":"\/var\/lib\/mesos\/slave\/provisioner\/containers\/0c5cf368-1735-4489-aa6f-edcb8c8b68c8\/backends\/overlay\/rootfses\/f10a0444-b960-4999-b3ed-d22dea5df960\/mnt\/mesos\/sandbox"}'
Prepared mount '{"flags":14,"source":"proc","target":"\/proc","type":"proc"}'
Executing pre-exec command '{"arguments":["mount","-n","-t","ramfs","ramfs","\/var\/lib\/mesos\/slave\/slaves\/e77195fd-0c64-4b78-8d51-1223609f3b73-S0\/frameworks\/e77195fd-0c64-4b78-8d51-1223609f3b73-0001\/executors\/1producer-topic-performancetest.json.3ed2a367-9f28-11e8-982a-3a84e2ff092b\/runs\/0c5cf368-1735-4489-aa6f-edcb8c8b68c8\/.secret-06519df1-4c54-4a9c-9798-9c9f538440df"],"shell":false,"value":"mount"}'
Changing root to /var/lib/mesos/slave/provisioner/containers/0c5cf368-1735-4489-aa6f-edcb8c8b68c8/backends/overlay/rootfses/f10a0444-b960-4999-b3ed-d22dea5df960
1127164 records sent, 225432.8 records/sec (53.75 MB/sec), 273.8 ms avg latency, 705.0 max latency.
2322632 records sent, 464526.4 records/sec (110.75 MB/sec), 147.6 ms avg latency, 719.0 max latency.
2507524 records sent, 501504.8 records/sec (119.57 MB/sec), 170.6 ms avg latency, 803.0 max latency.
2477901 records sent, 495580.2 records/sec (118.16 MB/sec), 114.3 ms avg latency, 456.0 max latency.
10000000 records sent, 434027.777778 records/sec (103.48 MB/sec), 150.00 ms avg latency, 803.00 ms max latency, 20 ms 50th, 342 ms 95th, 433 ms 99th, 459 ms 99.9th.
```

#### Initial Thoughts:
Here we can see that increasing the parameters of the Kafka framework we are able to increase the throughput performance almost double to > 400k messages per second.

Remove the service:
```
dcos marathon app remove 250-baseline
```


### Kafka Consumer Performance Testing

In this test we are using the following parameters:
- Topic: performancetest
- Number of Messages to Consume: 1.5M
- Threads: 1

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/1consumer-topic-performancetest.json
```

Navigate to the DC/OS UI --> Services --> 1consumer-topic-performancetest --> logs --> Output (stdout) to view performance test results. Example Output (Edited for readability):
```
start.time - 2018-08-13 18:55:38:126
end.time - 2018-08-13 18:56:12:282
data.consumed.in.MB - 3576.2863
MB.sec - 104.7045
data.consumed.in.nMsg - 15000032
nMsg.sec - 439162.4312
rebalance.time.ms - 3021
fetch.time.ms - 31135
fetch.MB.sec - 114.8639
fetch.nMsg.sec - 481773.9521
```

Remove the Service:
```
dcos marathon app remove 1consumer-topic-performancetest
```

## Step 5: Understand baseline performance

My variable parameter was `record-size` in bytes which I averaged across 5 runs:

**Record Size: 250 bytes**:

Run the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/250-baseline.json
```

Example Output over 5 runs:
```
10000000 records sent, 452324.950244 records/sec (107.84 MB/sec), 94.85 ms avg latency, 776.00 ms max latency, 86 ms 50th, 330 ms 95th, 436 ms 99th, 533 ms 99.9th.
10000000 records sent, 457435.615937 records/sec (109.06 MB/sec), 102.11 ms avg latency, 660.00 ms max latency, 39 ms 50th, 485 ms 95th, 581 ms 99th, 647 ms 99.9th.
10000000 records sent, 450409.872984 records/sec (107.39 MB/sec), 49.25 ms avg latency, 393.00 ms max latency, 7 ms 50th, 196 ms 95th, 289 ms 99th, 318 ms 99.9th.
10000000 records sent, 416770.859382 records/sec (99.37 MB/sec), 141.61 ms avg latency, 2603.00 ms max latency, 20 ms 50th, 218 ms 95th, 398 ms 99th, 420 ms 99.9th.
10000000 records sent, 452652.543907 records/sec (107.92 MB/sec), 134.19 ms avg latency, 818.00 ms max latency, 74 ms 50th, 642 ms 95th, 692 ms 99th, 703 ms 99.9th.

Average: 445918.77 records/sec, 106.32 MB/sec, 104.4 ms avg latency, 1050 avg ms max latency
```

Remove the Service:
```
dcos marathon app remove 250-baseline
```

**Record Size: 500 bytes**:

Run the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/500-baseline.json
```

Example output over 5 runs:
```
10000000 records sent, 230973.553528 records/sec (110.14 MB/sec), 505.08 ms avg latency, 1318.00 ms max latency, 185 ms 50th, 526 ms 95th, 686 ms 99th, 760 ms 99.9th.
10000000 records sent, 229547.332660 records/sec (109.46 MB/sec), 508.49 ms avg latency, 1024.00 ms max latency, 265 ms 50th, 728 ms 95th, 881 ms 99th, 954 ms 99.9th.
10000000 records sent, 231873.304426 records/sec (110.57 MB/sec), 502.54 ms avg latency, 2119.00 ms max latency, 292 ms 50th, 902 ms 95th, 997 ms 99th, 1009 ms 99.9th.
10000000 records sent, 232018.561485 records/sec (110.64 MB/sec), 505.30 ms avg latency, 1601.00 ms max latency, 1029 ms 50th, 1413 ms 95th, 1549 ms 99th, 1579 ms 99.9th.
10000000 records sent, 236748.029073 records/sec (112.89 MB/sec), 493.91 ms avg latency, 1749.00 ms max latency, 177 ms 50th, 428 ms 95th, 475 ms 99th, 494 ms 99.9th.

Average: 232232.16 records/sec, 110.74 ms avg latency, 1562.2 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 500-baseline
```

**Record Size: 1 kB**

Run the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/1000-baseline.json
```

Example output over 5 runs:
```
10000000 records sent, 107101.928906 records/sec (102.14 MB/sec), 598.85 ms avg latency, 2896.00 ms max latency, 3 ms 50th, 585 ms 95th, 689 ms 99th, 710 ms 99.9th.
10000000 records sent, 63005.223133 records/sec (60.09 MB/sec), 1030.05 ms avg latency, 22907.00 ms max latency, 401 ms 50th, 1785 ms 95th, 22628 ms 99th, 22869 ms 99.9th.
10000000 records sent, 113606.670984 records/sec (108.34 MB/sec), 564.88 ms avg latency, 2677.00 ms max latency, 8 ms 50th, 922 ms 95th, 2120 ms 99th, 2662 ms 99.9th.
10000000 records sent, 63212.723457 records/sec (60.28 MB/sec), 942.20 ms avg latency, 39612.00 ms max latency, 811 ms 50th, 1754 ms 95th, 30059 ms 99th, 31870 ms 99.9th.
10000000 records sent, 109065.526568 records/sec (104.01 MB/sec), 587.74 ms avg latency, 5129.00 ms max latency, 1105 ms 50th, 1749 ms 95th, 1881 ms 99th, 1919 ms 99.9th.

Average: 91198.41 records/sec, 86.97 ms avg latency, 14644 ms avg max latency
```

### Initial Thoughts:
With a single isolated producer, you can see that our cluster can handle high throughput, but latency performance is clearly an issue. We will modify the tests to try and find an ideal high throughput and latency balance.

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
- batch.size - 100000
- linger.ms - 10
- compression.type - lz4
- acks - 1
- buffer.memory - default

Deploy the service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/1producer-lower-topic-performancetest.json
```

Example Output in the logs:
```
10000000 records sent, 717360.114778 records/sec (171.03 MB/sec), 8.83 ms avg latency, 226.00 ms max latency, 8 ms 50th, 13 ms 95th, 17 ms 99th, 25 ms 99.9th.
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
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/1producer-higher-topic-performancetest.json
```

Example Output in the logs:
```
10000000 records sent, 610761.619740 records/sec (145.62 MB/sec), 70.41 ms avg latency, 336.00 ms max latency, 77 ms 50th, 129 ms 95th, 142 ms 99th, 160 ms 99.9th.
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
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/1consumer-higher-topic-performancetest.json
```

Output:
```
start.time - 2018-08-13 20:08:18:902
end.time - 2018-08-13 20:08:42:726
data.consumed.in.MB - 5090.1122
MB.sec - 213.6548
data.consumed.in.nMsg - 15000098
nMsg.sec - 629621.3062
rebalance.time.ms - 3021
fetch.time.ms - 20803
fetch.MB.sec - 244.6816
fetch.nMsg.sec - 721054.5594
```

Remove the Service:
```
dcos marathon app remove 1consumer-higher-topic-performancetest
```

### Conclusions

#### Producers
Lower Range - 60% increase in Throughput
Higher Range - 37% increase in Throughput

By tuning for throughput and increasing the batch.size, linger.ms, and compression.type parameters we can see a significant increase in throughput performance as well as latency performance of our Kafka cluster. For a 250 byte record it seems as though the lower end ranges are more ideal, resulting in >700K records/sec at a low 8.83 ms avg latency. The upper end also saw improvements in performance, but may be more ideal for a situation where the record size is much larger. 

For the rest of the testing, we will utilize the Lower Range parameters, but it would be advised to do more A/B testing within the range to optimize for your specific record-size

#### Consumers
Increasing fetch.min.bytes from 1 --> 1000000 resulted in a 43% increase in performance of our Consumer from 440K records/sec to 630K records/sec

## Horizontal Scale
Now that we have reached a "peak" in our current configuration (3CPU, 12GB MEM, 25GB DISK) lets horizontally scale our cluster to see what performance benefits we can gain. Begin so by adding some nodes to your DC/OS cluster. We started this guide with 5, and for the rest of this guide we will continue to scale test using up to 35 private agents

### DC/OS Cluster Prerequisites
- 1 Master
- 12 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine
- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM See here for more recommended instance types by Confluent
	- EBS Backed Storage - 60 GB

### Kafka Cluster Parameters
- 6x Brokers
- 3 CPU
- 12GB MEM
- 25 GB Disk
- 512 MB JVM Heap Size 

As you can see, nothing has changed above from our prior configuration except for scaling from 3 to 6 Kafka brokers. You can do so by passing an update command with an updated options.json file, or through the UI change Kafka broker count to 6.

To validate that our deployment is correct:
```
dcos confluent-kafka plan status deploy
```

Output should look similar to below:
```
$ dcos confluent-kafka plan status deploy
deploy (serial strategy) (COMPLETE)
└─ broker (serial strategy) (COMPLETE)
   ├─ kafka-0:[broker] (COMPLETE)
   ├─ kafka-1:[broker] (COMPLETE)
   ├─ kafka-2:[broker] (COMPLETE)
   ├─ kafka-3:[broker] (COMPLETE)
   ├─ kafka-4:[broker] (COMPLETE)
   └─ kafka-5:[broker] (COMPLETE)
```

### Run the Kafka Performance Test
Now lets run the same single producer Kafka performance test optimized for throughput as before on our 6 broker node Kafka cluster

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/1producer-lower-topic-performancetest.json
```

Example Output in Logs:
```
10000000 records sent, 780518.264127 records/sec (186.09 MB/sec), 8.70 ms avg latency, 214.00 ms max latency, 8 ms 50th, 13 ms 95th, 19 ms 99th, 28 ms 99.9th.
```

Remove the Service:
```
dcos marathon app remove 1producer-lower-topic-performancetest
```

As we can see from above, our throughput for a single producer hasnt increased too much, however in order to gain the benefits of horizontal scaling we will also throw multiple producers at the same topic to see how much total throughput we can get out of the Kafka deployment.

## Running Multiple Producers in Parallel
In order to attack this throughput problem with multiple producers in parallel, we will run the performance test as a service in DC/OS and scale it  to run multiple producers. Note that running multiple producers from the same node is less effective in this situation because our bottleneck may start to come from other places, such as the NIC. Keeping the producers on seperate nodes is more ideal for our current testing case as we can then remove the Producer as the throughput bottleneck.

Here is the example application definition for our performance test service that we will call `3producer-topic-performancetest.json`
```
{
  "id": "/3producer-topic-performancetest",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "cmd": "kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=lz4 batch.size=100000 linger.ms=10 retries=0 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025 && sleep 120",
  "constraints": [
    [
      "hostname",
      "UNIQUE"
    ]
  ],
  "container": {
    "type": "MESOS",
    "volumes": [],
    "docker": {
      "image": "confluentinc/cp-kafka",
      "forcePullImage": false,
      "parameters": []
    }
  },
  "cpus": 4,
  "disk": 0,
  "instances": 3,
  "maxLaunchDelaySeconds": 3600,
  "mem": 13000,
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
  "fetch": []
}
```

Description of Producer Service:
- 3x Instances to start
- 4 CPU
- 13GB MEM
- Constraint: HOSTNAME / UNIQUE
- Sleep 120 seconds and restart

Launch the marathon service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/3producer-topic-performancetest.json
```

Navigate to the UI --> Services --> confluent-producer --> logs --> Output (stdout) to view performance test results:
```
(AT BEGINNING OF FILE)
Marked '/' as rslave
Prepared mount '{"flags":20480,"source":"\/var\/lib\/mesos\/slave\/slaves\/e77195fd-0c64-4b78-8d51-1223609f3b73-S5\/frameworks\/e77195fd-0c64-4b78-8d51-1223609f3b73-0001\/executors\/3producer-topic-performancetest.bd9c6c11-9f38-11e8-982a-3a84e2ff092b\/runs\/23af5707-253d-4648-ac6c-e5363659bd35","target":"\/var\/lib\/mesos\/slave\/provisioner\/containers\/23af5707-253d-4648-ac6c-e5363659bd35\/backends\/overlay\/rootfses\/ec8ab944-100a-43b7-9433-08c799e4f9aa\/mnt\/mesos\/sandbox"}'
Prepared mount '{"flags":14,"source":"proc","target":"\/proc","type":"proc"}'
Executing pre-exec command '{"arguments":["mount","-n","-t","ramfs","ramfs","\/var\/lib\/mesos\/slave\/slaves\/e77195fd-0c64-4b78-8d51-1223609f3b73-S5\/frameworks\/e77195fd-0c64-4b78-8d51-1223609f3b73-0001\/executors\/3producer-topic-performancetest.bd9c6c11-9f38-11e8-982a-3a84e2ff092b\/runs\/23af5707-253d-4648-ac6c-e5363659bd35\/.secret-db1d03bf-3c20-4656-b2c6-c34bcff99d4d"],"shell":false,"value":"mount"}'
Changing root to /var/lib/mesos/slave/provisioner/containers/23af5707-253d-4648-ac6c-e5363659bd35/backends/overlay/rootfses/ec8ab944-100a-43b7-9433-08c799e4f9aa
3356567 records sent, 671313.4 records/sec (160.05 MB/sec), 9.9 ms avg latency, 233.0 max latency.
3845743 records sent, 769148.6 records/sec (183.38 MB/sec), 9.1 ms avg latency, 41.0 max latency.
10000000 records sent, 744934.445769 records/sec (177.61 MB/sec), 9.79 ms avg latency, 233.00 ms max latency, 8 ms 50th, 15 ms 95th, 24 ms 99th, 34 ms 99.9th.
```

### Example total throughput from 3 Producers
Since I have scaled up to 12 nodes in my DC/OS cluster but Kafka is only consuming 7 nodes, I have 5 isolated nodes available for my Producers to utilize for this test. First we will start with 3 Producers

Output from Logs:
```
10000000 records sent, 744934.445769 records/sec (177.61 MB/sec), 9.79 ms avg latency, 233.00 ms max latency, 8 ms 50th, 15 ms 95th, 24 ms 99th, 34 ms 99.9th.
10000000 records sent, 775614.674630 records/sec (184.92 MB/sec), 9.21 ms avg latency, 216.00 ms max latency, 9 ms 50th, 17 ms 95th, 30 ms 99th, 52 ms 99.9th.
10000000 records sent, 671276.095858 records/sec (160.04 MB/sec), 9.01 ms avg latency, 227.00 ms max latency, 8 ms 50th, 13 ms 95th, 18 ms 99th, 28 ms 99.9th.

Total Throughput: 2191825.22 records/sec, 522.57 MB/sec, 9.34 ms avg latency, 225.33 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 3producer-topic-performancetest
```

As you can see from above, running multiple Producers in parallel I was able to push ~2.2M records/sec to my single `performancetest` topic. We could probably handle even more, which we will continue to test below

### Example total throughput from 5 Producers

Launch the marathon service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/5producer-topic-performancetest.json
```

Output from Logs:
```
10000000 records sent, 724427.702115 records/sec (172.72 MB/sec), 10.92 ms avg latency, 245.00 ms max latency, 9 ms 50th, 17 ms 95th, 31 ms 99th, 67 ms 99.9th.
10000000 records sent, 715205.263911 records/sec (170.52 MB/sec), 10.95 ms avg latency, 216.00 ms max latency, 8 ms 50th, 19 ms 95th, 72 ms 99th, 109 ms 99.9th.
10000000 records sent, 730940.720708 records/sec (174.27 MB/sec), 10.47 ms avg latency, 251.00 ms max latency, 10 ms 50th, 22 ms 95th, 106 ms 99th, 147 ms 99.9th.
10000000 records sent, 683060.109290 records/sec (162.85 MB/sec), 9.25 ms avg latency, 213.00 ms max latency, 8 ms 50th, 19 ms 95th, 84 ms 99th, 106 ms 99.9th.
10000000 records sent, 771485.881808 records/sec (183.94 MB/sec), 10.10 ms avg latency, 232.00 ms max latency, 9 ms 50th, 17 ms 95th, 29 ms 99th, 36 ms 99.9th.

Total Throughput: 3625119.68 records/sec, 864.3 MB/sec, 10.34 ms avg latency, 231.4 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 5producer-topic-performancetest
```

### Conclusions
As you can see from above, as we scale our Producers in parallel we can observe a linear relationship between adding more Producers and the Throughput increase. Now we will continue to scale our DC/OS cluster as well as our Kafka deployment to see if we can get even higher than 3.6 million records/sec with 5 brokers.

## Optional: Scale your Cluster Again to test 10/15 Producers as well as adding more Partitions

### DC/OS Cluster Prerequisites
- 1 Master
- 25 Private Agents
- DC/OS CLI Installed and authenticated to your Local Machine
- AWS Instance Type: m3.xlarge - 4vCPU, 15GB RAM See here for more recommended instance types by Confluent
	- EBS Backed Storage - 60 GB

### Test Setup:
- 6x Kafka Brokers
- 10/15 Producers
Total: 22 Nodes (We will scale to 9x brokers later)

### Example output from 10 Producers

Deploy the service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/10producer-topic-performancetest.json
```

Output from Logs:
```
10000000 records sent, 755115.910292 records/sec (180.03 MB/sec), 11.03 ms avg latency, 231.00 ms max latency, 10 ms 50th, 24 ms 95th, 37 ms 99th, 49 ms 99.9th.
10000000 records sent, 749512.816669 records/sec (178.70 MB/sec), 11.66 ms avg latency, 239.00 ms max latency, 10 ms 50th, 26 ms 95th, 45 ms 99th, 89 ms 99.9th.
10000000 records sent, 700182.047332 records/sec (166.94 MB/sec), 13.87 ms avg latency, 245.00 ms max latency, 11 ms 50th, 49 ms 95th, 120 ms 99th, 139 ms 99.9th.
10000000 records sent, 690989.496960 records/sec (164.74 MB/sec), 13.09 ms avg latency, 220.00 ms max latency, 10 ms 50th, 24 ms 95th, 43 ms 99th, 61 ms 99.9th.
10000000 records sent, 698470.349934 records/sec (166.53 MB/sec), 12.47 ms avg latency, 212.00 ms max latency, 10 ms 50th, 28 ms 95th, 92 ms 99th, 149 ms 99.9th.
10000000 records sent, 728226.041363 records/sec (173.62 MB/sec), 12.50 ms avg latency, 227.00 ms max latency, 10 ms 50th, 23 ms 95th, 38 ms 99th, 86 ms 99.9th.
10000000 records sent, 717257.208435 records/sec (171.01 MB/sec), 12.47 ms avg latency, 229.00 ms max latency, 10 ms 50th, 47 ms 95th, 80 ms 99th, 94 ms 99.9th.
10000000 records sent, 706713.780919 records/sec (168.49 MB/sec), 12.75 ms avg latency, 226.00 ms max latency, 11 ms 50th, 37 ms 95th, 56 ms 99th, 72 ms 99.9th.
10000000 records sent, 732278.851787 records/sec (174.59 MB/sec), 13.18 ms avg latency, 232.00 ms max latency, 11 ms 50th, 53 ms 95th, 96 ms 99th, 124 ms 99.9th.
10000000 records sent, 733675.715334 records/sec (174.92 MB/sec), 12.20 ms avg latency, 227.00 ms max latency, 10 ms 50th, 23 ms 95th, 42 ms 99th, 61 ms 99.9th.

Total Throughput: 7212422.22 records/sec, 1719.57 MB/sec, 12.52 ms avg latency, 228.8 ms avg max latency
```

Remove the Service:
```
dcos marathon app remove 10producer-topic-performancetest
```

### Example output from 15 Producers

Deploy the service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/15producer-topic-performancetest.json
```

Output from logs:
```
10000000 records sent, 738334.317779 records/sec (176.03 MB/sec), 58.79 ms avg latency, 341.00 ms max latency, 13 ms 50th, 142 ms 95th, 226 ms 99th, 264 ms 99.9th.
10000000 records sent, 720461.095101 records/sec (171.77 MB/sec), 59.65 ms avg latency, 316.00 ms max latency, 13 ms 50th, 114 ms 95th, 184 ms 99th, 211 ms 99.9th.
10000000 records sent, 714030.703320 records/sec (170.24 MB/sec), 59.98 ms avg latency, 374.00 ms max latency, 89 ms 50th, 222 ms 95th, 332 ms 99th, 365 ms 99.9th.
10000000 records sent, 720253.529242 records/sec (171.72 MB/sec), 58.68 ms avg latency, 337.00 ms max latency, 13 ms 50th, 111 ms 95th, 159 ms 99th, 175 ms 99.9th.
10000000 records sent, 707013.574661 records/sec (168.57 MB/sec), 59.36 ms avg latency, 381.00 ms max latency, 84 ms 50th, 212 ms 95th, 311 ms 99th, 359 ms 99.9th.
10000000 records sent, 703333.802223 records/sec (167.69 MB/sec), 58.73 ms avg latency, 358.00 ms max latency, 13 ms 50th, 105 ms 95th, 194 ms 99th, 258 ms 99.9th.
10000000 records sent, 704274.948940 records/sec (167.91 MB/sec), 60.77 ms avg latency, 363.00 ms max latency, 92 ms 50th, 225 ms 95th, 288 ms 99th, 346 ms 99.9th.
10000000 records sent, 706514.059630 records/sec (168.45 MB/sec), 60.43 ms avg latency, 370.00 ms max latency, 89 ms 50th, 223 ms 95th, 325 ms 99th, 361 ms 99.9th.
10000000 records sent, 699839.037021 records/sec (166.85 MB/sec), 64.88 ms avg latency, 335.00 ms max latency, 98 ms 50th, 238 ms 95th, 296 ms 99th, 316 ms 99.9th.
10000000 records sent, 702740.688686 records/sec (167.55 MB/sec), 60.56 ms avg latency, 330.00 ms max latency, 14 ms 50th, 121 ms 95th, 209 ms 99th, 252 ms 99.9th.
10000000 records sent, 687379.708551 records/sec (163.88 MB/sec), 58.40 ms avg latency, 336.00 ms max latency, 15 ms 50th, 149 ms 95th, 213 ms 99th, 259 ms 99.9th.
10000000 records sent, 663217.933413 records/sec (158.12 MB/sec), 57.41 ms avg latency, 343.00 ms max latency, 80 ms 50th, 225 ms 95th, 293 ms 99th, 329 ms 99.9th.
10000000 records sent, 664098.817904 records/sec (158.33 MB/sec), 57.51 ms avg latency, 300.00 ms max latency, 82 ms 50th, 228 ms 95th, 265 ms 99th, 290 ms 99.9th.
10000000 records sent, 730994.152047 records/sec (174.28 MB/sec), 27.65 ms avg latency, 270.00 ms max latency, 10 ms 50th, 28 ms 95th, 44 ms 99th, 65 ms 99.9th.
10000000 records sent, 693914.370967 records/sec (165.44 MB/sec), 20.09 ms avg latency, 294.00 ms max latency, 10 ms 50th, 190 ms 95th, 243 ms 99th, 270 ms 99.9th.

Total Throughput: 10556400.74 records/sec, 2516.83 MB/sec, 54.86 ms avg latency, 336.53 ms avg max latency
```

Remove Service:
```
dcos marathon app remove 15producer-topic-performancetest
```

### Initial Thoughts:
As you can see from above, our 6 broker node Kafka cluster is holding up a throughput of > 10M messages/sec. It is important to note though that there was an increase in latency here when we scaled to 15 Producers. We will first see if adding more partitions will help relieve the latency, as well as continue to scale the cluster to 9 broker nodes to see if we can continue to increase performance.

## Increasing Topic Partitions
As we increase the number of Kafka brokers in our cluster, we start to be able to tinker more with topic partitions. Partitions are a unit of parallelism in Kafka.

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
dcos confluent-kafka topic create performancetest2 --partitions 20 --replication 3
dcos confluent-kafka topic create performancetest3 --partitions 30 --replication 3
```

### Re-test 15 Producers on more partitions to see if Latency decreases

**15 Producers - 20 Partitions - Topic: performancetest2**

Deploy the Service:
```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/15producer-topic-performancetest2.json
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
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/15producer-topic-performancetest3.json
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
dcos confluent-kafka plan status deploy
```

Output should look similar to below:
```
$ dcos confluent-kafka plan status deploy
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
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/tests/25producer-topic-performancetest3.json
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
