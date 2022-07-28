---
title:  Operational Insights FAQ
linkTitle: FAQ
weight: 400
date: 2022-06-20
description: Frequently asked questions and answers on Operational Insights.
---

## Do I need an API Builder subscription?

No, the delivered API builder Docker image includes the community version and is supported as part of this project.

## Do I need an Elastic subscription?

The component can be operated in production without a subscription and is thereby community supported. In this case, **open an issue on this GitHub project**.

If you need official Elastic support to run your Elastic Stack, or additional features like alerts or machine learning for operation, then you need a subscription to Elastic. You can find more information at <https://www.elastic.co/pricing/>.

## Is this component officially supported by Axway?

Yes, the component is supported by Axway. A distinction is made between Community and Axway supported releases, which are marked accordingly in the release directory. For example, currently version 4.2.0 is Axway supported and accordingly you can create normal Axway support cases.

### Axway Supported Release

Aligned with the Axway API-Management release cycle, 1 community release is selected at a time, tested with the Axway API-Management solution, and finally marked as Axway supported. From this point on, this will be the Axway Supported Release and there will be bug fixes for it as needed, but no new features. Therefore, there is always only 1 Axway supported release.
However, this does not mean that the Elastic solution will only work with the corresponding Axway API-Management release, but rather that there will be no more bug fixes for other releases. If you find a bug in a release that is no longer supported, you will need to upgrade to the next Axway Supported version to get a bug fix.

### Community Releases

All other releases are community releases, which are released independently of the Axway API-Management release cycle. These always represent the latest state of features and bug fixes. For these releases, you receive community support according to the best-effort approach. Report issues, feature requests, etc. as an issue directly on the GitHub project.

## Do I need to be an Elastic Stack expert?

No, you do not need to be or become an expert on the Elastic stack. The solution configures the Elasticsearch cluster automatically including future updates. Logstash and Filebeat are preconfigured with the determined optimal parameters for the solution.

## Do I need a specific API Gateway version to use this solution?

