---
title: Update Operational Insights in Helm
linkTitle: Update Helm
weight: 210
date: 2022-08-04
description: Instructions on how to update Operational Insights between versions in a Helm chart.
---

<!-- https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#upgrade-the-release -->

If a new Elastic version is installed as a result of the upgrade, it is important that at least 3 Elasticsearch nodes are running. This is the only way to ensure that a new master node can be selected during the upgrade and that updated nodes can join the cluster.

Watch this video to see a demonstration how to update the Elastic-Stack on Kubernetes using HELM: <https://youtu.be/3-7db1eYIc0>

Example how to upgrade an existing release:

```bash
helm upgrade -n apim-elk -f myvalues.yaml axway-elk https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.2.0/helm-chart-apim4elastic-v4.2.0.tgz
```

## Required resources

The following resources were determined to be able to process up to 1,000 TPS in real-time. However, this is the absolute maximum and you should add more capacity if you have constantly 1.000 TPS or above.

If you have fewer transactions per second, then you can make the platform smaller accordingly. Use the [performance information from the classic deployment](/docs/operational_insights/production_setup/op_insights_setup_prod_docker/#configure-the-retention-period) as a reference, since the data is comparable.

| Component                | Requested Memory | Memory limit | Request CPU | CPU Limit | Notes                                |
| ------------------------ | ---------------- | ------------ | ----------- | --------- | ------------------------------------ |
| API-Builder4Elastic 1    | 80Mi             | 150Mi        | 100m        | 100m      | 2 Instances are recommended for HA   |
| API-Builder4Elastic 2    | 80Mi             | 150Mi        | 100m        | 100m      |                                      |
| Logstash incl. Memcached | 6.5Gi            | 6.5Gi        | 2000m       | 4000m     | For 1.000 TPS 4 Logstash instances   |
| Logstash incl. Memcached | 6.5Gi            | 6.5Gi        | 2000m       | 4000m     | are required                         |
| Logstash incl. Memcached | 6.5Gi            | 6.5Gi        | 2000m       | 4000m     | Instance 3                           |
| Logstash incl. Memcached | 6.5Gi            | 6.5Gi        | 2000m       | 4000m     | Instance 4                           |
| Elasticsearch 1          | 14Gi             | 16Gi         | 2000m       | 4000m     | For 1.000 TPS a 5 Node Elasticsearch |
| Elasticsearch 2          | 14Gi             | 16Gi         | 2000m       | 4000m     | cluster is required                  |
| Elasticsearch 3          | 14Gi             | 16Gi         | 2000m       | 4000m     | Node 3                               |
| Elasticsearch 4          | 14Gi             | 16Gi         | 2000m       | 4000m     | Node 4                               |
| Elasticsearch 5          | 14Gi             | 16Gi         | 2000m       | 4000m     | Node 5                               |
| Kibana                   | 300Mi            | 500Mi        | 500m        | 1000m     |                                      |
| ------------------------ | ---------------- | ------------ | ----------- | --------- | ------------------------------------ |
| Total                    | ~97Gi            | ~140Gi       | ~19         | ~38       | Supposed to handle up to 1.000 TPS   |

Filebeat is not yet tested as part of the Kubernetes deployment. Memory is ap. between 150Mi and 250Mi. CPU between 500m and 1000m.

For Elasticsearch the maximum number of file descriptors must be at least 65536. (See for instance, <https://documentation.sisense.com/latest/linux/dockerlimits.htm>)

## Helm update FAQ

### Why Helm Release-Name axway-elk?

The release name must currently: `axway-elk`, because many resources, like Services, ConfigMaps or Secrets are created with it and referenced in the standard `values.yaml`. An example is the Elasticsearch-Service: `axway-elk-apim4elastic-elasticsearch`, which is for instance used for the standard elasticsearchHosts: "<https://axway-elk-apim4elastic-elasticsearch:9200>". This restriction may be changed in a later release to get more flexibility.

### Can I use Auto-Scaling?

Apart from API-Builder4Elastic, this is unfortunately not possible. Elasticsearch automatically stores the data on multiple cluster nodes and would have to rebalance the cluster again and again when adding or removing nodes/pods. Logstash also does not offer the option of autoscaling. However you can change the number of PODs for, lets say, Elasticsearch, Logstash at any time. The condition is that they do not change within a short time, otherwise no advantage is gained.

### Can I easily add or remove instances?

Yes, they can change the number of nodes via the scaling function or via their values.yaml using replicas. This works for API-Builder4Elastic, Logstash, Elasticsearch and Kibana.
For Elasticsearch, make sure the cluster is in Green state before removing a node to avoid data loss. If you change the Logstash instances, it is recommended to restart all Filebeat instances afterwards. The reason for this are the persistent connections. You may also need to adjust the filebeat configuration.

### How can I increase the disk space for Elasticsearch?

If Elasticsearch is running in Kubernetes as a StatefulSet, you can increase the disk space as follows.

1. Make sure that the StorageClass you are using supports volumeExpansion. allowVolumeExpansion: true
2. Edit existing PVC for already running PoDs and increase the capacity
3. Update the VolumeClaimTemplate in your Helm values
    ```bash
    volumeClaimTemplate:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: gp2
      resources:
        requests:
          storage: 100Gi
    ```
4. Now delete the StatefulSet without deleting the pods.
    ```bash
    kubectl -n apim delete sts axway-elk-apim4elastic-elasticsearch --cascade=orphan
    ```
5. Perform a helm upgrade to reinstall the modified StatefulSet.
6. Redeploy the PODs of the stateful set.
    ```bash
    kubectl -n apim rollout restart sts axway-elk-apim4elastic-elasticsearch
    ```