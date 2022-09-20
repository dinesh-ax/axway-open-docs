---
title: Local Lookup
linkTitle: Local lookup configuration
weight: 56
date: 2022-09-20
description: Use of local configuration files to enhance logs
---

## Setup local lookup

With our AAOI solution, it is possible to use local configuration files for the API lookup in addition to the API Manager. This makes it possible to:

* Enrich native APIs
    * APIs that have been exposed natively through the API Gateway policies not having a context
    * you can configure all information that would normally be retrieved from the API Manager via this lookup file (Organization, API-Name, API Method-Name, Custom-Properties)
    * for example display a /healthcheck as Healthcheck API in Kibana dashboards
* Ignore events
    * Additionall you can ignore OpenTraffc events so that they are not indexed or stored in Elasticsearch.
    * For example, the path: /favicon.ico, as this event does not add value.

Note that the local configuration file is used before the API Manager lookup. If there is a match, no lookup to the API Manager is performed.

To enable the local lookup, you must perform the following steps:

1. Add your config file. It is best to copy the delivered template: config/api-lookup-sample.json to your config/api-lookup.json.

    ```bash
    cp config/api-lookup-sample.json config/api-lookup.json
    ```

2. Activate the config file. In your .env file you must then enable the configuration file to be used by the API-Builder. To do this, configure or enable the following environment variable restart

    ```bash
For docker-compose use case set the env variable and restart:
    API_BUILDER_LOCAL_API_LOOKUP_FILE=./config/api-lookup.json
    docker stop apibuilder4elastic
    docker-compose up -d
    ```

    ```bash
For helm use case expose variable localAPILookup in **myvalues.yaml** file and run helm upgrade:
    localAPILookup: "./config/api-lookup.json"
    helm upgrade -n apim-elk -f myvalues.yaml axway-elk axway/aaoi-helm-prod
    ```

If an event is to be indexed, the API builder will try to read this file and will acknowledge this with the following error if the file cannot be found:

```bash
Error reading API-Lookup file: './config/api-lookup.json'
```

### Ignore events

At this point, we intentionally refer to events and not APIs, because different events (TransactionSummy, CircuitPath, TransactionElement) are created in the OpenTraffic log for each API call. Each is processed separately by Logstash and stored using an `upsert` in Elasticsearch with the same Correlation-ID. Most of these events contain the path of the called API, but unfortunately not all.
This is especially important when ignoring events so that they are not stored in Elasticsearch entirely. Since all events are processed individually, it must also be decided individually to ignore an event. Therefore, to ignore for example the Healthcheck API entirely, the following must be configured in the lookup file:

```bash
{
    "/healthcheck": {
        "ignore": true,
        "name": "Healthcheck API",
        "organizationName": "Native API"
    },
    "Policy: Health Check": {
        "ignore": true
    }
}
```

This will ignore events based on the path (TransactionSummary & TransactionElement) and the policy name (CircuitPath). You can see the result in the API-Builder log on level INFO with the following lines:

```log
Return API with apiPath: '/healthcheck', policyName: '' as to be ignored: true
```

or

```log
Return API with apiPath: '', policyName: 'Health Check' as to be ignored: true
```

The information is cached in Logstash via memcached for 1 hour, so you will not see the loglines in the API-Builder for each request. You can of course force a reload of updated configuration via docker restart memcached.

It is additionally possible to overlay the local lookup file per group and region. This allows you, for example, to return different information per region for the same native API or to ignore an API only in a specific region. To do this, create files with the same base name and then a qualifier for the group and/or for the group plus region. Examples:

```
api-lookup.group-2.json
api-lookup.group-2.us.json
```

Again, it is not possible to specify only the region, but only in combination with the appropriate group.
