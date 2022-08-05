---
title: Application and performance monitoring
linkTitle: Application and performance monitoring
weight: 50
date: 2022-08-05
description: placeholder - needs an introduction
---

<!-- https://git.ecd.axway.org/apigw/apigateway-openlogging-elk/-/tree/master/apm -->

**placeholder** - needs an introduction

Elastic [Application Performance Monitoring](https://www.elastic.co/observability/application-performance-monitoring) (APM) allows you to monitor the APIBuilder4Elastic (**placeholder: is this the correct name?**) better than just using logs directly from the API Builder. Of course, you can also add other API-Builder services or other applications.

After APM is set up, you can use Kibana -> Observability -> APM (**placeholder**: where's this Menu located?) to access a series of application performance dashboards. The following are examples of ???

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

## Setup APM

Operational Insights, respectively the APIBuilder4Elastic, is prepared to use an external APM-Server or to deploy directly with the solution. You can enable the APM-Service via Docker Compose in your `.env` file or in the Helm chart. The activation is divided into 2 steps. The setup and start of the APM server and the connection to the APIBuilder4Elastic. Step 1 is of course not necessary if you use an existing external APM server.

### Docker Compose

If you are deploying Operational Insights in Docker Compose, follow this section:

#### 1. Start the APM server

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

#### 2. Activate APM service in APIBuilder4Elastic

```bash
Application performance monitoring enabled. Using APM-Server: https://axway-elk-apm-server:8200
```

If no APM service is specified, then the following default is used: <https://apm-server:8200>. However, you can configure it yourself using the parameter: APM_SERVER. For further parameters please refer to the env-sample.

### Helm

If you are deploying Operational Insights with Helm, follow this section:

#### 1. Start the APM server in your Helm chart

In your `local-values.yml` you activate the APM server by means of. Additionally, please set the Elasticsearch cluster UUID to include the APM server in stack monitoring:

```bash
apm-server:
  enabled: true
  elasticsearchClusterUUID: 3hxrsNg6QXq2wSkVWGTD4A
```

#### 2. Activate APM service in APIBuilder4Elastic in your Helm chart

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
