---
title: Configure a production setup for Docker compose
linkTitle: Production setup for Docker Compose
weight: 10
date: 2022-07-20
description: Configure a production setup using Docker Compose to test Operational Insights in a highly available environment.
---

This section covers advanced configuration topics that are required for a production environment deployed with Docker compose.

## Before you start

* Ensure that you have all [prerequisites](/docs/amplify_analytics/op_insights_prerequisites/) in place.
* Ensure that you have applied the [basic configuration](link) for general frameworks.

<!--
- Configure High availability
- Elasticsearch cluster
- Logstash, API-Builder and Memcached
- Traffic-Payload
- Activate user authentication

(migrated) Traffic-Payload
(migrated) Setup Elasticsearch Multi-Node
(migrated) Setup API-Manager
(migrated) Setup local lookup
(migrated) Custom properties
(migrated) Activate user authentication
Enable Metricbeat
Configure cluster UUID
Custom certificates
Secure API-Builder Traffic-Monitor API
(migrated) Lifecycle Management (rename to Configure the retention period)
-->

## Configure traffic payload

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

The following sections explain how to setup Elasticsearch in a multi-node environment.

#### 1. Setup Cluster nodes

The **solution** is prepared for 5 nodes but can easily be extended to more nodes if needed. To configure multiple hosts, add the following to the `.envfile`:

````bash
ELASTICSEARCH_HOSTS=https://ip-172-31-61-143.ec2.internal:9200,https://ip-172-31-57-105.ec2.internal:9201
````

If you need special configuration please use the parameters `ELASTICSEARCH_PUBLISH_HOST<n>`, `ELASTICSEARCH_HOST<n>_HTTP` and `ELASTICSEARCH_HOST<n>_TRANSPORT`.

You may also change the cluster name if you prefer: `ELASTICSEARCH_CLUSTERNAME=axway-apim-elasticsearch`.

#### 2. Bootstrap the cluster

{{< alert title="Note" >}}
You can skip this step if you want to extend the cluster from the basic setup, since the basic setup has already initialized a cluster.
{{< /alert >}}

We recommend starting one node after the next. The first node will initially set up the cluster and bootstrap it. Start the first cluster node with the following statement:

```bash
docker-compose --env-file .env -f elasticsearch/docker-compose.es01.yml -f elasticsearch/docker-compose.es01init.yml up -d
```

This node automatically becomes the master node.

#### 3. Add additional nodes

You can add cluster nodes at any time to increase available disk space or CPU performance. To achieve resilience, it is strongly recommended set up at least 2 or even better 3 cluster nodes to also perform maintenance tasks such as updates. For more information see, [Designing for resilience](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html). It is also possible to have two Elasticsearch nodes running on the same machine.

To add a cluster node you need to configure ELASTICSEARCH_HOSTS and execute the following command:

```
# To add for instance a third node ELASTICSEARCH_HOSTS must contain three nodes
docker-compose --env-file .env -f elasticsearch/docker-compose.es03.yml up -d
```

Additionally please check in Kibana the new node has successfully joined the cluster and shards are assigned to it.

#### 4. Restart clients

Do you have changed the list of available Elasticsearch Nodes via the parameter: ELASTICSEARCH_HOSTS. For example from a single-node to a multi-node cluster, then it is strongly recommended to restart the corresponding clients (Kibana, Filebeat, Logstash, API-Builder). Via docker-compose, so that the containers are created with the new ELASTICSEARCH_HOSTS parameter. This ensures that clients can use the available Elasticsearch nodes for a fail-over in case of a node downtime.

## Configure API Manager

Before a document is send to Elasticsearch, additional information for the processed API is requested by Logstash from the API Manager through an API lookup. This lookup is handled by the API-Builder and performed against the configured API-Manager.
By default the configured Admin Node Manager host is also used for the API-Manager or the configured API-Manager URL:

```none
API_MANAGER=https://my.apimanager.com:8075
```

### Configure multiple API Managers

If you have several API Managers within your domain, you have to configure a mapping of which group (groupId) belongs to which API Manager. The group-id represents the Domain-Group and is attached to each Open-Traffic- or Metric-Event. It's used by Logstash to send the request to the API-Builder Lookup-API and is used there to perform the lookup against the belonging API-Manager.

The following syntax is used for this:

```
API_MANAGER=group-2|https://api-manager-1:8075, group-5|https://api-manager-2:8275
```

