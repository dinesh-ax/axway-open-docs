---
title: Amplify Analytics Operational Insights
linkTitle: Configure Operational Insights
weight: 80
no_list: true
date: 2022-06-09
hide_readingtime: true
description: Configure API Gateway with Elasticsearch to manage your metrics database and use Operational Insights component to observe millions of requests across different API Gateway instances.
---

Amplify Analytics solution (Business Insights, Consumer Insights, and Operational Insights) (**link to the overall solution**) helps you to leverage information about your APIs usage to not only manage your technology infrastructure and operations better, but also to generate additional insights for your businesses.

Operational insights is one of the components of Amplify Analytics solution. It works on top of the analytics stack, [Elasticsearch, Logstash, and Kibana](https://www.elastic.co/elasticsearch/) (ELK), and its advantage over [API Gateway Traffic Monitor](/docs/apimanager_analytics/analytics_intro/) is that it allows you to observe millions of requests across different API Gateway instances in a long time frame. It uses Elasticsearch as the data source for API Gateway built-in [Traffic Monitor](/docs/apimanager_analytics/analytics_intro/), which resolves the performance and scalability challenges with OpsDB database.

## How it works

Operational Insights imports the log files produced by API Gateways instances into an Elasticsearch cluster. After the data has been indexed, it can be used by various clients, such as Kibana, which allows you to visualize the data in dashboards, and the API Gateway Manager **Traffic Monitor**, which can access the data.

The following are the tooling required to use Operational Insights:

### Filebeat

Runs directly on the API gateways as a Docker container or as a native application. It streams the generated logfiles to the deployed Logstash instances. The OpenTraffic log, Event log, Trace messages, and Audit logging are streamed. All components, besides Filebeat, can be deployed and configured in a highly available way.

### Logstash

Pre-process the receive events before sending them to Elasticsearch. As part of this processing, some of the data (for example, API details) are enriched using APIs provided by [API Builder](/docs/api_mgmt_overview/api_mgmt_components/apibuilder/). This allows to access additional information such as policies, custom properties, and so on in Kibana and other applications. This information is cached in Memcached.

### Memcached

Used by Logstash to cache information (API details) retrieved from API Builder so that information does not have to be retrieved repeatedly.

### API Builder

Perform the following tasks:

* Provides REST APIs for Logstash processing. For this purpose, it mainly uses the API Manager REST API to retrieve the information.
* Provides the same REST API expected by the Traffic Monitor, but based on Elasticsearch. The Admin Node Manager is then redirected to the API builder traffic monitor API for some of the request.
* Configures Elasticsearch for Operational Insights. This includes index templates, ILM policies, and so on. This makes it easy to update Operational Insights.

### Elasticsearch

Ultimately, all information is stored in an Elasticsearch cluster in various indexes, then it is available to Kibana and API Builder. After this data is indexed, it can also be used by other clients.

### Kibana

Used to visualize the indexed data in dashboards. Operational Insights provides some default dashboards, but it is also possible to add custom dashboards to the solution.

### Traffic Monitor

The standard API Gateway Traffic Monitor, which is shipped with Operational Insights, is based on a REST API that is provided by the Admin Node Manager. By default, the traffic information is loaded from the OBSDB running on each API Gateway instance.

API Builder is partly reimplementing this REST API, which allows the Traffic Monitor to use data from ElasticSearch instead of the internal OBSDB. That means, you can use the same tooling as of today, but the underlying implementation of the Traffic Monitor is now pointing to Elasticsearch instead of the internal OPSDB hosted by each API Gateway instance. This improves performance dramatically, as Elasticsearch can scale across multiple machines if required and other dashboards can be created for instance with Kibana.

The link between Elasticsearch and the API-Gateway Traffic Monitor is an API Builder project, that is exposing the same Traffic Monitor API, but it is implemented using Elasticsearch instead of the OPSDB. API Builder is available as a ready-to-use Docker image and pre-configured in the `docker-compose` file.

## Architecture using Docker Compose

The following image shows the overall architecture of the Elasticsearch components running with API Gateway Manager deployed with Docker compose:

![Docker Compose architecture](/Images/op_insights/op_insights_DockerComposeArchitecture.png)

<!-- <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#overview> -->

This architecture makes it possible to collect data from API gateways instances into a centralized Elasticsearch instance to have the data available with the best possible performance, independent from the network performance.

It also helps with data persistency, when running API Gateway in a Docker environment where containers are started and stopped, as it avoids to lose data when an API Gateway container is stopped.

For more examples of architectures, see [Docker Compose architecture examples](/docs/operational_insights/production_setup/op_insights_arch_examples/).

## Key benefits

Amplify Analytics Operational Insights components provides the following key benefits:

### Performance

When having many API Gateway instances with millions of requests, the API Gateway [Traffic Monitor](/docs/apim_reference/monitor_traffic_events_metrics/) can become slow and the observation period quite short. Operational Insights solve that performance issue, and make it possible to observe a long time frame and get other benefits by using a standard external datastore, the Elasticsearch.

### Visibility

API Manager users can use Traffic Monitor and see only the traffic of their own APIs. This allows API service providers, who have registered their APIs, to monitor and troubleshoot their own traffic without the need of a central team.

### Analytics

Deliver standard dashboards that provide analysis capabilities across multiple perspectives. It also allows you to add your own dashboards.

## High level steps to use Operational Insights

The following is a summary of the high level steps to use Operational Insights:

* Ensure that you have read the [prerequisites](/docs/operational_insights/op_insights_prerequisites/) section.
* Configure your system for a [single node](/docs/operational_insights/basic_setup/op_insights_setup_basic_docker/) Elasticsearch cluster.
* Size your [infrastructure](/docs/operational_insights/op_insights_infra_size).
* Configure Elasticsearch in your [API Gateway Manager](/docs/operational_insights/production_setup/op_insights_setup_prod_docker/#configure-api-manager).
* Configure your system for a [production](/docs/operational_insights/production_setup/op_insights_setup_prod_docker/) environment.

## Monitoring and reporting with API Gateway Analytics

After you integrate Operational Insights component to you API Gateway Manger, you can avail of better performance to monitor the traffic of your APIs, and you can make use of a large range of Kibana dashboard to support you to understand and analyze your data from different perspectives.

For more information, see [Enable monitoring and metrics](/docs/operational_insights/op_insights_monitoring/).

## Updates

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#updates -->

Operational Insights is under continuous development, and with each release the following artifacts might change:

* All Docker Compose files.
* Elasticstack version.
* Logstash pipelines.
* Elasticsearch configuration (for example, Index templates, ILM-Policies, Transformations).
* Filebeat configuration.
* API Builder Docker container version.
* Kibana dashboards.
* Scripts, and so on.

All these tools play together and only work if they are from the same release. Operational Insights component checks if, for example, the index templates have the required version.

It is strongly discouraged to make changes in any files, except the `.env` file and the `config` folder. These will be overwritten with the next release. This is the only way to easily update from one version to the next. If you need to change files, it is recommended to make this change automatically and repeatably (for example, <https://www.ansible.com>).

For more information on how to update Operational Insights, see [Update Operational Insights in Helm](/docs/operational_insights/op_insights_updatehelm/).