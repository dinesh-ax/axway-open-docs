---
title:  Operational Insights changelog
linkTitle: Changelog
weight: 300
date: 2022-07-28
description: Changelog OF Axway API Management based on the Elastic stack.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/CHANGELOG.md -->

## [2.0.0] 2021-01-26

### Added

* Support for API custom properties.
    * Configured custom properties are indexed according to their configuration in API Manager. They can be used to create custom dashboard or perform custom queries.
* Regional support to store data from different regions separately.
    * Makes it possible to use Operational Insights with multiple domains. See <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#different-topologiesdomains>.
* Local API lookup for native API enrichment and disable APIs and events. See <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/README.md#setup-local-lookup>.
* Payload-Support
    * See <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#traffic-payload>.
* Flexible User Authorization. See, <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/README.md#customize-user-authorization>.
* Life cycle management. See, <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/README.md#lifecycle-management>.
* API Overview dashboard shows average response time.
* Initial early version of new Platform-Health-Dashboard.
    * The dashboard will be improved with future releases.
* Added Metricbeat support.
    * Used to monitor the Elastic stack.
    * Used to monitor API Gateways & Docker-Containers (Optional).

### Changed

* Updated API-Builder to version Dubai
* Elasticsearch configuration now managed by API-Builder project and no longer by Logstash
    * Index-Templates
    * ILM-Policies has been added
    * Rollup-Jobs has been added

### Fixed

* Trace-Messages for an API-Request now sorted correctly.
* API-Manager configuration not properly injected into API-Builder container.
* Kibana failed to start, when using multiple Elasticsearch hosts.

## [1.0.0] 2020-10-01

### Added (placeholder: Markdownlinter doesn't allow repetition of headings)

* Initial version