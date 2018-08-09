# WIP - Advanced Load Testing Kafka
Lets take our prior example and expand on it. We're going to try to change up some parameters and see what performance we get

## Prerequisites
For this guide, the specs of my cluster are as stated below:
- DC/OS 1.11
- 1 Master
- 4 Private Agents
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

For our Advanced Guide we will later scale to a larger Kafka cluster size to observe performance improvements:

### Our Advanced Kafka Framework Parameters
- 3x Brokers
- 3 CPU
- 12GB MEM
- 25 GB Disk
- 512 MB JVM Heap Size

See `options.json` configuration below, as you can see there are many parameters in Kafka that we can tune:
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
dcos package install confluent-kafka --options=https://raw.githubusercontent.com/ably77/DCOS-Kafka-Performance/master/options.json --yes
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
  10.0.4.103   10.0.4.103  72244e5f-7a62-4058-b987-6b00244e9fce-S0  agent            aws/us-west-2  aws/us-west-2b
  10.0.4.202   10.0.4.202  72244e5f-7a62-4058-b987-6b00244e9fce-S1  agent            aws/us-west-2  aws/us-west-2b
  10.0.7.244   10.0.7.244  72244e5f-7a62-4058-b987-6b00244e9fce-S3  agent            aws/us-west-2  aws/us-west-2b
  10.0.7.87    10.0.7.87   72244e5f-7a62-4058-b987-6b00244e9fce-S2  agent            aws/us-west-2  aws/us-west-2b
master.mesos.  10.0.3.25     72244e5f-7a62-4058-b987-6b00244e9fce   master (leader)  aws/us-west-2  aws/us-west-2b
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
808317 records sent, 161663.4 records/sec (38.54 MB/sec), 4.0 ms avg latency, 251.0 max latency.
1423986 records sent, 284797.2 records/sec (67.90 MB/sec), 3.4 ms avg latency, 82.0 max latency.
1689771 records sent, 337954.2 records/sec (80.57 MB/sec), 2.8 ms avg latency, 21.0 max latency.
5000000 records sent, 270577.412198 records/sec (64.51 MB/sec), 3.18 ms avg latency, 251.00 ms max latency, 2 ms 50th, 6 ms 95th, 14 ms 99th, 21 ms 99.9th.
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
start.time - 2018-08-09 00:04:56:595
end.time - 2018-08-09 00:05:09:994
data.consumed.in.MB - 3576.2787
MB.sec - 266.9064
data.consumed.in.nMsg - 15000000
nMsg.sec - 1119486.5288
rebalance.time.ms - 3044
fetch.time.ms - 10355
fetch.MB.sec - 345.3673
fetch.nMsg.sec - 1448575.5674
```

- Topic: performancetest
- Number of Messages to Consume: 1.5M
- Threads: 5

Example Output (Edited for readability):
```
start.time - 2018-08-09 00:14:52:512
end.time - 2018-08-09 00:15:03:801
data.consumed.in.MB - 2151.4893
MB.sec - 190.5828
data.consumed.in.nMsg - 15000000
nMsg.sec - 1328727.0795
rebalance.time.ms - 3019
fetch.time.ms - 8270
fetch.MB.sec - 260.1559
fetch.nMsg.sec - 1813784.7642
```

## Understand baseline performance

My variable parameter was `record-size` in bytes which I averaged across 5 runs:

**Record Size: 250 bytes**:
```
5000000 records sent, 243510.446598 records/sec (58.06 MB/sec), 3.64 ms avg latency, 326.00 ms max latency, 2 ms 50th, 9 ms 95th, 19 ms 99th, 38 ms 99.9th.
5000000 records sent, 268125.268125 records/sec (63.93 MB/sec), 3.13 ms avg latency, 265.00 ms max latency, 2 ms 50th, 7 ms 95th, 18 ms 99th, 33 ms 99.9th.
5000000 records sent, 243831.073832 records/sec (58.13 MB/sec), 3.21 ms avg latency, 257.00 ms max latency, 3 ms 50th, 8 ms 95th, 18 ms 99th, 39 ms 99.9th.
5000000 records sent, 270577.412198 records/sec (64.51 MB/sec), 3.18 ms avg latency, 251.00 ms max latency, 2 ms 50th, 6 ms 95th, 14 ms 99th, 21 ms 99.9th.
5000000 records sent, 238435.860753 records/sec (56.85 MB/sec), 3.21 ms avg latency, 266.00 ms max latency, 2 ms 50th, 7 ms 95th, 18 ms 99th, 36 ms 99.9th.

