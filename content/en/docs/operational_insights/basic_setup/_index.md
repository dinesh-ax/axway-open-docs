---
title: Configure a basic setup
linkTitle: Basic setup
weight: 30
no_list: true
date: 2022-07-20
description: Configure a basic setup for Operational Insights to test Elasticsearch in a single instance.
---

The basic setup explains the individual components, how they can be deployed and work together. This is a fully functional configuration to get you started as fast and as easy as possible, and which allows you to try out Operational Insights and gain experience, and be prepared for the deployment in your production environment.

{{< alert title="Note">}}This configuration is recommended for development environments.{{< /alert >}}

After completing the basic setup you will have a single node Elasticsearch instance including multiple Kibana dashboards. At this point, API Gateway Manager is not yet configured, therefore this single instance receives data from 1 to many API Gateways via Filebeat, Logstash, and API Builder, and is accessible via the Traffic Monitor.

You can also import and use the sample Kibana dashboard or create your own visualizations.

This basic setup uses minimal parameters to run and test Operational Insights on a machine with at least 16 GB of RAM including the API Management platform (for example, like the Axway internal API Management reference environment). Therefore, this is not suitable for a production environment without further configuration.

For a production environment, check the parameters mentioned in the [env-sample](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/7d91baaf8009ceb09ac9f3889752912b17b83736/env-sample#L604-L648) and set them if necessary.

You can deploy Operational Insights using Docker Compose or Helm charts. The following sections cover the common setup for both Helm and Docker Compose options.

## Enable Open Traffic Event Log

You must [enable Open-Traffic Event log](/docs/apim_administration/apigtw_admin/admin_open_logging/#configure-open-traffic-event-logging) for your API Gateway instances to allow your log files to be created by default at `apigateway/logs/opentraffic`. This location is required for when configuring Filebeat later.

To avoid data loss, it is strongly recommended to increase the disk space for the Open Traffic logs from 1 GB to at least 8 GB, particularly if you have a lot of traffic. For example, if you have 100 TPS on one API Gateway instance, depending on your custom policies, the oldest log file will be deleted after approximately 30 minutes with only 1 GB OpenTraffic log configured. If for any reason Filebeat and Logstash are not running to process events for more than 15-20 minutes, you will have a loss of data as it also takes some time to catch up.

## Install Kibana dashboards

To install the Kibana dashboards, use the Kibana Menu **Stack Management > Saved Objects > Import saved objects** and select the following file from the release package, `kibana/dashboards/7/*.ndjson`.

In the **Import options** window, select **Automatically overwrite conflicts**. This is the default setting, and you must always select it regardless of whether you are installing the dashboards for the first time or after an update.

You can create customized visualizations and dashboards, but do not change the existing ones as they will be overwritten with the next update. If you have created your own visualizations and dashboards, they will not be changed by the import.

## Setup Admin Node Manager

Watch this video for an overview, [Traffic-Monitor and Kibana Dashboard](https://youtu.be/OZ0RNnqE6hs).

Configure Admin Node Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances. The Admin Node Manager API listens to port 8090 for administrative traffic by default and itt is responsible for serving the Traffic Monitor. You must now configured node manager to use the API Builder REST API instead.

To configure the node manager to render log data from Elasticsearch, follow these steps:

1. Open the node manager configuration in Policy Studio. For more information, see [Authentication and RBAC with Active Directory](/docs/apim_administration/apigtw_admin/general_rbac_ad_ldap/#use-the-ldap-policy-to-protect-management-services).
2. Import the provided policy fragment (`nodemanager/policy-use-elasticsearch-api-7.7.0.xml`) from the release package you have downloaded. This imports the policy, "Use Elasticsearch API".
    Do not use the XML file downloaded directly from the GitHub project as it might contain different certificates.
3. [Update main policy](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/nodemanager) by inserting the "Use Elasticsearch API" policy as a callback policy (see filter, **Shortcut filter**) into the main policy, **Protect Management Interfaces**, and wire it like shown in the following image.
    **IMAGE**
4. Disable the audit log for **Failure** transactions to avoid not needed log messages in the node manager trace file.
5. Open the `apigateway-install-dir>/apigateway/conf/envSettings.props` file and add the following new environment variable, `env.API_BUILDER_URL=https://apibuilder4elastic:8443`.
6. If you are using [multiple regions](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#different-topologiesdomains), you must configure the appropriate region to restrict the data of node manager to the correct regional data, for example, `env.REGION=US`.
7. Copy the new configuration from the Policy Studio project folder (path on Linux, `/home/<user>/apiprojects/\<project-name\>`) back to the Admin Node Manager folder (`\<install-dir\>/apigateway/conf/fed`).
8. Restart the Admin Node Manager.

## Traffic Monitor view for API Manager users

Watch this video for an overview, [Traffic Monitor for API Manager users](https://youtu.be/X08bQPC1sc4).

In larger companies hundreds of API service providers are using the API Manager or the APIM-CLI to register their own services and APIs. While the service providers require access to the Traffic Monitor to monitor their own APIs independently, during registration, the corresponding APIs are assigned to API Manager organizations, which logically split them up, but, the standard traffic monitor does not know the organization concept and therefore, cannot restrict the view for a user based on the organization of an API.

Operational Insights solves this problem by storing the API transactions in Elasticsearch with the appropriate organization. Then, the API organization is used when reading the traffic data from Elasticsearch according to the following table.

| API Gateway Manager | API Manager | Restriction                       | Comment     |
|---------------------|-------------|-----------------------------------|-------------|
| Administrator       | N/A         | Unrestricted access               | A Gateway Manager user is considered as an Admin when they own the permission, `adminusers_modify`.  |
| Operator            | API Admin   | All APIs having a Service-Context | By default, each API processed by the API Manager has a Service-Context. Pure Gateway APIs, for example, `. /healthcheck`, will not be visible. |
| Operator            | Org Admin   | APIs of its own organization      | Such a user will only see the APIs that belong to the same organization as himself.  |
| Operator            | User        | APIs of its own organization      | The same rules apply as for the Org-Admin           |

### Setup API Manager user in API Gateway Manager

To give API Manager users an restricted access to the API Gateway Traffic-Monitor, the user must be configured in the API-Gateway-Manager with the same login name as in the API-Manager. Here, for example, an LDAP connection can be a simplification.
Additionally you need to know, that by default, all API-Gateway Traffic-Monitor users get unrestricted access only if one of their roles includes the permission: `adminusers_modify`. But typically, only a full API-Administrator has this right and therefore only these users can see the entire traffic. All other users get a restricted view of the API traffic and will be authorized according to the configured authorization configuration described below.
However, with the parameter: UNRESTRICTED_PERMISSIONS you can configure which right(s) a user must have to get unrestricted access.

### Customize user authorization

By default, the organization(s) of the API-Manager user is used for authorization to the Traffic-Monitor. This means that the user only sees traffic from his the organization(s) he belongs in API-Manager. From a technical point of view, an additional filter clause is added to the Elasticsearch query, which results in a restricted result set. An example: `{ term: { "serviceContext.apiOrg": "Org-A" }}`.

Since version 2.0.0, it is alternatively possible to use an external HTTP service for authorization instead of the API Manager organizations, to restrict the Elasticsearch result based on other criterias.

To customize user authorization, you need to configure an appropriate configuration file as described here:

```bash
# Copy the provided example 
cp ./config/authorization-config-sample.js ./config/my-authorization-config.js
# Customize your configuration file as needed
vi ./config/my-authorization-config.js
# Setup your .env file to use your authorization config file
vi .env
AUTHZ_CONFIG=./config/my-authorization-config.js
# Recreate the API-Builder container
docker stop apibuilder4elastic
docker-compose up -d
```

In this configuration, which also contains corresponding Javascript code, necessary parameters and code is stored, for example to parse the response and to adjust the Elasticsearch query. The example: <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/config/authorization-config-sample.js> contains all required documentation.

Once this configuration is stored, the API Manager Organization based authorization will be replaced.

Note:

* Besides the API-Manager Organization autorization only `externalHTTP` is currently supported.
* Only 1 authorization method can be enabled
* It is also possible to disable user authorization completely. To do this, set the parameter: `enableUserAuthorization`: false.
* If you have further use-cases please create an issue describing the use-case/requirements.

## Next steps

This section covers the basic setup for any framework you with to use. To continue your basic configuration in specific frameworks, see:

* Basic setup for Helm (**link**).
* Basic setup for Docker compose (**link**).