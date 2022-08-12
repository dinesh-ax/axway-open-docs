---
title: Update Operational Insights in Docker Compose
linkTitle: Update in Docker Compose
weight: 200
date: 2022-07-28
description: Instructions on how to update Operational Insights between versions in a Docker Compose environment.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/UPDATE.md -->

The basic principle of Operational Insights is that no files delivered with a release are changed. This is the best way to ensure the updated without any problems. Only your `*.env` file must be carried over from one release to the next. It is not planned to backport bugfixes (besides for the Axway Supported version) or enhancements in older releases.

This section describes how to update based on Docker Compose deployment. If you have deployed Operational Insights on a Kubernetes cluster using Helm, see [Upgrade Helm](/docs/operational_insights/op_insights_updatehelm/).

To see a table with the list of changed components, see [Release history - Changed components](/docs/operational_insights/op_insights_release_history/).

## Zero downtime upgrade

With the following steps you can update the solution without downtime. Of course, this requires that all components (Logstash, API Builder, Memcached) are running at least 2x and configured accordingly. So all filebeats have to communicate with all logstash hosts.

## About components update

The core component is the API Builder application which provides the information about the necessary configuration. In principle, it contains the desired or necessary state suitable for the version, especially about the Elasticsearch configuration, such as index templates, ILM policies, etc. If the version is updated, the API builder checks the current configuration in Elasticsearch and adjusts it if necessary to fit the corresponding version. This includes necessary changes for bug fixes or enhancements.

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#updates -->

All components of Operational Insights ecosystem play together and only work if they are from the same release.Operational Insights checks if, for example, the index templates have the required version.

It is strongly discouraged to make changes in any files of the project, except the `.env` file and the `config` folder. This is the only way to easily update from one version to the next. If you encounter a problem or need a feature, please open an issue that can be integrated directly into the solution.