Average: 252,896.1 records/sec, 60.3 MB/sec, 3.27 ms avg latency, 272.8 ms max latency
```

**Record Size: 500 bytes**:
```
5000000 records sent, 250953.623770 records/sec (119.66 MB/sec), 3.96 ms avg latency, 268.00 ms max latency, 3 ms 50th, 13 ms 95th, 33 ms 99th, 62 ms 99.9th.
5000000 records sent, 257479.787837 records/sec (122.78 MB/sec), 3.84 ms avg latency, 254.00 ms max latency, 3 ms 50th, 10 ms 95th, 22 ms 99th, 35 ms 99.9th.
5000000 records sent, 244774.073530 records/sec (116.72 MB/sec), 3.78 ms avg latency, 270.00 ms max latency, 3 ms 50th, 10 ms 95th, 21 ms 99th, 39 ms 99.9th.
5000000 records sent, 245591.630237 records/sec (117.11 MB/sec), 3.69 ms avg latency, 268.00 ms max latency, 3 ms 50th, 10 ms 95th, 19 ms 99th, 30 ms 99.9th.
5000000 records sent, 251357.329580 records/sec (119.86 MB/sec), 3.53 ms avg latency, 253.00 ms max latency, 3 ms 50th, 9 ms 95th, 20 ms 99th, 42 ms 99.9th.

Average: 250,031.3 records/sec, 119.23 MB/sec, 3.76 ms avg latency, 262.6 ms max latency
```

**Record Size: 1 kB**
```
5000000 records sent, 235938.089845 records/sec (225.01 MB/sec), 5.01 ms avg latency, 261.00 ms max latency, 3 ms 50th, 18 ms 95th, 61 ms 99th, 97 ms 99.9th.
5000000 records sent, 231932.461267 records/sec (221.19 MB/sec), 5.62 ms avg latency, 265.00 ms max latency, 3 ms 50th, 22 ms 95th, 133 ms 99th, 160 ms 99.9th.
5000000 records sent, 233404.910839 records/sec (222.59 MB/sec), 4.53 ms avg latency, 259.00 ms max latency, 3 ms 50th, 12 ms 95th, 30 ms 99th, 80 ms 99.9th.
5000000 records sent, 238061.229348 records/sec (227.03 MB/sec), 4.61 ms avg latency, 255.00 ms max latency, 3 ms 50th, 15 ms 95th, 40 ms 99th, 59 ms 99.9th.
5000000 records sent, 235194.505856 records/sec (224.30 MB/sec), 4.59 ms avg latency, 270.00 ms max latency, 3 ms 50th, 18 ms 95th, 49 ms 99th, 88 ms 99.9th.

Average: 234,906.2 records/sec, 224.02 MB/sec, 4.87 ms avg latency, 262 ms max latency
```

## Goal: Increase Throughput

For increasing throughput of Producers, Confluent recommends:
- batch.size: increase to 100000-200000 (default 16384)
- linger.ms: increase to 10-100 (default 0)
- compression.type = lz4 (default none)
- acks = 1 (default 1)
- buffer.memory: increase if there are a lot of partitions (default 33554432)

For increasing throughput of Consumers, Confluent recommends:
- fetch.min.bytes: increase to ~1000000 (default 1)


### Lets try the minimum ranges on 10M records:
- batch.size - 100000
- linger.ms - 10
- compression.type - lz4
- acks - 1
- buffer.memory - default

Command:
```
kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=lz4 batch.size=100000 linger.ms=10 retries=0 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Output:
```
10000000 records sent, 346188.465000 records/sec (82.54 MB/sec), 7.75 ms avg latency, 243.00 ms max latency, 8 ms 50th, 13 ms 95th, 23 ms 99th, 44 ms 99.9th.
```

### Lets try the maximum range parameters on 10M records:
- batch.size - 200000
- linger.ms - 100
- compression.type - lz4
- acks - 1
- buffer.memory - default

Command:
```
kafka-producer-perf-test --topic performancetest --num-records 10000000 --record-size 250 --throughput 1000000 --producer-props acks=1 buffer.memory=67108864 compression.type=lz4 batch.size=200000 linger.ms=100 retries=0 bootstrap.servers=kafka-0-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-1-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025,kafka-2-broker.confluent-kafka.autoip.dcos.thisdcos.directory:1025
```

Output:
```
10000000 records sent, 349968.502835 records/sec (83.44 MB/sec), 61.24 ms avg latency, 372.00 ms max latency, 61 ms 50th, 105 ms 95th, 110 ms 99th, 122 ms 99.9th.
```

### Conclusions
Both lower and upper range adjustments result in a >35% increase in throughput performance


## Goal: Optimize for Latency
For optimizing latency of Producers, Confluent recommends:
- linger.ms - 0
- compression.type - none
- acks - 1

For optimizing latency of Brokers, Confluent recommends:
- num.replica.fetchers - increase if followers can't keep up with the leader (default = 1)

For optimizing latency of Consumers, Confluent recommends:
- fetch.min.bytes - 1 (default 1)

## Goal: Optimize for Durability
For optimizing durability of Procuers, Confluent recommends:
- replication.factor - 3, configure per topic
- acks - all
- retries - 1
- max.in.flight.requests.per.connection - 1 (default 5)
	- to prevent out of order messages

### Continue on