In this example, all events of `group`-2 are enriched with the help of the API manager (<https://api-manager-1:8075>) and of `group-5` accordingly with <https://api-manager-2:8275>.

### Configure different topologies and domains

**placeholder**: is this supposed to be a subsection of Configure API Manager?

From version 2.0.0 it is additionally possible to use the solution with different domains and topologies. An example are different hubs (for example, US, EMEA, APAC), each having their own Admin Node Manager and API-Manager, but still all API Events should be stored in a central Elasticsearch instance.

For this purpose the configurable `GATEWAY_REGION` in Filebeat is used. If this region is configured (e.g. US-DC1), all documents from this region are stored in separate indices, which nevertheless enable global analytics in the Kibana dashboards.

(**placeholder**: Image index_per_region.png)

Also in this case, the API Managers must or can be configured according to the Region & Group-ID of the event. Example:

```bash
API_MANAGER=https://my-apimanager-0:8075, group-1|https://my-api-manager-1:8175, group-5|https://my-api-manager-2:8275, group-6|US|https://my-api-manager-3:8375, group-6|eu|https://my-api-manager-4:8475
```

In this example, API-Managers are configured per Region & Group-ID. So if an event is processed which has a Region and Group-ID matching the configuration, then the configured API-Manager is used. This includes the lookup for the API details as well as the user lookup for the authorization.
If the region does not fit, a fallback is made to a group and last but not least to the generally stored API manager.
A configuration only per region is not possible!

When the API Builder is started, to validate the configuration, a login to each API-Manager is performed. Currently the same API manager user (API_MANAGER_USERNAME/API_MANAGER_PASSWORD) is used for each API Manager.

### Configure Admin Node Manager per region

**placeholder**: is this supposed to be a subsection of Configure API Manager or Configure different topologies and domains?

If you use the solution with multiple regions and different domains, all events/documents are stored in ONE Elasticsearch. Therefore you also need to tell the Admin-Node-Manager in each region, which data (indices) to use. If you don't do that, the Admin-Node-Manager will show the entire traffic from all regions which may not be desired but is also possible.
To do this, you need to store the appropriate region, which is also specified in the Filebeats for the API gateways, in the conf/envSettings.props file and restart the node manager. Example: REGION=US
This way the Admin-Node-Manager will only select data from these regional indexes. Learn more about the Admin-Node-Manager configuration (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/nodemanager>) (**placeholder**: shouldn't link to APIM docs which talks about the NM instead of creating a page for that?)

## Setup local lookup

**placeholder**: what does he means by lookup?

Starting with version 2.0.0 of the solution, it is optionally possible to use local configuration files for the API lookup in addition to the API Manager. This makes it possible to:

* Enrich native APIs
    * APIs that have been exposed natively through the API-Gateway policies not having a context
    * you can configure all information that would normally be retrieved from the API-Manager via this lookup file (Organization, API-Name, API-Method-Name, Custom-Properties)
    * for example display a /healthcheck as Healthcheck API in Kibana dashboards
* Ignore events
    * Additionall you can ignore OpenTraffc events so that they are not indexed or stored in Elasticsearch.
    * For example, the path: /favicon.ico, as this event does not add value.

Note that the local configuration file is used before the API-Manager lookup. If there is a match, no lookup to the API-Manager is performed.

To enable the local lookup, you must perform the following steps:

1. Add your config file. It is best to copy the delivered template: config/api-lookup-sample.json to your config/api-lookup.json.

    ```bash
    cp config/api-lookup-sample.json config/api-lookup.json
    ```

2. Activate the config file. In your .env file you must then enable the configuration file to be used by the API-Builder. To do this, configure or enable the following environment variable:

    ```bash
    API_BUILDER_LOCAL_API_LOOKUP_FILE=./config/api-lookup.json
    ```

3. Restart API-Builder:

```bash
docker stop apibuilder4elastic
docker-compose up -d
```

If an event is to be indexed, the API builder will try to read this file and will acknowledge this with the following error if the file cannot be found:

```bash
Error reading API-Lookup file: './config/api-lookup.json'
```

### Ignore events

At this point, we intentionally refer to events and not APIs, because different events (TransactionSummy, CircuitPath, TransactionElement) are created in the OpenTraffic log for each API call. Each is processed separately by Logstash and stored using an `upsert` in Elasticsearch with the same Correlation-ID. Most of these events contain the path of the called API, but unfortunately not all.
This is especially important when ignoring events so that they are not stored in Elasticsearch entirely. Since all events are processed individually, it must also be decided individually to ignore an event. Therefore, to ignore for example the Healthcheck API entirely, the following must be configured in the lookup file:

```bash
{
    "/healthcheck": {
        "ignore": true,
        "name": "Healthcheck API",
        "organizationName": "Native API"
    },
    "Policy: Health Check": {
        "ignore": true
    }
}
```

This will ignore events based on the path (TransactionSummary & TransactionElement) and the policy name (CircuitPath). You can see the result in the API-Builder log on level INFO with the following lines:

```log
Return API with apiPath: '/healthcheck', policyName: '' as to be ignored: true
```

or

```
Return API with apiPath: '', policyName: 'Health Check' as to be ignored: true
```

The information is cached in Logstash via memcached for 1 hour, so you will not see the loglines in the API-Builder for each request. You can of course force a reload of updated configuration via docker restart memcached.

It is additionally possible to overlay the local lookup file per group and region. This allows you, for example, to return different information per region for the same native API or to ignore an API only in a specific region. To do this, create files with the same base name and then a qualifier for the group and/or for the group plus region. Examples:

```
api-lookup.group-2.json
api-lookup.group-2.us.json
```

Again, it is not possible to specify only the region, but only in combination with the appropriate group.

## Custom properties

The solution supports configured API Manager API custom properties by default. This means that the custom properties are indexed within the field: `customProperties` in Elasticsearch and can be used for customer-specific evaluations.

Since version 4.3.0, it is also possible to index runtime attributes, i.e. policy attributes, in Elasticsearch and then analyze them in Kibana, for example.

The following steps are necessary:

### Export the custom attributes

In Policy Studio, configure which attributes should be exported to the transaction event log. Learn more (<https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_reference/log_global_settings/index.html#transaction-event-log-settings>) how to configure messages attributes to be stored in transaction events.

All exported attributes will be found in the Elasticsearch indices: `apigw-traffic-summary` and `apigw-hourly-traffic-summary` within the field `customMsgAtts`.

### Configure custom properties

In order to be able to use your attributes in aggregations for reports later on, you have to configure them for the solution in advance. By using the parameter: `EVENTLOG_CUSTOM_ATTR` you define which attributes should be indexed in Elasticsearch.
It is important that the fields have as high cardinality as possible to work efficiently in aggregations. The fields are indexed as a keyword in Elasticsearch. It is also possible to index fields as text, but these are not available in the long-term data and not recommended.

After restarting APIBuilder4Elastic, the solution will configure Elasticsearch (index templates & transform job) according to the parameter: `EVENTLOG_CUSTOM_ATTR`. Note that it takes 4 hours for the custom properties to be available in the transformed data (apigw-hourly-traffic-summary).

(**placeholder**) Watch this video that demonstrate how to ingest the HTTP user-agent into Elasticsearch: [Axway APIM with Elasticsearch - Use custom attributes](https://youtu.be/F0LCZhnWLkg).

## Activate user authentication

(**placeholder** introduction ??)

### Generate Built-In user passwords

This step can be ignored, when you are using an existing Elasticsearch cluster. Elasticsearch is initially configured with a number of built-in users, that don't have a password by default. So, the first step is to generate passwords for these users. It is assumed that the following command is executed on the first elasticsearch1 node:

```bash
docker exec elasticsearch1 /bin/bash -c "bin/elasticsearch-setup-passwords auto --batch --url https://localhost:9200"
```

As a result you will see the randomly generated passwords for the users: apm_system, kibana_system, kibana, logstash_system, beats_system, remote_monitoring_user and elastic. These passwords needs to be configured in the provided .env.

### Setup passwords

Please update the `.env` and setup all passwords as shown above. If you are using an existing Elasticsearch please use the passwords provided to you. The `.env` contains information about each password and for what it is used:

```bash
FILEBEAT_MONITORING_USERNAME=beats_system
FILEBEAT_MONITORING_PASSWORD=DyxUva2a6CwedZUhcpFH

KIBANA_USERNAME=kibana_system
KIBANA_PASSWORD=qJdDRYyL97lP5ERkHHrj

LOGSTASH_MONITORING_USERNAME=logstash_system
LOGSTASH_MONITORING_PASSWORD=Y6J6vgw9Z0RTPcFR8Qp3

LOGSTASH_USERNAME=elastic
LOGSTASH_PASSWORD=2x8vxZrvXX9a3KdGuA26

API_BUILDER_USERNAME=elastic
API_BUILDER_PASSWORD=2x8vxZrvXX9a3KdGuA26
```

### Disable anonymous user

In the `.env` file uncomment the following line:

```bash
ELASTICSEARCH_ANONYMOUS_ENABLED=false
```

After restart, Kibana will prompt to login before continue. Intially you may use the elastic user account to login and then create individual users and permissions.

### Restart services

After you have configured all passwords and configured security, please restart all services.

* Filebeat - It is now using the beats_system user to send monitoring information
* Logstash - Using the logstash_system to send monitoring data and logstash user to insert documents
* Kibana - Is using the kibana_system to send monitoring data
* API-Builder - Is using the API-Builder user to query Elasticsearch

It's very likely that you don't use the super-user `elastic` for `API_BUILDER_USERNAME`. It's recommended to create dedicated account. The monitoring users are used to send metric information to Elasticsearch to enable stack monitoring, which gives you insight about event processing of the complete platform.

(**placeholder**) Watch this video for a demonstration: [Authentication for Elasticsearch](https://youtu.be/4xxqV7PyBRQ).

## Custom certificates

a

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