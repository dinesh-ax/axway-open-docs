---
title: Additional features
linkTitle: Additional features
weight: 60
date: 2022-06-09
description: Configure additional features for Operational insights.
---

## Consult long-term Analytics

Operational insights supports long-term analytics capabilities out of the box in addition to relatively short-term operations and the corresponding real-Time dashboard.

For this purpose, the highly granular raw data received from API Gateway instances is transformed into entity-centric indices that are pre-aggregated and require only a fraction of the necessary disk space. Note that this reduces the ability to analyze data down to the minute.

To transform the raw data, the solution delivers and automatically installs a ready-made transformation job and provides corresponding dashboards.

(**instead of adding screenshots of the dashboards, would it be possible to show in a video? so we could add the link to the video here**)

The transformation works with a delay of 3 hours. This means that real-time data will show in the Quarterly/Yearly dashboards only after this time. This delay allows you to suspend ingesting data for a maximum of 3 hours, so the transformation job will not lose data.

Long-term Analytics data will only be available after approximately 1 hour of the start of the transformation. Until then, the corresponding dashboards will show errors. For example,

```bash
The field "name-of-a-field" associated with this object no longer exists. Please use another field. Please wait at least 1 hour for the data to be prepared accordingly or create the transformation job manually executing the following command in the APIBuilder4Elastic container:

wget --no-check-certificate https://localhost:8443/api/elk/v1/api/setup/transform/apigw-traffic-summary
```

## Support for Geolocation

You can see geomaps to see from which regions API requests are processed. These dashboards are broken down on the basis of real-time data in a map to the corresponding city and, for historical data, the number of requests per country.

(**screenshot** <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/geo-map-mixed.png>)

The IP address of the client is determined from the Transaction event log, converted to a corresponding location by the Logstash plugin [Geoip](https://www.elastic.co/guide/en/logstash/current/plugins-filters-geoip.html) and stored in Elasticsearch. The process is enabled by default, but can be disabled via the `GEOIP_ENABLED` [parameter](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/7d91baaf8009ceb09ac9f3889752912b17b83736/env-sample#L604-L648).

Most likely, your API management solution is running behind a firewall or load balancer, so the actual IP address of the client is not included in the event log. To pass the correct IP address to the solution, please configure a custom attribute (by default xForwardedFor) for the transaction event log, which contains the correct IP address. You can obtain this from the X-Forwarded-For header, for example.
