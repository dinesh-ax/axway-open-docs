---
title: Configure a production setup for Docker compose
linkTitle: Production setup for Docker compose
weight: 10
date: 2022-07-20
description: Configure a production setup using Docker compose to test Operational Insights in a single instance.
---

This section covers advanced configuration topics that are required for a production environment deployed with Docker compose. Follow the next sections to configure Operational Insights in your Docker compose production environment.

## Before you start

* Ensure that you have all [prerequisites](/docs/amplify_analytics/op_insights_prerequisites/) in place.
* Ensure that you have applied the [basic configuration](link) for general frameworks.

<!--
(conceptual topic. Should be around the overview of this bundle) Architecture examples 
Traffic-Payload
Setup Elasticsearch Multi-Node
Setup API-Manager
Setup local lookup
Custom properties
Activate user authentication
Enable Metricbeat
Configure cluster UUID
Custom certificates
Secure API-Builder Traffic-Monitor API
(added here) Lifecycle Management (rename to Configure the retention period)
-->

## Traffic Payload

The payload belonging to an API request is not written directly to the open traffic event log and therefore not stored in Elasticsearch.

To clarify what is meant by payload at this point the following example screenshot.

<!-- I replaced the screenshot with the actual text -->

```bash
HTTP/1.1 200 OK
Date:Wed, 13 Jan 2021 23:08:58GMT
CSRF-Token:62325BA818F8203917CB61AE883346D7F7A206E564E26008CAC3ED37386B1B7B
Content-type:application/json
Cache-Control:no-cache,no-store,must-revalidate
Pragma:no-cache
Expires:0
X-Frame-Options:DENY
X-Content-Type-Options:nosniff
X-XSS-Protection:0
Server:Gateway
Connection:close
X-CorrelationID:Id-8a7dff5f6612be6b4aa9d851 0

{"id":"1eb19f0c-810a-4ab1-94c6-bf85833754a8","organizationId":"2b4a2c5a-827c-4be7-8dc9-ffbd5f086144,ou=organizations,ou=APIPortal","lastSeen":1610579331159,"changePassword":false}
```

This payload, if not configured as explained below, will only be displayed as long as it is in the OBSDB. After that, NO DATA is displayed instead of the payload.

Configure the payload as follows:

### Export the payload from API Gateway

