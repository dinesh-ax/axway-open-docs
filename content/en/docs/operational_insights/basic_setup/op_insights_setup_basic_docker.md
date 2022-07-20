---
title: Configure a basic setup for Docker compose
linkTitle: Basic setup for Docker compose
weight: 10
date: 2022-07-20
description: Configure a basic setup using Docker compose to test Elasticsearch in a single instance.
---

The following sections cover the configuration specific for using Docker Compose (virtual machines).

## Before you start

* Ensure that you have all [prerequisites](/docs/amplify_analytics/op_insights_prerequisites/) in place.
* Ensure that you have applied the [basic configuration](link) for general frameworks.

## Download and extract the release package

Download the relevant ELK package.

The community release always reflects the state of development. Check the [changelog](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/CHANGELOG.md) to ensure that you have select the correct version. For more information, see [Is this component officially supported by Axway](/docs/amplify_analytics/op_insights_faq/#is-this-component-officially-supported-by-axway).

We recommend that you run Operational Insights on different machines. Therefore, download and unpack the release package on each machine.

**Axway supported version**:

```bash
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.2.0/axway-apim-elk-v4.2.0.tar.gz -O - | tar -xvz
```

**Community version**:

```bash
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.5.0/axway-apim-elk-v4.5.0.tar.gz -O - | tar -xvz
```

To simplify updates, it is recommended to create a symlink folder and rename the provided file `env-sample` to `.env` in each machine as follows:

```bash
ln -s axway-apim-elk-v1.0.0 axway-apim-elk
cd axway-apim-elk-v1.0.0
cp env-sample .env
```

You must store the `.env` file as a central configuration file in a version management system.

## Setup Elasticsearch

Watch this video for a demonstration, [Setup Single Node Elasticsearch cluster](https://youtu.be/x-OdAdV2N7I).

If you are using an existing Elasticsearch cluster, you can skip this section and go straight to sections Logstash/Memcached and APIBuilder4Elastic.

Open the .env file and configure the ELASTICSEARCH_HOSTS. At this point, configure only one Elasticsearch node. You can start with a single node and add more nodes later.

This URL is used by all Elasticsearch clients (Logstash, API-Builder, Filebeat) of the solution to establish communication. If you use an external Elasticsearch cluster, specify the node(s) that are given to you. Note that the hostnames must be resolvable within the docker containers. Some parameters to consider to change before starting the cluster:

```bash
ELASTICSEARCH_HOSTS=https://my-elasticsearch-host.com:9200
ELASTICSEARCH_CLUSTERNAME=axway-apim-elasticsearch-prod
ES_JAVA_OPTS="-Xms8g -Xmx8g"
```

Run the following command to initialize a new Elasticsearch Cluster which is going through an appropriate bootstrapping. Later you can add more nodes to this single node cluster. Please do not use the init extension when restarting the node:

```bash
docker-compose --env-file .env -f elasticsearch/docker-compose.es01.yml -f elasticsearch/docker-compose.es01init.yml up -d
```

Wait until the cluster has started and then call the following URL:

```bash
curl -k GET https://my-elasticsearch-host.com:9200
{
  "name" : "elasticsearch1",
  "cluster_name" : "axway-apim-elasticsearch",
  "cluster_uuid" : "nCFt9WhpQr6JSOVY_h48gg",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "d34da0ea4a966c4e49417f2da2f244e3e97b4e6e",
    "build_date" : "2020-09-23T00:45:33.626720Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

At this point you can already add the cluster UUID to the .env (ELASTICSEARCH_CLUSTER_UUID) file. With that, the Single-Node Elasticsearch Cluster is up and running.

## Setup Kibana

Watch this video for a demonstration, [Setup Kibana](https://youtu.be/aLODAuXDMzY).

If you are using an existing Elasticsearch cluster, you can skip this section and go straight to sections Logstash/Memcached and APIBuilder4Elastic.

For Kibana all parameters are already stored in the .env file. Start Kibana with the following command:

```bash
docker-compose --env-file .env -f kibana/docker-compose.kibana.yml up -d
```

You can address Kibana at the following URL Currently no user login is required.

```
https://my-kibana-host:5601
```

If Kibana doesn't start (>3-4 minutes) or doesn't report to be ready, use docker logs Kibana to check for errors.

## Setup Logstash, API Builder, and Memcached

Watch this video for a demonstration, [Setup Logstash and API-Builder](https://youtu.be/lnSjF2tUS8Y).

It is recommended to deploy these components on one machine, so they are in a common Docker-Compose file and share the same network. Furthermore, a low latency between these components is beneficial. This allows you to use the default values for Memcached and API Builder. Therefore you only need to specify where the Admin-Node-Manager or the API manager can be found for this step. If necessary you have to specify an API-Manager admin user.

```bash
ADMIN_NODE_MANAGER=https://my-admin-node-manager:8090
API_MANAGER_USERNAME=elkAdmin
API_MANAGER_PASSWORD=elastic
```

If you are using an existing Elasticsearch cluster and have therefore skipped the previous two sections, please also configure the Elasticsearch hosts, any necessary users (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#activate-user-authentication>) and the Elasticsearch server (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#custom-certificates>) CA here.

```bash
ELASTICSEARCH_HOSTS=https://my-existing-elasticsearch-host1.com:9200, https://my-existing-elasticsearch-host2.com:9200, https://my-existing-elasticsearch-host3.com:9200
```

To start all three components the main Docker-Compose file is used:

```bash
docker-compose up -d
```

Check that the docker containers for Logstash, API Builder and Memached are running.

```bash
[ec2-user@ip-172-31-61-59 axway-apim-elk-v1.0.0]$ docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS                 PORTS                              NAMES
d1fcd2eeab4e        docker.elastic.co/logstash/logstash:7.12.2  "/usr/share/logstash…"   4 hours ago         Up 4 hours             0.0.0.0:5044->5044/tcp, 9600/tcp   logstash
4ce446cafda1        cwiechmann/apibuilder4elastic:v1.0.0        "docker-entrypoint.s…"   4 hours ago         Up 4 hours (healthy)   0.0.0.0:8443->8443/tcp             apibuilder4elastic
d672f2983c86        memcached:1.6.6-alpine                      "docker-entrypoint.s…"   4 hours ago         Up 4 hours             11211/tcp                          memcached
```

It may take some time (2-3 minutes) until Logstash is finally started.

```bash
docker logs logstash
Pipelines running {:count=>6, :running_pipelines=>[:".monitoring-logstash", :BeatsInput, :Events, :DomainAudit, :TraceMessages, :OpenTraffic], :non_running_pipelines=>[]}
Successfully started Logstash API endpoint {:port=>9600}
```

Notes:

* The Logstash API endpoint (9600) is not exposed outside of the docker container.
* Logstash is configured not to create indexes or index templates in Elasticsearch. These will be installed later by the API Builder application when the first events are received. The reason is that they may need to be created according to the region.

## Setup Filebeat

Watch this video for a demonstration, [Setup Filebeat](https://youtu.be/h0AdztZ2bSE).

Finally Filebeat must be configured and started. You can start Filebeat as Docker-Container using the Docker-Compose files and mount the corresponding directories into the container. Alternatively you can install Filebeat natively on the API gateway and configure it accordingly. It is important that the filebeat/filebeat.yml file is used as base. This file contains instructions which control the logstash pipelines.

The following instructions assume that you set up Filebeat based on filebeat/docker-compose.filebeat.yml.

This is an important step, as otherwise Filebeat will not see and send any Event data. Add the following configuration. At this point you can already configure the filebeat instances with a name/region if you like.

```bash
APIGATEWAY_LOGS_FOLDER=/opt/Axway/APIM/apigateway/logs/opentraffic
APIGATEWAY_TRACES_FOLDER=/opt/Axway/APIM/apigateway/groups/group-2/instance-1/trace
APIGATEWAY_EVENTS_FOLDER=/home/localuser/Axway-x.y.z/apigateway/events
APIGATEWAY_AUDITLOGS_FOLDER=/home/localuser/Axway-x.y.z/apigateway/logs
GATEWAY_NAME=API-Gateway 3
GATEWAY_REGION=US
```

Audit-Logs are optional. If you don't want them indexed just point to an invalid folder. You can find more information for each parameter in the env-sample.

To start Filebeat:

```bash
docker-compose --env-file .env -f filebeat/docker-compose.filebeat.yml up -d
```

Use docker logs filebeat to check that no error is displayed. If everything is ok, you should not see anything.
Check in Kibana (Menu --> Management --> Stack Management --> Index Management) that the indexes are filled with data.

If you encounter issues please see the Troubleshooting section.

## Next steps

After you have tried the solution in a single node elasticsearch scenario, ensure that you have read [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) then you can proceed to setup Operational insights in your production environment.
