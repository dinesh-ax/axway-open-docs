---
title: Configure an Admin Node Manager policy
linkTitle: Configure an Admin Node Manager policy
weight: 65
date: 2022-08-12
description: Configure an Admin Node Manager policy.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/nodemanager -->

We recommend that you use the provided policy fragment, `policy-use-elasticsearch-api-7.7.0.xml` to automatically configure your Admin Node Manager policy, but if you would like to setup the policy follow this section.

## Create the policy manually

Create a new policy and name it, **Use Elasticsearch API**. This policy will decide on what API calls can be routed to Elasticsearch.

The following image shows how the policy looks like:

<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/node-manager-use-es-api.png>

## Extract query parameters into attributes

This **Extract REST Request Attributes** filter is used to extract given REST API query parameters into attributes, which is required to get the optional parameter, `useOpsdb` which can be used to skip Elasticsearch and use the internal OpsDB.

## Skip Elasticsearch

This **Compare Attribute** filter is used to check if the parameter `useOpsdb` is set to 'true'. If 'true', Elasticsearch is not used to handle this request.

To make use of this optional parameter, you must configure it in your `<apigateway>/config/acl.json` as an allowed parameter as follows. If you do not configure this parameter, the ANM will return a `403` error:

```bash
"ops_get_messages" : { "path" : "/ops/search?protocol=&format=&from=&count=&order=&rorder=&ago=&field=&value=&op=&jmsPropertyName=&jmsPropertyValue=&useOpsdb=" },
```

After enabling the parameter and force the use of OpsDB, you must send a request to the ANM Traffic Monitor. For example:

```bash
https://admin-nodemanaget:8090/api/router/service/instance-1/ops/search?useOpsdb=true
```

## Check whether endpoints are managed by Elasticsearch API

The **Compare Attribute** filter named **Is managed by Elasticsearch API?** checks for each endpoint based on the attribute `http.request.path` if the requested API can be handled by the API Builder ElasticSearch Traffic Monitor API.

As a basis for decision-making, a criteria for each endpoint needs to be added to the filter configuration.

The following endpoints are currently supported by the API Builder based Traffic Monitor API:

| Endpoint       | Expression               | Comment |
| :---          | :---                 | :---  |
| **Search**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/search$` | This endpoint which provides the data for the HTTP Traffic overview and all filtering capabilities|
| **Circuitpath**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/stream\/[A-Za-z0-9]+\/[^\/]+\/circuitpath$` | Endpoint which provides the data for the Filter Execution Path as part of the detailed view of a transaction|
| **Trace**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/trace\/[A-Za-z0-9]+[\?]?.*$` | Endpoint which returns the trace information and the **getinfo** endpoint which returns the request detail information including the http header of each leg|
| **GetInfo**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/[A-Za-z0-9]+\/[A-Za-z0-9]+\/[\*0-9]{1}\/getinfo[\?]?.*$` |Endpoint provides information for the Requesr- Response-Details|
| **Payload**     | `^\/api\/router\/service\/[A-Za-z0-9-.]+\/ops\/stream\/.*\/\d+\/(?:sent|received)$` |Payload endpoint|

The following image shows how the **Compare Attribute** filter looks like:

<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/IsmanagedbyElasticsearchAPI.png>

## Set region filter

The **Set Attribute** filter, `Set region filter`, creates a new attribute: `regionFilter`, which is used during the connect to restrict the result based on the region of the Admin Node Manager. It ods using the environment variable, `env.REGION`. This setup is optional.  

<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/setRegionFilter.png>

Sample: `region=${env.REGION == '[invalid field]' ? "" : env.REGION}`

## Add region filter

The **Scripting** filter, using Javascript, adds the **Region** filter, which is optional to the `http.request.rawURI` attribute.

```json
function invoke(msg) {
    var httpRequestRawURI = msg.get("http.request.rawURI");
    var regionFilter = msg.get("regionFilter");

    if (httpRequestRawURI.contains('?')) {
        httpRequestRawURI += "&" + regionFilter;
    } else {
        httpRequestRawURI += "?" + regionFilter;
    }
    msg.put("http.request.rawURI", httpRequestRawURI);
    return true;
}
```

## Connect to Elasticsearch API

The URL of the **Connect to URL** filter points to your running API Builder docker container and port, which defaults to `8889`, using the environment variable, `API_BUILDER_URL`.

Additionally, the URL is forwarding the optional region filter based on the configured REGION to ensure the Admin Node Manager loads the correct regional data.

Sample: `${env.API_BUILDER_URL}/api/elk/v1${http.request.rawURI}`

## Is not implemented

If a given protocol, such as `Directory` for instance, is not implemented, API Builder will return a `501` error to indicate the request should be handled by the OpsDB.

The following image shows how the **Compare attribute** filter looks like:

<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/is_not_implemented.png>
