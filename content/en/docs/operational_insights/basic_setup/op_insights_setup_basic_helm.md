---
title: Configure a basic setup for Helm charts
linkTitle: Basic setup for Helm
weight: 20
date: 2022-07-20
description: Configure a basic setup using Helm charts to test Elasticsearch in a single instance.
---

The following sections cover the configuration specific for deploying Axway API Management for Elastic solution on a Kubernetes or OpenShift cluster using Helm. The provided Helm chart is extremely flexible and configurable. You can decide which components to deploy, use your own labels, annotations, secrets, and volumes to customize the deployment to your needs.

<!-- <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm> -->

The following diagram shows an overview of the architecture to be deployed in the Kubernetes cluster. The example is for an environment where the API management platform is external to Kubernetes, so Filebeat is also external.

<!-- <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#architecture-overview> -->

![Helm architecture](/Images/op_insights/op_insights_arch_kubernetes.png)

## Before you start

* Ensure that you have all [prerequisites](/docs/amplify_analytics/op_insights_prerequisites/) in place.
* Ensure that you have applied the [basic configuration](link) for general frameworks.

## Get started

Watch [this video](https://youtu.be/w4n9JcBA-X4) to see how to deploy the solution on Kubernetes.

The video explains how you could start the solution with minimal setup.

## Next steps

After you have tried the solution in a single node elasticsearch scenario, ensure that you have read [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) then you can proceed to setup Operational insights in your production environment.
