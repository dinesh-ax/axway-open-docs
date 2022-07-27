---
title: Amplify Analytics Operational Insights
linkTitle: Configure Operational Insights
weight: 80
date: 2022-06-09
hide_readingtime: true
description: Configure API Gateway with Elasticsearch to manage your metrics database and use Operational Insights component to observe millions of requests across different API Gateway instances.
---

Amplify Analytics solution (Business insights, Consumer insights, and Operational insights) (**link to the overall solution**) helps you to leverage information about your APIs usage to not only manage your technology infrastructure and operations better, but also to generate additional insights for your businesses.

Operational insights is one of the components of Amplify Analytics solution. It works on top of the analytics stack, [Elasticsearch, Logstash, and Kibana](https://www.elastic.co/elasticsearch/) (ELK), and its advantage over [API Gateway Traffic Monitor](/docs/apimanager_analytics/analytics_intro/) is that it allows you to observe millions of requests across different API Gateway instances in a long time frame. It uses Elasticsearch as the data source for API Gateway built-in [Traffic Monitor](/docs/apimanager_analytics/analytics_intro/), which resolves the performance and scalability challenges with OpsDB database.

## How it works

Operational insights imports the log files produced by the API Gateways around the globe into an Elasticsearch cluster. Once the data has been indexed, it can be used by various clients. One of the clients is Kibana, to visualize the data in dashboards, and another client is the standard API Gateway Manager Traffic Monitor, which can access the data.

The following are the **components** used in Operational insights component:

* **Filebeat**: Runs directly on the API gateways as a Docker container or as a native application. It streams the generated logfiles to the deployed Logstash instances. The OpenTraffic log, Event log, Trace messages and Audit logging are streamed. All components, besides Filebeat, can be deployed and configured in a highly available way.
* **Logstash**: Pre-process the received events before sending them to Elasticsearch. As part of this processing, some of the data (for example, API details) are enriched using APIs provided by [API Builder](/docs/api_mgmt_overview/api_mgmt_components/apibuilder/). This makes it possible to access additional information such as policies, custom properties, and so on in Kibana and other applications. This information is cached in Memcached.
* **Memcached**: Used by Logstash to cache information (API details) retrieved from API Builder so that information does not have to be retrieved repeatedly.
* **API-Builder**: Perform the following tasks:
    * Provides some REST APIs for Logstash processing. For this purpose, it mainly uses the API Manager REST API to retrieve the information.
    * Provides the same REST API expected by the Traffic Monitor, but based on Elasticsearch. The Admin Node Manager is then redirected to the API builder traffic monitor API for some of the request.
    * Configures Elasticsearch for Operational insights. This includes index templates, ILM policies, and so on. This makes it easy to update Operational insights.
* **Elasticsearch**: Ultimately, all information is stored in an Elasticsearch cluster in various indexes and is thus available to Kibana and API Builder. After this data is indexed,it can also be used by other clients.
* **Kibana**: Used to visualize the indexed data in dashboards. Operational insights provides some default dashboards, but it is also possible to add custom dashboards to the solution.
* **Traffic Monitor**: TO DO

## Architecture using Docker Compose

The following image shows the overall architecture of the Elasticsearch components running with API Gateway Manager deployed with Docker compose:

![Docker Compose architecture](/Images/op_insights/op_insights_DockerComposeArchitecture.png)

<!-- <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#overview> -->

This architecture makes it possible to collect data from API Gateways running all over the world into a centralized Elasticsearch instance to have the data available with the best possible performance, independent from the network performance.

It also helps, when running API Gateway in a Docker environment, where containers are started and stopped, as it avoids to lose data when an API Gateway container is stopped.

For more examples of architectures, see [Docker Compose architecture examples](link).

## Architecture using Helm

<!-- <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm> -->

The following diagram shows an overview of the architecture to be deployed in the Kubernetes cluster. The example is for an environment where the API management platform is external to Kubernetes, so Filebeat is also external.

<!-- <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#architecture-overview> -->

![Helm architecture](/Images/op_insights/op_insights_HelmArchitecture.png)

## Key benefits

API Management Operational Insights components provides the following key benefits:

### Performance

When having many API Gateway instances with millions of requests, the API Gateway [traffic monitor](/docs/apim_reference/monitor_traffic_events_metrics/) can become slow and the observation period quite short. Operational insights solve that performance issue, and make it possible to observe a long time frame and get other benefits by using a standard external datastore, the Elasticsearch

### Visibility

The solution allows API Manager users to use the standard traffic monitor and see only the traffic of their own APIs. This allows API service providers who have registered their APIs to monitor and troubleshoot their own traffic without the need for a central team.

### Analytics

Deliver standard dashboards that provide analysis capabilities across multiple perspectives. It also allows you to add your own dashboards.

## High level steps to use Operational Insights

The following is a summary of the high level steps to use Operational Insights:

* Ensure that you have read the prerequisites section (**link**).
* Configure your system for a single node Elasticsearch cluster (**link**).
* Size your infrastructure (**link**).
* Configure your API Gateway Manager with Elasticsearch (**link**).
* Configure your system for a production environment (**link**).

## Monitoring and reporting with API Gateway Analytics

After you integrate Operational Insights component to you API Gateway Manger, you can avail of better performance to monitor the traffic of your APIs, and you can make use of a large range of Kibana dashboard to support you to understand and analyze your data from different perspectives.
