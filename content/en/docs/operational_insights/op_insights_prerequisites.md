---
title: Operational Insights prerequisites
linkTitle: Prerequisites
weight: 10
date: 2022-06-09
description: Prerequisites to configure Operational Insights in API Management.
---

Deploying Operational Insights requires knowledge of the Axway API Management solution, basic understanding of Docker, and a solid understanding of [HTTPS and server certificates](https://www.ssl.com/article/browsers-and-certificate-validation/) and how to validate them via trusted CAs.

You have two options to deploy Operational Insights:

* On a Docker orchestration platform, such as Kubernetes or OpenShift cluster, by using the provided Helm chart.
* On virtual machines with Docker installed, by using Docker Compose.

This page covers the prerequisites for both virtual machine based and Docker Orchestration platform deployments.

## General prerequisites

The following prerequisites are common for all types of deployment.

### Elastic stack

Operational Insights is based on the [Elastic Stack](https://www.elastic.co/elastic-stack/). It can run completely in docker containers, which for example are started on the basis of the `docker-compose.yaml` file or in a Docker Orchestration framework.

It is also possible to use existing components, such as an Elasticsearch cluster or a Kibana instance to avail of the flexibility of using, for instance, an Elasticsearch service at AWS or Azure, or use Filebeat manually installed on the API Gateway machines.

{{< alert title="Note" >}}
Operational Insights has been tested with Elasticsearch >7.10.x version.
{{< /alert >}}

### API Gateway and API Manager

Operational Insights is designed to work with classical and EMT API Management deployment models. Because it is mainly based on events given in the [Open Traffic Event Log](/docs/apim_reference/monitor_traffic_events_metrics/#open-traffic-event-log-settings), you must ensure that this setting is enabled. Also, event logs are indexed and stored in Elasticsearch, which allows for system-monitoring information and to highlight annotations based on [governance alerts](/docs/apim_administration/apimgr_admin/api_mgmt_alerts/#alert-descriptions) in API Manager.

Operational Insights works only with API Management 7.7 [January 2020](/docs/apim_relnotes/) onwards. Because of `Dateformat` changes in the open traffic format, older versions of API Gateway will shown errors in the Logstash processing.

## Helm prerequisites

The following are the requirements to deploy Operational Insights on Docker Orchestration platforms using Helm charts.

* Kubernetes >= 1.19 (At least three dedicated worker nodes for three Elasticsearch instances).
* Helm >= 3.3.0
* kubectl installed and configured.
* OpenShift (not yet tested (Please create an issue if you need help))
* See required resources - <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#required-resources>
* Ingress controller already installed. See <https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/>

### Kubernetes and OpenShift knowledge

Even though the Helm chart makes deploying the solution on Kubernetes or OpenShift quite easy, extensive knowledge about these platforms and Helm is mandatory.

You must be familiar with:

* Concepts of Helm, how to create a Helm chart, installation and upgrade.
* Kubernetes resources such as deployments, configMaps, and secrets.
* Kubernetes networking, Ingress, Services, and load balancing.
* Kubernetes volumes, persistent volumes, and volume mounts.

## Docker Compose prerequisites

The following are the requirements to deploy Operational Insights using Docker Compose on virtual machines with Docker installed.

### Docker

Components such as the API Builder project are supposed to run as a Docker container. The Elasticsearch stack is using standard Docker images, which are configured with environment variables and some mount points, allowing for great flexible where you can run them with the provided Docker Compose or with a Docker Orchestration platform (Kubernetes, OpenShift) to get elastic scaling and self-healing.

### Docker Compose

Your virtual machines must have Docker Compose installed and executable.

### Elastic stack for Docker Compose

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#requirements -->

If you are using an existing Elastic Search environment including Kibana, the following requirements apply.

* Minimal Elasticsearch is version 7.10.x with X-Pack enabled
* Depending on the traffic enough disk-space available (it can be quite heavy depending on the traffic volume)

### Users and roles

The following table represents a suggestion of which roles should be created for the solution to work. You are also welcome to divide the roles differently and assign them to the users accordingly.

| Role              | Cluster privileges                                                    | Index privileges                   | Kibana                   |
| :---              | :---                                                                  | :---                               | :---                     |
| [axway_apigw_write](elasticsearch/usersAndRoles#role-axway_apigw_write) | `monitor`                                                             | `apigw-* - write`                  | No                       |
| [axway_apigw_read](elasticsearch/usersAndRoles#role-axway_apigw_read)  | `monitor`                                                             | `apigw-* - read`                   | No                       |
| [axway_apigw_admin](elasticsearch/usersAndRoles#role-axway_apigw_admin) | `monitor`, `manage_ilm`, `manage_index_templates`, `manage_transform` | `apigw-* - monitor, view_index_metadata, create_index`, `apim-* - read,view_index_metadata`| Yes (All or Custom)  |
| [axway_kibana_write](elasticsearch/usersAndRoles#role-axway_kibana_write)| None                                                                 | None                               | Yes (Analytics All)      |
| [axway_kibana_read](elasticsearch/usersAndRoles#role-axway_apigw_read) | None                                                                  | None                               | Yes (Analytics Read)     |

The following table assumes that the same user should also be used for stack monitoring. You can also split this into two users if necessary.

| Username                  | Roles                                                        | Comment                                                        |
| :---                      | :---                                                         | :---                                                           |
| [axway_logstash](elasticsearch/usersAndRoles#user-axway_logstash)            | `axway_apigw_write`, `logstash_system`                       | Parameter: `LOGSTASH_USERNAME` and `LOGSTASH_SYSTEM_USERNAME`  |
| [axway_apibuilder](elasticsearch/usersAndRoles#user-axway_apibuilder)          | `axway_apigw_read`, `axway_apigw_admin`                      | Parameter: `API_BUILDER_USERNAME`                              |
| [axway_filebeat](elasticsearch/usersAndRoles#user-axway_filebeat)            | `beats_system`                                               | Parameter: `BEATS_SYSTEM_USERNAME`                             |
| [axway_kibana_read](elasticsearch/usersAndRoles#user-axway_kibana_read)         | `axway_apigw_read`, `axway_kibana_read`                      | Read only access to Dashboards                                 |
| [axway_kibana_write](elasticsearch/usersAndRoles#user-axway_kibana_write)        | `axway_apigw_read`, `axway_kibana_write`                     | Write access to Dashboard, Visualizations and some other information such as ILM-Policies.                      |
| [axway_kibana_admin](elasticsearch/usersAndRoles#user-axway_kibana_admin)        | `axway_apigw_read`, `axway_apigw_admin`, `axway_kibana_write`, `monitoring_user` | Access to Stack-Monitoring and APM         |

## Next steps

After you ensured that you have all prerequisites in place, you can [setup a basic environment](/docs/amplify_analytics/op_insights_config_elastic_singlenode) to test the solution in a development environment.
