---
title: Configure Elasticsearch for production environment
linkTitle: Production setup
weight: 50
date: 2022-06-09
description: Configure Elasticsearch for production environment.
---

This section covers advanced configuration topics that are required for a production environment. It is assumed that you have already familiarized yourself with the solution using the Basic setup.

Architecture examples
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
(**done**) Lifecycle Management (rename to Configure the retention period)

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