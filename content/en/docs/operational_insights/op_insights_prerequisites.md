---
title: Operational Insights prerequisites
linkTitle: Prerequisites
weight: 10
date: 2022-06-09
description: Prerequisites to configure Operational Insights in API Management.
---

Deploying Operational Insights requires knowledge of the Axway API Management solution, basic understanding of Docker, and a solid understanding of [HTTPS and server certificates](https://www.ssl.com/article/browsers-and-certificate-validation/) and how to validate them via trusted CAs.

You have two options to deploy Operational Insights:

* On a container orchestration platform, such as Kubernetes or OpenShift cluster, by using the provided Helm chart.
* On servers or virtual machines (VMs) with Docker installed, by using Docker Compose.

This page covers the prerequisites for both of these options

## General prerequisites

The following prerequisites are common for all types of deployment.

### Elastic stack

Operational Insights is based on the [Elastic Stack](https://www.elastic.co/elastic-stack/). It can run completely in docker containers, which for example are started on the basis of the `docker-compose.yaml` file or in a Docker Orchestration framework.

It is also possible to use existing components, such as an Elasticsearch cluster or a Kibana instance, to avail of the flexibility of using, for instance, an Elasticsearch service at AWS or Azure, or use Filebeat manually installed on the API Gateway machines.

{{< alert title="Note" >}}
Operational Insights has been tested with Elasticsearch >7.10.x version.
{{< /alert >}}

### API Gateway and API Manager

Operational Insights is designed to work with Classic and EMT API Management deployment models. Because it is mainly based on events given in the [Open Traffic Event Log](/docs/apim_reference/monitor_traffic_events_metrics/#open-traffic-event-log-settings), you must ensure that this setting is enabled. Further details on payload settings below. 

Operational Insights works only with API Management 7.7 [January 2020](/docs/apim_relnotes/) onwards. Because of `Dateformat` changes in the open traffic format, older versions of API Gateway will display errors in the Logstash processing.

#### Configure traffic payload

The payload part of an API request is not written directly to the open traffic event log and therefore not is not stored in Elasticsearch.

Here is an example of what a payload might look like:


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

This payload, if not configured as explained below, will only be displayed as long as it remains in the OpsDB. After that, there will be no payload data available.

#### Configure the payload

In order to make the payload available in the Traffic Monitor via the solution, it must be exported from the API gateway to the runtime.

1. In the Policy Studio tree, select **Server Settings** > **Logging** > **Open Traffic Event Log**.
1. Select **Enable Open Traffic Event Log**.
1. Specify the required settings (for example, directory, max disk space, etc).
1. Under **Payload Settings** select **Use filesystem** (change the **Filesystem directory** value if you wish).
1. Click **Save** at the bottom right.
1. Click **Deploy** in the toolbar to deploy your settings to the API Gateway.

Repeat this step for each API gateway group for which you want to make the payload available. You may change the **Filesystem directory** path if required, for example to write the payload to an NFS volume.

#### Make the payload available

The saved payload must be made available to the API-Builder Docker container as a mount under `/var/log/payloads`. You can find an example in the docker-compose.yml:
`${APIGATEWAY_PAYLOADS_FOLDER}:/var/log/payloads` shared volume into the API Builder container.

#### Make the payload available per region

If you are using the region feature (**placeholder**, see Setup API-Manager > Different Topologies/Domains), that is, collecting API Gateways of different Admin Node Manager domains into a central Elasticsearch instance, then you also need to make the payload available to the API builder regionally separated. For example, if you have defined the region like, `REGION=US-DC1`, all traffic payload from these API Gateways must be made available to the API Builder as follows:

```bash
/var/log/payloads/us-dc1/<YYY-MM-DD>/<HH.MI>/<payloadfile>
```

So you need to make the existing structure available in a regional folder. For this, the region must be in lower case. And you have to configure each Admin-Node-Manager with the correct region (**placeholder**, see Setup API-Manager > Different Topologies/Domains).

Note:

* Payload handling is enabled by default. So it is assumed that you provide the payload to the API Builder container. Set the parameter `PAYLOAD_HANDLING_ENABLED=false` if you do not need this.
* Payload shown in the Traffic-Monitor UI is limited to 20 KB by default. If required the payload can be downloaded completely using the Save option.


Also, event logs are indexed and stored in Elasticsearch, which allows for system-monitoring information and to highlight annotations based on [governance alerts](/docs/apim_administration/apimgr_admin/api_mgmt_alerts/#alert-descriptions) in API Manager.

## Helm prerequisites

The following are the requirements specific to deploy Operational Insights on Docker Orchestration platforms using Helm charts.

* Kubernetes >= `1.19` (At least three dedicated worker nodes for three Elasticsearch instances).
* Helm >= `3.3.0`.
* `kubectl` installed and configured.
* OpenShift (not yet tested).
* [Required resources](/docs/operational_insights/op_insights_updatehelm/#required-resources).
* [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) already installed.

{{< alert title="Note for helm users" >}}
To access sample files referred to throughout this documentation for user customizations, please follow the instructions, in the [docker compose section](/docs/operational_insights/basic_setup/op_insights_setup_basic_docker/#download-and-extract-the-release-package) to download the released package 
{{< /alert >}}

### Kubernetes and OpenShift knowledge

Even though the Helm chart facilitates the deploy, extensive knowledge about these platforms and Helm is required:

* Concepts of Helm: how to create a Helm chart, installation, and upgrade.
* Kubernetes resources such as deployments, configMaps, and secrets.
* Kubernetes networking, Ingress, Services, and load balancing.
* Kubernetes volumes, persistent volumes, and volume mounts.

## Docker Compose prerequisites

The following are the requirements to deploy Operational Insights specifically using Docker Compose on physical servers or VMs with Docker installed.

### Docker

Components such as API Builder are supposed to run as a Docker container. The Elasticsearch stack is using standard Docker images, which are configured with environment variables and some mount points, allowing for great flexible where you can run them with the provided Docker Compose or with a Docker Orchestration platform (Kubernetes, OpenShift) to get elastic scaling and self-healing.

### Docker Compose

Your server or VMs must have Docker Compose installed and executable.

### Elastic stack for Docker Compose

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#requirements -->

If you are using an existing Elasticsearch environment including Kibana, the following requirements apply.

* Minimal Elasticsearch is version 7.10.x with X-Pack enabled.
* Depending on the traffic, enough disk-space available (it can be quite heavy depending on the traffic volume).

### Users and roles

The following table represents a suggestion of which roles should be created to work with Operational Insights, but you can configure the roles differently and assign them to the users accordingly.

| Role                                                                    | Cluster privileges  | Index privileges   | Kibana    |
| :---                                                                    | :---                | :---               | :---      |
| [axway_apigw_write](elasticsearch/usersAndRoles#role-axway_apigw_write)  | `monitor`          | `apigw-* - write`  | No         |
| [axway_apigw_read](elasticsearch/usersAndRoles#role-axway_apigw_read)    | `monitor`          | `apigw-* - read`   | No         |
| [axway_apigw_admin](elasticsearch/usersAndRoles#role-axway_apigw_admin)  | `monitor`, `manage_ilm`, `manage_index_templates`, `manage_transform` | `apigw-* - monitor, view_index_metadata, create_index`, `apim-* - read,view_index_metadata`| Yes (All or Custom)  |
| [axway_kibana_write](elasticsearch/usersAndRoles#role-axway_kibana_write)| None               | None               | Yes (Analytics All)   |
| [axway_kibana_read](elasticsearch/usersAndRoles#role-axway_apigw_read)   | None               | None               | Yes (Analytics Read)     |

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

After you ensured that you have all prerequisites in place, you can [setup a basic environment](/docs/operational_insights/basic_setup/) to run tests in a development environment.
