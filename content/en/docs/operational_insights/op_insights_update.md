---
title: Update Operational Insights
linkTitle: Update Operational Insights
weight: 200
date: 2022-07-28
description: Instructions on how to update Operational Insights between versions.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/UPDATE.md -->

The basic principle of Operational Insights is that no files delivered with a release are changed. This is the best way to ensure the updated without any problems. Only your `*.env` file must be carried over from one release to the next. It is not planned to backport bugfixes (besides for the Axway Supported version) or enhancements in older releases.

This section describes how to update based on Docker Compose deployment. If you have deployed the solution on a Kubernetes cluster using Helm, see Upgrade a Helm-Deployment. <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#upgrade-the-release>

## Zero downtime upgrade

With the following steps you can update the solution without downtime. Of course, this requires that all components (Logstash, API Builder, Memcached) are running at least 2x and configured accordingly. So all filebeats have to communicate with all logstash hosts.

## General overview

The core component is the API Builder application which provides the information about the necessary configuration. In principle, it contains the desired or necessary state suitable for the version, especially about the Elasticsearch configuration, such as index templates, ILM policies, etc. If the version is updated, the API builder checks the current configuration in Elasticsearch and adjusts it if necessary to fit the corresponding version. This includes necessary changes for bug fixes or enhancements.

### Upgrade approach

* Load and unpack the current or desired release. It is recommended to unpack it next to the existing release.
* Copy your existing .env file from the current installation.
    * We recommend you to use a symlink to a central `.env` file, which should also be versioned if necessary. It is pointed out in this document, if parameters have changed or new ones have been added.
* Enure that you carry over your own certificates into the new release.
* Depending on which components have changed,
    * These containers must be stopped and then restarted based on the new release with `docker-compose up -d`, for example, if no change is noted in logstash, then this component can continue to run.
* API Builder delivers the configuration for Elasticsearch as well.
    * If API Builder is updated, then it checks on restart and periodically if the Elasticsearch configuration is correct. In addition, API builder checks whether the filebeat and logstash configuration corresponds to the expected version.

## Release history - Changed components

This table should help you to understand which components have changed with which version. For example, it is not always or very rarely necessary to update Filebeat. If the Elastic version has changed between the some releases, you do not necessarily have to follow it. Of course it is recommended to be on a recent Elastic stack version, because only for this version bugfixes are released by Elastic. Learn here how to update the Elastic stack.

On the other hand, the API builder Docker image, as a central component of the solution, will most likely change with each release.

(**placeholder**)

TO BE CONTINUED