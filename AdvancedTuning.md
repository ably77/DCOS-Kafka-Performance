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
943363 records sent, 188672.6 records/sec (44.98 MB/sec), 5.8 ms avg latency, 278.0 max latency.
1330321 records sent, 266064.2 records/sec (63.43 MB/sec), 3.4 ms avg latency, 45.0 max latency.
1521870 records sent, 304374.0 records/sec (72.57 MB/sec), 3.1 ms avg latency, 31.0 max latency.
5000000 records sent, 260783.393314 records/sec (62.18 MB/sec), 3.79 ms avg latency, 278.00 ms max latency, 3 ms 50th, 9 ms 95th, 21 ms 99th, 60 ms 99.9th.
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
start.time - 2018-08-09 19:07:31:979
end.time - 2018-08-09 19:07:45:777
data.consumed.in.MB - 3576.2787
MB.sec - 259.1882
data.consumed.in.nMsg - 15000000
nMsg.sec - 1087114.0745
rebalance.time.ms - 3047
fetch.time.ms - 10751
fetch.MB.sec - 332.6461
fetch.nMsg.sec - 1395219.0494
```

## Understand baseline performance

My variable parameter was `record-size` in bytes which I averaged across 5 runs:

**Record Size: 250 bytes**:
```
5000000 records sent, 260783.393314 records/sec (62.18 MB/sec), 3.79 ms avg latency, 278.00 ms max latency, 3 ms 50th, 9 ms 95th, 21 ms 99th, 60 ms 99.9th.
5000000 records sent, 266937.162992 records/sec (63.64 MB/sec), 3.73 ms avg latency, 284.00 ms max latency, 3 ms 50th, 10 ms 95th, 18 ms 99th, 25 ms 99.9th.
5000000 records sent, 257984.624116 records/sec (61.51 MB/sec), 3.63 ms avg latency, 261.00 ms max latency, 3 ms 50th, 9 ms 95th, 30 ms 99th, 72 ms 99.9th.
5000000 records sent, 241091.663050 records/sec (57.48 MB/sec), 3.63 ms avg latency, 250.00 ms max latency, 3 ms 50th, 9 ms 95th, 18 ms 99th, 26 ms 99.9th.
5000000 records sent, 254065.040650 records/sec (60.57 MB/sec), 3.57 ms avg latency, 260.00 ms max latency, 3 ms 50th, 8 ms 95th, 18 ms 99th, 27 ms 99.9th.

Average: 256172.38 records/sec, 61.07 MB/sec, 3.67 ms avg latency, 266.6 ms max latency
```

**Record Size: 500 bytes**:
```
5000000 records sent, 236787.270316 records/sec (112.91 MB/sec), 4.47 ms avg latency, 279.00 ms max latency, 3 ms 50th, 15 ms 95th, 43 ms 99th, 72 ms 99.9th.
5000000 records sent, 232277.246121 records/sec (110.76 MB/sec), 4.46 ms avg latency, 264.00 ms max latency, 3 ms 50th, 15 ms 95th, 44 ms 99th, 67 ms 99.9th.
5000000 records sent, 238481.350758 records/sec (113.72 MB/sec), 3.85 ms avg latency, 285.00 ms max latency, 3 ms 50th, 10 ms 95th, 20 ms 99th, 29 ms 99.9th.
5000000 records sent, 246657.786986 records/sec (117.62 MB/sec), 4.09 ms avg latency, 282.00 ms max latency, 3 ms 50th, 11 ms 95th, 23 ms 99th, 38 ms 99.9th.
5000000 records sent, 242812.742813 records/sec (115.78 MB/sec), 4.82 ms avg latency, 272.00 ms max latency, 3 ms 50th, 16 ms 95th, 60 ms 99th, 192 ms 99.9th.

Average: 239403.28 records/sec, 114.16 MB/sec, 4.34 ms avg latency, 276.4 ms max latency
```

**Record Size: 1 kB**
```
5000000 records sent, 222736.992160 records/sec (212.42 MB/sec), 5.08 ms avg latency, 265.00 ms max latency, 3 ms 50th, 15 ms 95th, 32 ms 99th, 46 ms 99.9th.
5000000 records sent, 208064.583247 records/sec (198.43 MB/sec), 5.24 ms avg latency, 255.00 ms max latency, 4 ms 50th, 18 ms 95th, 37 ms 99th, 56 ms 99.9th.
5000000 records sent, 222084.036599 records/sec (211.80 MB/sec), 5.46 ms avg latency, 273.00 ms max latency, 4 ms 50th, 19 ms 95th, 35 ms 99th, 53 ms 99.9th.
5000000 records sent, 211220.006759 records/sec (201.44 MB/sec), 6.11 ms avg latency, 273.00 ms max latency, 3 ms 50th, 17 ms 95th, 41 ms 99th, 66 ms 99.9th.
5000000 records sent, 229263.148242 records/sec (218.64 MB/sec), 5.54 ms avg latency, 269.00 ms max latency, 4 ms 50th, 18 ms 95th, 45 ms 99th, 69 ms 99.9th.

Average: 218673.75 records/sec, 208.55 MB/sec, 5.49 ms avg latency, 267 ms max latency
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


### Lets try the lower end range parameters of the recommendations above:
- number of records - 10M
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
10000000 records sent, 328331.746397 records/sec (78.28 MB/sec), 8.00 ms avg latency, 256.00 ms max latency, 8 ms 50th, 13 ms 95th, 18 ms 99th, 27 ms 99.9th.
```

### Lets try the upper end range parameters of the recommendations above:
- number of records - 10M
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
10000000 records sent, 346404.323126 records/sec (82.59 MB/sec), 62.27 ms avg latency, 352.00 ms max latency, 58 ms 50th, 102 ms 95th, 107 ms 99th, 118 ms 99.9th.
```

### Conclusions
Both lower and upper range adjustments result in a >30% increase in throughput performance from tuning for throughput.


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
For optimizing durability of Producers, Confluent recommends:
- replication.factor - 3, configure per topic
- acks - all
- retries - 1
- max.in.flight.requests.per.connection - 1 (default 5)
	- to prevent out of order messages

For optimizing durability of Brokers, Confluent recommends:
- default.replication.factor - 3 (default 1)
- auto.create.topics.enable - false (default true)
- min.insync.replicas - 2 (default 1)
- unclean.leader.election.enable - false (default true) 
- broker.rack - rack of the broker (default null)
- log.flush.interval.messages / log.flush.interval.ms - for topics with very low throughput, set message interval or time interval low as needed (default allows the OS to control flushing)

For optimizing durability of Consumers, Confluent recommends:
- auto.commit.enable - false (default true)

## Goal: Optimize for Availability
For optimizing availability of Brokers, Confluent recommends:
- unclean.leader.election.enable - true (default true)
- min.insync.replicas - 1 (default 1)
- num.recovery.threads.per.data.dir - number of directories in log.dirs (default 1)

For optimizing availability of Consumers, Confluent recommends:
- session.timeout.ms - as low as feasible (default 10000)
