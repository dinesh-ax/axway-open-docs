---
title: Size your infrastructure
linkTitle: Size your infrastructure
weight: 20
date: 2022-06-09
description: Size your infrastructure to be able to store and process millions of transactions per day and make them quickly available for traffic monitoring and analytics.
---

Operational Insights is designed to store and process millions of transactions per day and make them quickly available for traffic monitoring and analytics.

This advantage of being able to access millions of transactions is not free of charge with Elasticsearch, but is available in the size of the disc space provided. The solution has been extensively tested, especially for high-volume requirements. It processed 1010 transactions per second, up to 55 million transactions per day on the following infrastructure.

{{< alert title="Note">}}It is highly recommended that you size your infrastructure before configuring Operational Insights in your production environment.{{< /alert >}}

## Sizing recommendations

The following are important aspects for sizing your platform:

* **Transactions per second**: Transactions to be processed in real time.
* **Retention period**: This is reflected in the required disk space.

### Configure transactions per second

The number of concurrent transactions per second (TPS) that the entire platform must handle. The platform must therefore be scaled so that the events that occur on the basis of the transactions can be processed (ingested) in real time. It is important to consider the permanent load. As a general rule, more capacity should be planned to also quickly enable catch up operation after a downtime or maintenance.

The following table explains what a single component, such as Logstash or Filebeat can process in terms of TPS with `INFO` trace messages enabled to stay real time. The values in the table are the absolute maximum, which does not give any margin upwards for downtimes or maintenance of the platform. It is not possible to increase the capacity per component, so you must plan well for production.

The tests were performed on a number of **AWS EC2 instances**, using default parameters. To be able to reliably determine the limiting component, all other components were adequate sized and only the component under test was as stated in the table.

| Component               | Max. TPS           | Host-Machine | Config                | Comment |
| :---                    | :---               | :---         | :---                  | :---    |
| Filebeat                | >300               | t2.xlarge    | Standard              | Test was limited by the TPS the Mock-Service was able to handle. Filebeat can very likely handle much more volume.|
| Logstash                | 530                | t2.xlarge    | 6GB JVM-Head for Logstash  | Includes API-Builder & Memcache on the same machine running along with Logstash. Has processed ap. 3500 events per second. CPU is finally the limiting factor.  A production setup should have two Logstash nodes for high availability, which provides sufficient capacity for most requirements.|
| 2 Elasticsearch nodes   | 480                | t2.xlarge    | 8GB JVM-Heap for each node | Starting with a Two-Node cluster as this should be the mininum for a production setup. Kibana running on the first node.|
| 3 Elasticsearch nodes   | 740                | t2.xlarge    | 8GB JVM-Heap for each node | Data is searchable with a slight delay, but ingesting is not falling behind real-time in general up to the max. TPS.|
| 4 Elasticsearch nodes   | 1010                | t2.xlarge    | 8GB JVM-Heap for each node | |

Observe the following:

* Logstash, API Builder, Filebeat (for monitoring only), and Kibana are load balanced across all available Elasticsearch nodes. An external load balancer is not required as this is handled internally by each of the Elasticsearch clients.
* Do not size the Elasticsearch cluster node too large. The servers should not have more than 32GB memory because after that, the memory management kills the advantage again. In this case it is better to add another server. For more information, see [Results of infrastructure test](#results-of-infrastructure-test).

### Configure the retention period

The second important aspect for sizing is to configure the retention period, which defines how long data should be available. Accordingly, disk space must be made available.
In particular the Traffic-Summary and Traffic-Details indicies become huge and therefore play a particularly important role here. The solution is delivered with default values which you can read here. Based on the these default values which result in ap. 14 days the following disk space is required.

| Volume per day           | Total Disk-Space  | Comment |
| :---                     | :---              | :---    |
| up to 1 Mio  (~15 TPS)   | 100 GB            | 2 Elasticsearch nodes, each with 50 GB  |
| up to 5 Mio  (~60 TPS)   | 200 GB            | 2 Elasticsearch nodes, each with 100 GB |
| up to 10 Mio (~120 TPS)  | 250 GB            | 2 Elasticsearch nodes, each with 125 GB |
| up to 25 Mio (~300 TPS)  | 750 GB            | 3 Elasticsearch nodes, each with 250 GB |
| up to 50 Mio (~600 TPS)  | 1.5 TB            | 4 Elasticsearch nodes, each with 500 GB |

Tests were performed with log level `INFO`. If you run your API gateways with `DEBUG`, or have an unusually high number of log messages, more disk space might be necessary.

Note that when Elasticsearch is started by Docker Compose, its data is stored in an external volume. This is located at `/var/lib/docker` by default. Consequently, you must ensure that the available space is allocated there.

If the required storage space is unexpectedly higher, you proceed as following:

* Add an additional Elasticsearch cluster node at a later time.
    * Elasticsearch will then start balancing the cluster by moving shards to the new node, which will also improve the overall performance of the cluster.
* Increase the disk space of an existing node.
    * If the cluster state is green, you can stop a node, allocate more disk space, then start it again. The available disk space is used automatically by allocating shards.

For more information, see [Configure the retention period](/docs/amplify_analytics/op_insights_config_elastic_production#configure-the-retention-period)

### Results of infrastructure test

The following test infrastructure was used to determine the [maximum capacity or throughput](#transactions-per-second). The information is presented here so that you can derive your own sizing from it.

| Count | Node/Instance              |CPUS     | RAM   |Disc  | Component      | Version | Comment |
| :---: | :---                       | :---    | :---  | :--- | :---           | :---    | :---    |
| 6x    | AWS EC2 t2.xlarge instance | 4 vCPUS | 16GB  | 30GB | API-Management | 7.7-July| 6 API Gateways classical deployment, simulate traffic based on test-case |
| 4x    | AWS EC2 t2.xlarge instance | 4 vCPUS | 16GB  | 30GB | Logstash, API-Builder, Memcached | 7.10.0  | Logstash instances started as needed for the test. Logstash, API Builder, and Memcache always run together. |
| 5x    | AWS EC2 t2.xlarge instance | 4 vCPUS | 16GB  | 80GB | Elasticsearch  | 7.10.0  | Elasticsearch instances started as needed. Kibana running on the first node. |

There is no specific reason that EC2 t2.xlarge instances were used for the test setup. The deciding factor was simply the number of CPU cores and 16 GB RAM.

## Memory usage

To give you a good feel for the memory usage of the individual components, the following table shows the memory usage at around 330 transactions per second.

| Component      | Memory usage | Comment                                                                                                                  |
| :---           | :---         | :---                                                                                                                     |
| Elasticsearch  | 5.8 GB       | Configured to max. 8 GB, 5 Elasticsearch Hosts in total.                                                                  |
| Kibana         | 320 MB       | One Kibana instance running along with first Elasticsearch host.                                                       |
| Logstash       | 4.5 GB       | Configured to max. 6 GB, 4 Logstash processes running in total.                                                           |
| API-Builder    | 110-120 MB   | 4 API Builder docker containers running in total.                                                                         |
| Memcached      | 10-11 MB     | 4 Memcache instances. Memory finally depends on number of unique APIs, Apps, and so on, however, very unlikely more than 30 MB. |
| Filebeat       | 130 MB       | Filebeat is running as a Docker container alongside API Gateway. Filebeat itself is using approximately 30-35 MB.                  |

## Next steps

On the following sections, you are going to configure your API Gateway Manager to render log data, which is now provided by Elasticsearch instead of the individual API Gateway instances. For more information, see [Configure API Gateway Manager for Elasticsearch](/amplify_analytics/config_APIMng_elasticsearch).