In order to also make the payload available in the Traffic Monitor via the solution, this must also be exported from the API gateway to the runtime.
To do this, go to the Server Settings Open Traffic Event Log configuration (**placeholder** - where's it, in PS?) and enable the payload export:

(**placeholder** - instead of adding an image, complete the sentence "and enable the payload export by ??? configuring something? )

You need to repeat this step for each API gateway group for which you want to make the payload available. You can change the export path if necessary, for example to write the payload to an NFS volume.

### Make the payload available

The saved payload must be made available to the API-Builder Docker container as a mount under `/var/log/payloads`. You can find an example in the docker-compose.yml:
`${APIGATEWAY_PAYLOADS_FOLDER}:/var/log/payloads` shared volume into the API Builder container.

### Make the payload available per region

If you are using the region feature (**placeholder**, see Setup API-Manager > Different Topologies/Domains), that is, collecting API Gateways of different Admin Node Manager domains into a central Elasticsearch instance, then you also need to make the payload available to the API builder regionally separated. For example, if you have defined the region like, `REGION=US-DC1`, all traffic payload from these API Gateways must be made available to the API Builder as follows:

```bash
/var/log/payloads/us-dc1/<YYY-MM-DD>/<HH.MI>/<payloadfile>
```

So you need to make the existing structure available in a regional folder. For this, the region must be in lower case. And you have to configure each Admin-Node-Manager with the correct region (**placeholder**, see Setup API-Manager > Different Topologies/Domains).

Note:

* Payload handling is enabled by default. So it is assumed that you provide the payload to the API Builder container. Set the parameter `PAYLOAD_HANDLING_ENABLED=false` if you do not need this.
* Payload shown in the Traffic-Monitor UI is limited to 20 KB by default. If required the payload can be downloaded completely using the Save option.

## Setup Elasticsearch multi-node

For a production environment Elasticsearch must run a multi-node Elasticsearch cluster environment. Indices are configured so that available nodes are automatically used for primary and replica shards. If you use only one Elasticsearch node, the replica shards cannot be assigned to any node, which causes the cluster to remain in the Yellow state. This in turn leads to tasks not being performed by Elasticsearch. For example, lifecycle management of the indexes.
If you are using an external Elasticsearch cluster, you can skip most of the following instructions, besides step number 1 to configure your available Elasticsearch cluster nodes.

The setup of a Multi-Node Elasticsearch Cluster can be done with default settings via the parameter `ELASTICSEARCH_HOSTS`. Some hints for this: (**placeholder**: for this what?)

* The default ports are 9200, 9201, 9202 and 9300, 9301, 9302 for the Elasticsearch instances elasticsearch1, elasticsearch2 and elasticsearch3.
    * these ports are exposed by Docker-Compose through the docker containers
    * please configure ELASTICSEARCH_HOSTS accordingly with 9200, 9201, 9202
    * based on the HTTP ports, the transport port is derived. (9201 --> 9301, ...)
    * other ports are possible and can be configured
* Based on the specified URLs, the necessary Elasticsearch parameters are set when creating or starting the Elasticsearch Docker containers.
* Multiple Elasticsearch Nodes are only really useful if they actually run on different hosts.
    * Again, it is assumed that the release package is downloaded on the individual hosts and the .env file is provided.
* You can always add more nodes to the Elasticsearch cluster to provide additional disk space and computing power.
    * You can start with two nodes today and add another cluster node in 6 months if needed.
* Note that all clients (Filebeat, Logstash, API-Builder and Kibana) are using the given Elasticsearch hosts
    * that means, you don't need a loadbalancer in front of the Elasticsearch-Cluster to achieve high availability at this point
    * make sure, that all clients are configured and restarted with the available Elasticsearch hosts

Watch [this video](https://youtu.be/sM5-0c8aEZk) for a demonstration of how to add an elasticsearch node (**placeholder**: add the node where?)

### placeholder - need a title for this subsection

Need an introduction for this subsection.

* Setup Cluster-Nodes

The solution is prepared for 5 nodes but can easily be extended to more nodes if needed. To configure multiple hosts in the .envfile:

````bash
ELASTICSEARCH_HOSTS=https://ip-172-31-61-143.ec2.internal:9200,https://ip-172-31-57-105.ec2.internal:9201
````

If you need special configuration please use the parameters `ELASTICSEARCH_PUBLISH_HOST<n>`, `ELASTICSEARCH_HOST<n>_HTTP` and `ELASTICSEARCH_HOST<n>_TRANSPORT`.

You may also change the cluster name if you prefer: `ELASTICSEARCH_CLUSTERNAME=axway-apim-elasticsearch`.

* Bootstrap the cluster

placeholder

## placeholder

placeholder

## Configure the retention period

Since new data is continuously stored in Elasticsearch in various indexes, these must be removed after a certain period of time.

Since version 2.0.0, the solution uses the Elasticsearch [index lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) (ILM) feature for this purpose, which defines different lifecycle stages per index. The so-called ILM policies are automatically configured by the solution with default values using [configuration files](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/apibuilder4elastic/elasticsearch_config) and can be reviewed in Kibana. Beginning with version 4.1.0, you can also configure the lifecycle of the data yourself according to your requirements. The indices pass through stages such as Hot, Warm, Cold which can be used to deploy different performance hardware per stage. This means that traffic details from two weeks ago no longer have to be stored on high-performance machines.

The configuration is defined here per data type (e.g. Summary, Details, Audit, ...). The following table gives an overview about the default values. The number of days that is crucial for the retention period is the delete days. This gives the guaranteed number of days that the data is guaranteed to be available. More information on how the lifecycle works can be found later in this section. You can use the further phase, for example, to allocate more favorable resources accordingly.

**TABLE**:

As of version 4.1.0, you can configure how long the indexed data should be kept in Elasticsearch. Before starting, you should read and understand the following information thoroughly, because once deleted, data cannot be recovered.
Individual API transactions are stored as documents in Elasticsearch Indices. However, it is not the case that individual documents are ultimately deleted again, instead it is always an entire index with millions of transactions/documents. Therefore, you can only control the retention period for an entire index, not per document.
When API transactions are stored in an index, the size of the index increases accordingly. To prevent an index from growing infinitely, it can be rolled over after a certain time. A new active index is created, which is used to write the data. This replaces the old index, which is only used for reading. This process is called [rollover](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-rollover.html).

To avoid having to control this process manually, you can use ILM policies in Elasticsearch. These ILM policies perform the rollover based on defined rules and then send the index through further phases for various purposes.

These ILM policies are configured automatically by the solution with default values and are stored and managed for each index in Elasticsearch. The default values result in the data being available for at least 2 weeks.

If you would like to customize the lifecycle, then you can provide a corresponding configuration file from version 4.1.0 and use the parameter: `RETENTION_PERIOD_CONFIG`. This is used to adapt the ILM policies accordingly.

Here is an example:

```json
{
    "retentionPeriods": {
        "apigw-traffic-summary": {
            "rollover": {
                "max_age": "7d",
                "max_size": "15gb"
            }, 
            "retentionPeriod": "7d"
        }, 
        "apigw-traffic-details": {
            "rollover": {
                "max_age": "7d",
                "max_size": "15gb"
            }, 
            "retentionPeriod": "6d"
        }, 
        "apigw-traffic-trace": {
            "rollover": {
                "max_age": "7d",
                "max_primary_shard_size": "15gb"
            }, 
            "retentionPeriod": "5d"
        }
    }
}
```

The configuration is defined per index and is divided into two areas. When should the rollover happen and how many days after the rollover should the data still be available.
The following figure illustrates the process:

**IMAGE**:

The following steps show how to configure the retention period.

### 1. Create your retention period configuration file

Create a new file for your retention period configuration. For example,`config/custom-retention-period.json`. As a template, you can use the file, `config/my-retention-period-sample.json`.

### 2. Define the rollover period

It is important to understand that the time period until the rollover of an index is not exactly fixed.
For example, if you specify a maximum age and size for an index, then the index will be rolled over as soon as a condition is met.

* If the maximum size is too small for your transaction volume, then an index can meet the size condition in less than 24 hours and will be rolled over.
* If the maximum size is too large, the index will be rolled when it reaches the maximum age (e.g. after 7 days).

So how long the data is available from the very beginning to the end of an index is the sum of the period from the index's initial creation to the rollover plus the period until the delete. As the rollover date cannot be defined exactly, you need to monitor your system accordingly and adjust the lifecycle accordingly to get the desired retention time.

You can use the following conditions for the rollover:

* **max_age**: Defines the maximum age of an index until it is rolled over
* **max_size**: The maximum index size. As an index has a Primary and Replica the required disk space is doubled (max_size: 30gb turns it 60gb disk space used)
* **max_primary_shard_size**: Starting with an Elasticsearch version 7.13, you can also define the maximum shard size of an index. All indexes , except apigw-management-kpis and apigw-domainaudit, have 5 shards. So you have to multiply the specified size by 5.

For more information, see [ILM Rollover options](https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-rollover.html#ilm-rollover-options).

### 3. Define the retention period

With the parameter: retentionPeriod you define the time period for which the data is guaranteed to be available. As already described, the time until the rollover of the index adds to this. You can specify only days here.

### 4. Apply the configuration

The last step is to reference your configuration file in your .env file with the parameter: RETENTION_PERIOD_CONFIG=./config/custom-retention-period.json, then restart API Builder as follows:

```bash
docker-compose stop apibuilder4elastic docker-compose up
```

You can check in Kibana whether the ILM policy has been adjusted accordingly. To do this, go to Stack Management --> Index Lifecycle Policies - Open the corresponding policy here and check the phase.

### Further notes

* Changes to the ILM-Policy have no influence on indices that have already been rolled over, as these have already entered lifecycle management
* Indexes should not be too small, as this increases the load on Elasticsearch too much.
    * For each active index there are 5 Primary- and 5 Replica-Shards.
    * Each shard corresponds to a Lucene instance, which consumes corresponding resources.
    * The smaller an index, the more indexes, the more shards, the more resources are needed.
    * Elastic's recommendation is 30GB. The solution does not allow index size below 5GB.
* It's optional to use different hardware per stage

## Next steps

On the following section, you are going to ...