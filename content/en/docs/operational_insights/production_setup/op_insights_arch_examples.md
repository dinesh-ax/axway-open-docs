---
title: Docker Compose architecture examples
linkTitle: Architecture examples
weight: 30
date: 2022-07-27
description: Examples of Docker Compose architecture.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/architecture -->

Operational Insights was designed to scale with growing requirements. Therefore, each component (Logstash, API-Builder4Elastic, and so on) can be scaled in isolation. For example, if more Logstash capacity is needed, but no additional API-Builder4Elastic, Logstash can be scaled individually. This is the main reason why the solution is deployed based on Docker containers. Another reason is to make setup and update as easy as possible.

This section present various architecture deployment options. These should help to embed the solution into an existing infrastructure.

## AWS deployment based on EC2 machines

This section describes an example deployment architecture based on classic AWS-EC2 instances for a high availability solution in an AWS region with 2 availability zones. Operational Insights is completely deployed in this architecture. This means that Kibana, Elasticsearch and Filebeat are all deployed based on the project. Some notes on the architecture:

* Operational Insights is highly available with three Elasticsearch nodes.
    * The indices are stored by Elasticsearch on both nodes (Replica Shards).
    * A shared volume is not necessary.
* All clients (Logstash, API-Builder, Filebeat, Kibana) are configured on both available Elasticsearch nodes.
    * If one Elasticsearch node fails, the clients use the two nodes.
    * The Elasticsearch connection of Filebeat is used for sending telemetry data.
* API builder lookup is done within the availability zone. If API Manager is not available, the documents will continue to be initialized and an error message will be displayed in the traffic monitor.

The following diagram shows an **AWS EC2 HA setup with 1 Region and 2 Zones** example architecture.

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/architecture/aws-ec2-ha-one-region-2-zones/architecture-overview.png -->

![Architecture of AWS deployment based on EC2 machines](/Images/op_insights/op_insights_arch_awsEc2HaOneRegionTwoZones.png)

## Classic setup with native Filebeat

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/architecture/classic-simple-filebeat-native/architecture-example-1-classic-deployment.png -->
Example of a classic deploy of API Gateway with three nodes.

![Architecture of API Gateway classic deploy with three nodes](/Images/op_insights/op_insights_arch_oneClassicDeploy.png)

## Kubernetes deployment

For more information on how to deploy Operational Insights on Kubernetes, see [Configure a production setup for Helm](/docs/operational_insights/basic_setup/op_insights_setup_basic_helm/).

## Architecture FAQ

This section (**placehloder**)

### Can we support Operational Insights in non-docker mode?

No, the solution is designed to run based on Docker containers. It is also planned that the solution can be deployed on Kubernetes/OpenShift using HELM charts. A non-Docker mode would be the opposite direction.

### Can we get rid of API Builder and instead leverage policies in API Gateway/Manager for API detail lookup and other requirements?

No, a large part of the logic of the solution is in the API Builder application. Implementing this in policies might be possible, but managing & updating the individual customer installations would be very time-consuming and error-prone. So the customer has to reference the appropriate API-Builder image and you know by version exactly what code base the customer is running.

### Can we minimize the number of dependencies?

Elastic Search, Logstash, Kibana and FileBeat agents are mandatory.

### Can API Builder be made optional?

No, API-Builder is a substantial part of the solution. It’s exposing the fully tested Traffic-Monitor API, manages Elasticsearch, provides Lookup-APIs. Everything fully automated tested.

### Is it possible to use another caching solution than Memcached?

No, that is not possible. Logstash uses a filter to communicate with Memcached and is integrated into the Logstash pipelines. Only the interaction of the pipelines & Memcached has been tested and is thus supported.