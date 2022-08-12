---
title: Enable monitoring and metrics
linkTitle: Enable monitoring and metrics
weight: 50
date: 2022-08-05
description: Enable monitoring and metrics in Operational Insights deployed either in a Docker Compose environment or in Helm charts.
---

It is important that the Operational Insights is monitored appropriately and by default Internal Stack Monitoring is used for this purpose, which monitors the Elasticsearch cluster, Kibana, Logstash and Filebeat. You can alternatively use Metricbeat and you can find more guidance in this section on platform monitoring.

## Enable Metricbeat

In the default configuration, Operational Insights uses the so-called self-monitoring. This means that components such as Logstash, Kibana, Filebeat, and son on independently send monitoring information (metrics) to Elasticsearch. However, this approach is not recommended by Elastic and is deprecated. Metricbeat should be used instead.

Because Operational Insights cannot easily be delivered with a pre-configured Metricbeat, because this depends too much on the deployment, you must set some parameters in the `.env` file and then start Operational Insights.

This section covers how to enable Metricbeat and monitor Memcache, the running Docker containers in addition to the pure Elastic stack.

### 1. Activate Metricbeat

The first step to enable Metricbeat is to activate it.

* Check that the `METRICBEAT_USERNAME` and `METRICBEAT_PASSWORD` user is set up correctly. This user must have rights to Kibana (to upload dashboards) and Elasticsearch (to create indexes).
* Set the parameter `METRICBEAT_ENABLED=true`. This will be populated when the Metricbeat container starts.
* Set the parameter `SELF_MONITORING_ENABLED=false` to disable legacy monitoring.

### 2. Configure Metricbeat

Use the parameter METRICBEAT_MODULES, which must be set differently for each host, depending on which services are running on which host.

| Component              | Description                           |
| :---                   | :---                                  |
| **elasticsearch**      | Replaces internal monitoring so that Elasticsearch metrics appear in Kibana Stack-Monitoring. **Important**: ONE metricbeat monitors ALL Elasticsearch nodes based on the parameter: ELASTICSEARCH_HOSTS and Kibana if needed.                                   |
| **kibana**             | Enables monitoring of Kibana analogous to Elasticsearch. You can use a Metricbeat for monitoring Elasticsearch & Kibana as explained. |
| **logstash**           | Monitoring Logstash. Provides the data in Kibana Stack-Monitoring. |
| **filebeat**           | Monitoring Filebeat. Provides the data in Kibana stack-Monitoring. |
| **memcached**          | Captures statistics from Memcached. Is indexed, but currently not used in any dashboard and can be disabled if not needed. |
| **system**             | Provides system metrics such as disk IO, network IO, etc. **Important** In the default configuration out of the Docker container which limits the process view. Will be improved in a later release. See example: [System overview](imgs/metricbeat-system-overview.png) or [Host overview](imgs/metricbeat-api-host-overview.png) |
| **docker**             | Provides information about running Docker containers that can be displayed in dashboards. Docker containers are automatically detected on the host. See example: [Docker overview](imgs/containers-overview.png) |

Set the parameter `METRICBEAT_NODE_NAME` to a descriptive name, which should be displayed later in the Kibana dashboards for the host Metricbeat is running on.

### 3. Start Metricbeat

Start the Metricbeat container on each host:

```bash
docker-compose --env-file .env -f metricbeat/docker-compose.metricbeat.yml up -d
```

On each host a container `metricbeat` started with given configuration in the belonging .env file.

### 4. Disable self-monitoring

Ensure the parameter `SELF_MONITORING_ENABLED=false` is set. Then stop Elasticsearch, Kibana, Logstash, and Filebeat services and restart them with docker-compose that they are no longer using self-monitoring. A docker restart is not sufficient here. For example:

```bash
docker stop kibana
docker-compose --env-file .env -f kibana/docker-compose.kibana.yml up -d
```

{{< alert title="Note" color="primary" >}}
Before restarting an Elasticsearch Node, ensure the cluster state is green to stay online during the restart.
{{< /alert >}}

## Enable Elastic Application Performance Monitoring

You can enable Application Performance Monitoring (APM) to monitor APIBuilder4Elastic and other services (for example, API Builder Services).

Image: <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/apm/2_apm-apibuilder4elastic-overview.png>

