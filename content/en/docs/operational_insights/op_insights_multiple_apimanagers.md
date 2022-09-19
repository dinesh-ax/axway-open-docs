---
title: Multiple API Managers
linkTitle: Multiple API Managers
weight: 55
date: 2022-09-19
description: Configure multiple API Managers
---

If you have several API Managers within your domain, you must configure a mapping of which group (groupId) belongs to which API Manager. The group-id represents the Domain-Group and is attached to each Open-Traffic- or Metric-Event. It's used by Logstash to send the request to the API-Builder Lookup-API and is used there to perform the lookup against the belonging API-Manager.

The following syntax is used for this:

```none
API_MANAGER=group-2|https://api-manager-1:8075, group-5|https://api-manager-2:8275
```

```bash
helm example update the apimgrUrl variable in the values.yaml:
apibuilder4elastic:
  apimgrUrl: "group-2|https://api-manager-1:8075, group-5|https://api-manager-2:8275"
```

In this example, all events of `group`-2 are enriched with the help of the API manager (<https://api-manager-1:8075>) and of `group-5` accordingly with <https://api-manager-2:8275>.

### Configure different topologies and domains

**placeholder**: is this supposed to be a subsection of Configure API Manager?

From version 2.0.0 it is additionally possible to use the solution with different domains and topologies. An example are different hubs (for example, US, EMEA, APAC), each having their own Admin Node Manager and API-Manager, but still all API Events should be stored in a central Elasticsearch instance.

For this purpose the configurable `GATEWAY_REGION` in Filebeat is used. If this region is configured (e.g. US-DC1), all documents from this region are stored in separate indices, which nevertheless enable global analytics in the Kibana dashboards.

![Eg:](/Images/op_insights/op_insights_index_per_region.png)


Also in this case, the API Managers must or can be configured according to the Region & Group-ID of the event. Example:

```bash
docker-compose example - set the API_MANAGER variable in the .env file to multiple apimanagers:
API_MANAGER=https://my-apimanager-0:8075, group-1|https://my-api-manager-1:8175, group-5|https://my-api-manager-2:8275, group-6|US|https://my-api-manager-3:8375, group-6|eu|https://my-api-manager-4:8475
```

```bash
helm example update the apimgrUrl variable in the values.yaml:
apibuilder4elastic:
  apimgrUrl: "https://my-apimanager-0:8075, group-1|https://my-api-manager-1:8175, group-5|https://my-api-manager-2:8275, group-6|US|https://my-api-manager-3:8375, group-6|eu|https://my-api-manager-4:8475"
```

In this example, API-Managers are configured per Region & Group-ID. So if an event is processed which has a Region and Group-ID matching the configuration, then the configured API-Manager is used. This includes the lookup for the API details as well as the user lookup for the authorization.
If the region does not fit, a fallback is made to a group and last but not least to the generally stored API manager.
A configuration only per region is not possible!

When the API Builder is started, to validate the configuration, a login to each API-Manager is performed. Currently the same API manager user (API_MANAGER_USERNAME/API_MANAGER_PASSWORD) is used for each API Manager.

### Configure Admin Node Manager per region

**placeholder**: is this supposed to be a subsection of Configure API Manager or Configure different topologies and domains?

If you use the solution with multiple regions and different domains, all events/documents are stored in ONE Elasticsearch. Therefore you also need to tell the Admin-Node-Manager in each region, which data (indices) to use. If you don't do that, the Admin-Node-Manager will show the entire traffic from all regions which may not be desired but is also possible.
To do this, you need to store the appropriate region, which is also specified in the Filebeats for the API gateways, in the conf/envSettings.props file and restart the node manager. Example: REGION=US
This way the Admin-Node-Manager will only select data from these regional indexes. Learn more about the Admin-Node-Manager configuration [here:](/docs/operational_insights/op_insights_configANM.md)