Any API Gateway version that supports the  [Open Traffic Event Log](/docs/apim_reference/monitor_traffic_events_metrics/#open-traffic-event-log-settings) is supported. The component has been tested with API Management version 7.7 [July 2020](https://axway-open-docs.netlify.app/docs/apim_relnotes/20200730_apimgr_relnotes/) update.

## Will indexed data be deleted automatically?

Yes, as of version 2.0.0, each index created in Elasticsearch is assigned an Index Lifecycle Policy ([ILM](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)). This controls how long data is retained for each data type. For more information, see [Configure the retention period](/docs/amplify_analytics/op_insights_setup_production/#configure-the-retention-period).

## Can I use this component for multiple stages?

Technically this would certainly be possible, but definitely not recommended, as you would be mixing data from different stages, such as Prod, Test, Dev. You could certainly separate these with regions in individual indexes, but it still complicates the setup and data management. Another reason are updates of the solution which should certainly be done on a test environment before updating the production environment.

## Can I use my own existing Elasticsearch cluster?

Yes, you can use your own Elasticsearch cluster. As long as it's a >7.10.x version with X-Pack features enabled it's supported. For instance AWS-Elasticsearch service does not provide X-Pack is therefore not supported. Additionally >7.10 is required to run the transformation job correctly. For more information about missing buckets, see issue [#59591](https://github.com/elastic/elasticsearch/pull/59591) on elasticsearch GitHub project.

## Does the solution support high availability?

Yes, all components can be deployed in way to support HA for the entire platform? Load-Balancing is done internally, so no load balancers are required.

## Can I use the Filebeat version shipped with the API-Gateway?

No, as it is not a 7.x version and not tested with this solution at all.

## Can I run Filebeat as a native process?

Yes, you can run Filebeat natively instead of a Docker-Container if you prefer. As long as you are using the filebeat.yml and correct configuration that is supported. However, the solution has been tested with Filebeat 7.9, 7.10, 7.11 & 7.12. If you see performance degregation with an 'old' Filebeat the first advice would be to upgrade the Filebeat version.

## What happens if Logstash is down for a while?

In case Logstash is stopped, Filebeat cannot sent events any longer to Logstash. However, Filebeat remembers the position of the last sent events on each files and resumes at that position, when Logstash is available again. Of course, you have to make sure, files are available long enough. For instance the OpenTraffic-Event Logs are configured by default to 1GB, which is sufficient for around 30 minutes when having 300 TPS. You should increase the disk space.

## What happens if Filebeat is down for a while?

Filebeat stores the current position in a file (registry) storeing which events have already been passed on. If Filebeat is shut down cleanly and then restarted, Filebeat will pick up where it left off. Tests have shown that it can still be that a few events are lost.

## Can I run multiple Logstash instances?

Yes, Logstash is stateless (besides what is stored is Memcache), hence you can run as many Logstash instances as you like or need. In that case you have to provide multiple Logstash instances using the parameter LOGSTASH_HOSTS to the Filebeat instances so that they can load balance the events.

## Can I run multiple API-Builder instances?

Yes, and just provide the same configuration for each API-Builder Docker-Container. It's recommended to run the API-Builder process along Logstash & Memcache as these components are working closely together. Especially a low latency between Logstash & Memcache is crucial.

## Are there any limits in terms of TPS?

The solution was tested up to a maximum of 1,000 TPS and was thus able to process the events that occurred in real time. You can read more details in the [Infrastructure Sizing](/docs/amplify_analytics/op_insights_infra_size/) section. The solution is designed to scale to 5 Elasticsearch nodes as the indicies are configured on 5 shards. This means that Elasticsearch distributes each index evenly across the available nodes. More Elasticsearch nodes can also have an impact as there are a number of indices (e.g. per region, per type) and this allows Elasticsearch to balance the cluster even better. So in principle, there is no limit. Of course, the components like Logstash/API builder then need to scale as well.

## Can I customize / change the solution to my needs?

It is not recommended that you change the solution just by hacking/changing some of the files, as they are very likely be overwritten with the next version. With that, you will break the option to upgrade to a newer version. See here for more details.

## Are Trace-Messages stored in Elasticsearch?

Yes. Trace-Messages you see in the Traffic-Monitor for an API-Request are retrieved from the Elasticsearch cluster. Please have in mind, running an API-Gateway in DEBUG mode, handling millions of transactions a day, will signifcantly increase the disk-space requirements in Elasticsearch and the number of events to be processed. When enabling DEBUG, please monitor your Elasticsearch stack if event processing stays real-time.

## Can passwords given in the .env file be encrpyted?

No, but this file is used by docker-compose only. If you would like to avoid to have sensitive data stored, the recommended approach it deploy the solution in Kubernetes environment and store passwords in the Secure-Vault. With that, sensitive data is injected into the containers from a secure source.

## Why does Kibana dashboards show different volumes than the Traffic Monitor?

If the traffic monitor dashboard shows significantly more transactions than the Kibana dashboards, for example for the last 10 minutes, very likely the ingestion rate is not high enough. In other words, the ELK stack cannot process the data fast enough. If the behavior does not recover, you may need to add more Logstash and/or Elasticsearch nodes.

## Can I increase the disk volume on an Elasticsearch machine?

Yes, all indicies are configured to have one replica and therefore it's safe to stop a machine, increase the disk-space and restart it. Please make sure, the cluster state is green, before stopping an instance. The increased volume will be automatically detected by Elasticsearch and used to assign more shards to it. It's recommended to assign the same disk space to all Elasticsearch-Cluster nodes.

## During catch up, what should be the total events rate for Filebeat?

Tests show that Filebeat can send more than 3,000 events per second to Logstash instances. Of course, this number also depends on the event type. Trace messages can be processed faster than OpenTraffic event logs, for example. You can find an example [here](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/imgs/stack-monitoring/stack-monitoring-beats-instances.png).

## Can I use AWS Elasticsearch service?

No, the solution does not support AWS Elasticsearch Service because some important features that are part of X-Pack are not supported by AWS. For example, index lifecycle management (ILM).

## What is the average event latency for Logstash to process?

In a healthy environment, the event latency shown for Logstash event processing should be between 3-5ms. If latency is higher, please check that Memcached has enough resources and Filebeat is not reporting this error <https://www.elastic.co/guide/en/beats/filebeat/current/publishing-ls-fails-connection-reset-by-peer.html>. See the following examples for reference, <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/imgs/stack-monitoring>.

## Filebeat is reporting errors?

When Filebeat reports errors like the following, it might be that the registry is corrupt for any reason. When this happens, Filebeat basically stops for a second to send events, which may cause issues to stay real-time when running very high volume.

```bash
Harvester could not be started on new file: /var/log/opentraffic/group-2_instance-1_traffic.log, Err: error setting up harvester: Harvester setup failed. Unexpected file opening error: file info is not identical with opened file. Aborting harvesting and retrying file later again
```

## Which protocols are supported?

The following protocols are supported so far: HTTP, JMS and file transfer. This means that whenever you run a query via the API Gateway Traffic Monitor for one of these protocols, the data will come from Elasticsearch, otherwise it will continue to come from the OBSDB.

## Why only Administrators see JMS-Traffic?

JMS requests are not controlled by the API-Manager, therefore there is no association with an organization and therefore the result cannot be restricted accordingly. If you want, you can disable the complete user authorization by setting the parameter: enableUserAuthorization to false. See, <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#customize-user-authorization> or you can use the parameter `UNRESTRICTED_PERMISSIONS` to configure which users should see the entire traffic.

## Is EMT mode supported?

Yes, the component can be used when the API Management platform is deployed in a Docker orchestration platform in externally managed topology ([EMT](/docs/apim_installation/apigw_containers/container_intro/)) mode. With that, it is for instance possible to see traffic from containers (PODs) that have already being removed again in the traffic monitor. However, there is a limitation here that the server name is not displayed correctly. [Learn more](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/issues/114#issuecomment-864941677) on this limitation.

## Can I disable the OpsDB for traffic monitoring?

No. The settings can be found in Policy Studio **Server Settings > Monitoring > Traffic Monitor**. You must ensure that the traffic monitor is enabled, otherwise the policy execution path will not be displayed in the **Traffic Monitor** options. If you only use the [protocols supported](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#which-protocols-are-supported) by the Elastic solution, you can  reduce the size of the OpsDB if you like, since the data comes from the Elasticsearch database.

## Can I use the component without an API Manager?

Yes, from version >=4.2.0 this is possible. For this you have to set the parameter `API_MANAGER_ENABLED=false` and disable user authorization. See parameter, `[AUTHZ_CONFIG](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/7d91baaf8009ceb09ac9f3889752912b17b83736/env-sample#L604-L648)`.

## Can I use the solution with Elasticsearch >=8.0.0

No, Elasticsearch 8.0.0 introduces a number of breaking changes and the solution has not yet being updated to become compatible with it. Of course, it's the plan to adjust the solution to support it at a later time.