<!-- https://git.ecd.axway.org/apigw/apigateway-openlogging-elk/-/tree/master/apm -->

Elastic [Application Performance Monitoring](https://www.elastic.co/observability/application-performance-monitoring) (APM) allows you to monitor the APIBuilder4Elastic better than just using logs directly from the API Builder. Of course, you can also add other API-Builder services or other applications.

After APM is set up, you can use **Kibana > Observability > APM** (**placeholder**: where's this Menu located?) to access a series of application performance dashboards. The following are examples of ???

APM Services overview:

(**placeholder: screenshot**)

Selected APIBuilder4Elastic - Overview:

(**placeholder: screenshot**)

Selected APIBuilder4Elastic - Dependencies:

(**placeholder: screenshot**)

Selected APIBuilder4Elastic - Errors:

(**placeholder: screenshot**)

Selected APIBuilder4Elastic - Metrics:

(**placeholder: screenshot**)

APIBuilder4Elastic is prepared to use an external APM server or to deploy directly with Operational Insights. You can enable the APM service via Docker Compose in your `.env` file or in the Helm chart. The activation is divided into 2 steps, the setup and start of the APM server and the connection to the APIBuilder4Elastic. The setup steps is not necessary if you use an existing external APM server.

## Activate APM in a Docker Compose deploy

Follow these steps to activate APM:

### 1. Start the APM server

This launches the APM service as a Docker container. The configured Elasticsearch hosts (ELASTICSEARCH_HOSTS) and certificates are used to connect to Elasticsearch. If you want to have user login enabled, you also need to set the APM_USERNAME and APM_PASSWORD parameters. Please make sure that the initial setup user must have rights to create templates (e.g. the elastic user).

```bash
docker-compose -f apm/docker-compose.apm-server.yml up -d
```

This starts the APM service as a Docker container:

```none
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED        STATUS                  PORTS                                            NAMES
cdffd8117b8b   docker.elastic.co/apm/apm-server:7.15.2                "/usr/local/scripts/…"   16 hours ago   Up 16 hours             0.0.0.0:8200->8200/tcp                           apm-server
```

To make use of the APM-Service in APIBuilder4Elastic, you need to set the parameter `APM_ENABLED=true`. This will cause the API-Builder process to attempt to connect to the configured APM-Server. During start, at the very beginning, the following will be logged:

```bash
Application performance monitoring enabled. Using APM-Server: https://axway-elk-apm-server:8200
```

### 2. Activate APM service in APIBuilder4Elastic

```bash
Application performance monitoring enabled. Using APM-Server: https://axway-elk-apm-server:8200
```

If no APM service is specified, then the following default is used: <https://apm-server:8200>. However, you can configure it yourself using the parameter: APM_SERVER. For further parameters please refer to the env-sample.

## Activate APM in Helm

If you are deploying Operational Insights with Helm, follow this section:

### 1. Start the APM server in your Helm chart

In your `local-values.yml` you activate the APM server by means of. Additionally, please set the Elasticsearch cluster UUID to include the APM server in stack monitoring:

```bash
apm-server:
  enabled: true
  elasticsearchClusterUUID: 3hxrsNg6QXq2wSkVWGTD4A
```

### 2. Activate APM service in APIBuilder4Elastic in your Helm chart

Now you can further configure APIBuilder4Elastic with the following parameters.

```bash
apibuilder4elastic:
  apmserver:
   # If you want to use an external APM-Server, you can configure the URL here. 
   # Defaults to the internal APM-Server service if enabled
   serverUrl: {}
   # The Certificate-Authory to validate the APM-Server certificate
   # serverCaCertFile: "config/certificates/my-apmserver-ca.crt"
   # You may disable the certificate check
   # verifyServerCert: "false"
```

## Disk usage monitoring

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#disk-usage-monitoring -->

It is important that you monitor the disk usage of the Elasticsearch cluster and get alarmed accordingly. Elasticsearch also independently monitors disk usage against pre-configured thresholds and closes write operations when the high disk watermark index is exceeded. This means that no more new data can be written.

To avoid this condition, your alerts should already warn below the Elasticsearch thresholds. The thresholds for Elasticsearch:

* Low watermark for disk usage: 85%.
* High watermark for disk usage: 90%

So your alerts should report a critical alert before 90%. For more information, see [Disk-based shard allocation settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-cluster.html#disk-based-shard-allocation).