Of course you are welcome to create your own Kibana dashboards or clone and customize existing ones. However, if you need to change files, it is recommended to make this change automatically and repeatable (for example, <https://www.ansible.com>).

### Upgrade approach

* Load and unpack the current or desired release. It is recommended to unpack it next to the existing release.
* Copy your existing .env file from the current installation.
    * We recommend you to use a symlink to a central `.env` file, which should also be versioned if necessary. It is pointed out in this document, if parameters have changed or new ones have been added.
* Enure that you carry over your own certificates into the new release.
* Depending on which components have changed,
    * These containers must be stopped and then restarted based on the new release with `docker-compose up -d`, for example, if no change is noted in logstash, then this component can continue to run.
* API Builder delivers the configuration for Elasticsearch as well.
    * If API Builder is updated, then it checks on restart and periodically if the Elasticsearch configuration is correct. In addition, API builder checks whether the filebeat and logstash configuration corresponds to the expected version.

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/UPDATE.md#release-history---changed-components -->

## Upgrade components

The following example shows how to load/unpack release 4.0.3 and apply your current configuration from, for example, version 3.2.0. Regardless of which components have changed, you should install the same release package and take over your configuration on all machines to avoid confusion.

```bash
# Perform these steps on all belonging machines
# Get and extract the release package
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.0.3/axway-apim-elk-v4.0.3.tar.gz -O - | tar -xvz
# The following directory has been created
cd axway-apim-elk-v4.0.3
# Take over existing configuration from the previous version
cp ~/axway-apim-elk-v3.2.0/.env .
# Also copy existing certificates, scripts, extra config-files from the previous version
cp ~/axway-apim-elk-v3.2.0/config/all-my-custom-certificates ./config
```

You can then update each component as described below.

### API-Builder/Logstash/Memcached

If Filebeat should be updated with one of the releases that are between the current one and the new one, then please update Filebeat in advance.

API Builder, Logstash and Memcache work as a tight unit and should be stopped, updated together if possible. Please note, that changes to Logstash do not mean that the Logstash version has changed, but that the Logstash pipeline configuration has changed. To activate these changes the new release must be used to start Logstash.

The following steps illustrates an update to version 3.2.0:

```
docker-compose stop
docker-compose up -d
```

Repeat these steps on all machines running Logstash/API-Builder/Memcache.

If you have deployed the solution on a Kubernetes-Cluster using Helm read [Configure a basic setup for Helm charts](/docs/operational_insights/basic_setup/op_insights_setup_basic_helm/) for more information.

### Filebeat

If Filebeat changes with a version, you must update the corresponding configuration on all your API Gateway instances. It is recommended to update Filebeat as the first component, because the Filebeat configuration version is checked by the API-Builder application. If it does not match, Logstash will exit with an error message. Even if you run Filebeat as a native service, you have to copy the configuration (filebeat/filebeat.yml) from the release into your configuration.

The following steps illustrates an update to version 3.2.0 using the docker-compose approach:

```bash
docker stop filebeat
docker-compose --env-file .env -f filebeat/docker-compose.filebeat.yml up -d
```

### ANM config

Please follow the instructions to setup the Admin Node Manager (**placeholder: add link**) based on the most recent Policy-Fragment shipped with the release. Please make sure that they take their own certificates with them.

### Dashboards

Please follow the instructions to import Kibana-Dashboards (**placeholder: add link**) based on the most recent Dashboards shipped with the release. Select that existing objects are overwritten.

### Parameters

Sometimes it may be necessary to include newly introduced parameters in your .env file or you may want to use optional parameters. To do this, use the supplied env-sample as a reference and copy the desired parameters. Please check the changelog (**placeholder: add link**) which new parameters have been added.

### Elastic Config

Whether the Elastic configuration has changed is for information only and does not require any steps during the update.
The required Elasticsearch configuration is built into the API Builder application and Elasticsearch will be updated accordingly if necessary.
This includes the following components:

* Index configuration
* Index templates
* Index lifecycle policies
* Transform jobs

You can find the current configuration here: elasticsearch_config: <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/apibuilder4elastic/elasticsearch_config> (**placeholder: these are json files. How to make them available**)

## Update Elastic Stack version

Operational Insights ships the latest available Elastic version with new releases. However, this does not force you to update to the appropriate Elastic version with each update. So, for example, if version 3.4.0 ships with Elastic version 7.14.0, you can still stay on version 7.12.1. Thus, it can be concluded that the update of the Elastic stack can be done independently of the releases.

### Elasticsearch nodes required

Before proceeding with the upgrade, make sure that your Elasticsearch cluster consists of at least 3 nodes with the same version. For example 3 Elasticsearch nodes running on two machines is perfectly fine for this.
There are 3 Elasticsearch nodes required, as there must always be a master node in the cluster. If this master node is stopped, a quorum of remaining cluster nodes must still be running to elect a new master, otherwise an upgraded Elasticsearch node cannot join the cluster. For more information, see [Quorum-based decision making](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-quorums.html)

See [Setup Elasticsearch multi-node](/docs/operational_insights/production_setup/op_insights_setup_prod_docker/#setup-elasticsearch-multi-node) to learn about adding additional cluster nodes. After the upgrade, you can remove the third cluster node if necessary. Before proceeding, make sure that Elasticsearch is in the Green state.

Watch this video to see a demonstration how to update the Elastic-Stack: <https://youtu.be/Oht_Xnzurok>

### Upgrade steps

When you have a three node Elasticsearch-Cluster up and running, follow these steps to update the Elastic version.

#### 1. Update the .env file

* Open your `.env` file and change the parameter: `ELASTIC_VERSION` to the version as specified in the release or the version you would like to use
    * Make sure that the `.env` file contains the correct/same version on all machines
* To avoid any downtime, double check all Elasticsearch clients (API-Builder, Logstash, Filebeat) using the ELASTICSEARCH_HOSTS have multiple or all Elasticsearch nodes configured so that they can fail over

#### 2. Update Elasticsearch cluster

Updating the Elasticsearch cluster happens one node after next. Before you star, ensure the following:

* You have three Elasticsearch nodes.
* Before proceeding with the next node it is strongly recommended to validate a new master has been elected.
* The remaining nodes have enough disk space to take over the shards from the node to be upgraded.

Follow these steps to update your Elasticsearch cluster:

1. On the Elasticsearch node you would like to update, navigate into your ELK-Solution directory

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop the existing Elasticsearch container you would like to update

   ```bash
   docker stop elasticsearch1
   ```

3. Start a new Elasticsearch node (in this case Elasticsearch-Node-1), which will create a new container based on the version you configured in your .env file:

    ```bash
    docker-compose --env-file .env -f elasticsearch/docker-compose.es01.yml up -d
    ```

Repeat these steps on the remaining Eleasticsearch nodes, but make sure a new Elasticsearch-Master has been elected.

#### 3. Update Kibana

It is recommended to run the entire Elastic stack with the same version, so Kibana should/must be updated as well. To update Kibana you need to perform the following steps after adjusting the ELASTIC_VERSION accordingly. Kibana must be updated as it otherwise is no longer compatible with the Elasticsearch version.

Follow these steps to update Kibana:

1. On the Kibana node you would like to update, navigate to the directory where you installed Operational Insights:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop the existing Kibana container:

    ```bash
    docker stop kibana
    ```

3. Start a new Kibana container with the configured ELASTIC_VERSION in your .env file:

    ```bash
    docker-compose --env-file .env -f kibana/docker-compose.kibana.yml up -d
    ````

#### 4. Update Logstash

It is recommended to run the entire Elastic stack with the same version, so Logstash must be updated as well. Same procedure as for Kibana but repeat this on all Logstash nodes.

Follow these steps to update Logstash:

1. On the Logstash node you would like to update, navigate to the directory where you installed Operational Insights:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop the existing Logstash container:

    ```bash
    docker stop logstash
    ```

3. Start a new Logstash with the configured ELASTIC_VERSION in your .env file:

    ```bash
    docker-compose up -d
    ```

#### 5. Update Filebeat

It is recommended to run the entire Elastic stack with the same version, so Filebeat should/must be updated as well. Same procedure as for Kibana and Logstash but repeat this on all Filebeat nodes. Of course, these steps are only valid if you run Filebeat as part of the Docker-Compose approach.

Follow these steps to update Filebeat:

1. On the Filebeat node you would like to update, navigate into your ELK-Solution directory:

    ```bash
    cd axway-apim-elk-v4.0.3
    ```

2. Stop existing filebeat container

    ```bash
    docker stop filebeat
    ````

3. Start a new Filebeat with the configured ELASTIC_VERSION in your .env file

    ```bash
    docker-compose --env-file .env -f filebeat/docker-compose.filebeat.yml up -d
    ```

## FAQ

Thi section lists common questions around updating Operational Insights deployed in a Docker Compose environment.

### Do I need to update the Elasticsearch version?

The solution ships a defined Elasticsearch version with each release, which is used by all components of the Elastic stack (Filebeat, Kibana, ...). If nothing else is specified, then you can stay on the Elastic Stack version defined in your `.env` file.

### Can I skip several versions for an upgrade?

Yes, you can. Please note which components have been updated between the current and the new version.

### Can I downgrade back to a previous version?

No, you cannot downgrade from a newer version to an older one. You will see the following error message:

```none
cannot downgrade a node from version [7.16.1] to version [7.15.2]
```

Note that as soon as a newer Elasticsearch node was started, the data stored on the volume is already updated, which makes it impossible to start an older version.
