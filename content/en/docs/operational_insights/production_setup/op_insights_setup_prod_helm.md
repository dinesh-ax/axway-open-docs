---
title: Configure a production setup for Helm
linkTitle: Production setup for Helm
weight: 20
date: 2022-07-20
description: Configure a production setup using Helm charts to test Operational Insights in a single instance.
---

The following sections cover the configuration specific for deploying Axway API Management for Elastic solution on a Kubernetes or OpenShift cluster using Helm. The provided Helm chart is extremely flexible and configurable. You can decide which components to deploy, use your own labels, annotations, secrets, and volumes to customize the deployment to your needs.

## Before you start

* Ensure that you have all [prerequisites](/docs/amplify_analytics/op_insights_prerequisites/) in place.
* Ensure that you have applied the [basic configuration](link) for general frameworks.

## Get started

{{< alert title="Note" >}}
Watch [this video](https://youtu.be/w4n9JcBA-X4) to see how to deploy the solution on Kubernetes. The video explains how you could start the solution with minimal setup.
{{< /alert >}}

Nevertheless, you need sufficient resources (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#required-resources>) for it, if you take over the standard resources defined by the values.yaml (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/helm/values.yaml>). In the further course of the document how you can include your own Secrets, Volumes or adjust the required resources.
Create your own `myvalues.yaml` based on the standard values.yaml and configure needed parameters. All of the parameters are explained in detail in the charts values.yaml.

The following represents the most simple `myvalues.yaml` assuming the API Management Platform and Filebeat are running externally to the Kubernetes cluster as indicated in the Helm architecture diagram (**add link**):

```bash
apibuilder4elastic: 
  anmUrl: "https://my-admin-node-manager:8090"
  secrets: 
    apimgrUsername: "apiadmin"
    apimgrPassword: "changeme"
# Enable, if you would like to deploy a new Elasticsearch cluster for the solution
elasticsearch:
  enabled: true
  volumeClaimTemplate:
    accessModes: [ "ReadWriteOnce" ]
    resources:
      requests:
        storage: 1Gi
# Enable, if you would like to deploy Kibana for the solution
kibana:
  enabled: true
```

### Elasticsearch persistent volumes

As Elasticsearch is enabled in the previous example two persistent volumes, one for each Elasticsearch node, are required. So, first, create two persistent volumes.

The following should help you to get started, but these volumes are HostPath volumes pointing to a Worker-Node directory - This is not for production. For production use, use appropriate persistent volumes according to your environment/infrastructure. How to set this up is out of scope for this documentation.
Make sure to create a directory `/tmp/data` on your WorkerNodes and give it permissions for everybody.

```bash
kubectl apply -n apim-elk -f https://raw.githubusercontent.com/Axway-API-Management-Plus/apigateway-openlogging-elk/develop/helm/misc/pv-vol1.yaml
kubectl apply -n apim-elk -f https://raw.githubusercontent.com/Axway-API-Management-Plus/apigateway-openlogging-elk/develop/helm/misc/pv-vol2.yaml
```

### Install the Helm chart

With Elasticsearch volumes and your `myvalues.yaml` file in place, you can start the installation:

```bash
helm install -n apim-elk -f myvalues.yaml axway-elk https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.5.0/helm-chart-apim4elastic-v4.5.0.tgz
```

{{< alert title="Note" >}}
The Helm release name, `axway-elk` is mandatory. For more information, see (FAQ - Why Helm Release-Name axway-elk?)
{{< /alert >}}

To check the status of the deployment, pods, services, and son on, run the following commands:

```bash
# Check the installed release
helm list -n apim-elk
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION   
axway-elk       apim-elk        1               2021-05-03 14:22:08.9325287 +0200 CEST  deployed        apim4elastic-4.2.0              4.2.0

# Check the pods, with Elasticsearch and Kibana enabled
kubectl get pods -n apim-elk
NAME                                                         READY   STATUS    RESTARTS   AGE 
axway-elk-apim4elastic-apibuilder4elastic-65b5d56d77-5hv9z   1/1     Running   1          7h2m
axway-elk-apim4elastic-elasticsearch-0                       1/1     Running   0          7h2m
axway-elk-apim4elastic-elasticsearch-1                       1/1     Running   0          7h2m
axway-elk-apim4elastic-kibana-7c6d4b675f-dnxj7               1/1     Running   0          7h2m
axway-elk-apim4elastic-logstash-0                            1/1     Running   0          7h2m
axway-elk-apim4elastic-memcached-56b7447d9-25xwb             1/1     Running   0          7h2m

# Check deployed services
kubectl -n apim-elk get service
NAME                                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
axway-elk-apim4elastic-apibuilder4elastic           ClusterIP   None            <none>        8443/TCP            7h7m
axway-elk-apim4elastic-elasticsearch                ClusterIP   10.100.85.132   <none>        9200/TCP,9300/TCP   7h7m
axway-elk-apim4elastic-elasticsearch-headless       ClusterIP   None            <none>        9200/TCP,9300/TCP   7h4m
axway-elk-apim4elastic-kibana                       ClusterIP   10.105.84.214   <none>        5601/TCP            7h7m
axway-elk-apim4elastic-logstash                     NodePort    10.103.53.111   <none>        5044:32001/TCP      7h7m
axway-elk-apim4elastic-logstash-headless            ClusterIP   None            <none>        9600/TCP            7h7m
axway-elk-apim4elastic-memcached                    ClusterIP   10.108.48.131   <none>        11211/TCP           7h7m

# Describe a certain POD
kubectl -n apim-elk describe pod axway-elk-apim4elastic-elasticsearch-0

# Get the logs for a POD
kubectl -n apim-elk logs axway-elk-apim4elastic-apibuilder4elastic-65b5d56d77-5hv9z
```

If all configuration has worked and your Ingress configuration is running, you can access the different components on the following host-names:

* `https://kibana.apim4elastic.local`
* `https://apibuilder.apim4elastic.local`
* `https://elasticsearch.apim4elastic.local`

This assumes, that Ingress is configured and DNS-Resolution for apim4elastic.local points to your cluster IP. Of course you can configure different Ingress hostnames. More details is out of scope for this document.

At this point, it is still assumed, that the API Management platform is running externally. Therefore, as the next step, you need to connect one or more Filebeats to Logstash running in Kubernetes.

### Configure Logstash and Filebeat

The communication between Filebeat and Logstash is a persistent TCP connection. This means that once the connection has been etsablished, it will continue to be used for the best possible throughput. If you specify multiple Logstash instances in your Filebeat configuration, then Filebeat will establish multiple persistent connections and uses all of the them for load balancing and failover.

In the case of Kubernetes/OpenShift, multiple Logstash instances are running behind a Kubernetes service, which acts like a load balancer. However, due to the persistent connection, the load balancer/service cannot really distribute the load. Therefore, for high volumes, it is still the better option to let Filebeat do the load balancing.

#### NodePort Service

By default, the Helm chart deploys a NodePort service for Logstash and with that it becomes available on the configured port: 32001 on all nodes of the cluster.
You can now setup the corresponding nodes as Logstash hosts in your Filebeat configuration with Load-Balancing enabled and Filebeat will distribute the Traffic accross the available Logstashes. With that, it works almost the same as with the Docker-Compose deployment, as Filebeat establishes multiple persistent connections.
The following diagram illustrates the approach:

(**diagram**: <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#1-nodeport-service>)

This is an example setup:

```bash
# The service exposing Logstash as a NodePort on 32001
kubectl -n apim-elk get services axway-elk-apim4elastic-logstash -o wide
NAME                              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
axway-elk-apim4elastic-logstash   NodePort   10.110.89.215   <none>        5044:32001/TCP   85m   app=axway-elk-apim4elastic-logstash,chart=logstash,release=axway-elk

# The given NodePort (default 32001) is exposed on all Worker-Nodes:
kubectl get nodes -o wide
NAME                            STATUS   ROLES                  AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION                  CONTAINER-RUNTIME
ip-172-31-51-209.ec2.internal   Ready    <none>                 23h   v1.21.0   172.31.51.209   <none>        Amazon Linux 2   4.14.209-160.339.amzn2.x86_64   docker://19.3.13
ip-172-31-53-214.ec2.internal   Ready    <none>                 23h   v1.21.0   172.31.53.214   <none>        Amazon Linux 2   4.14.193-149.317.amzn2.x86_64   docker://19.3.6
ip-172-31-54-120.ec2.internal   Ready    <none>                 23h   v1.21.0   172.31.54.120   <none>        Amazon Linux 2   4.14.209-160.339.amzn2.x86_64   docker://19.3.13
ip-172-31-61-143.ec2.internal   Ready    control-plane,master   23h   v1.21.0   172.31.61.143   <none>        Amazon Linux 2   4.14.181-142.260.amzn2.x86_64   docker://19.3.6
```

And this would be the belonging configuration for the Filebeat Logstash output

```bash
output.logstash:
  # Based on our tests, the more WorkerNodes you add, the more likely the traffic 
  # is evenly distributed
  hosts: ["172.31.51.209:32001", "172.31.53.214:32001", "172.31.54.120:32001"]
  # Or as part of the .env:
  # LOGSTASH_HOSTS=172.31.51.209:32001,172.31.53.214:32001,172.31.54.120:32001
  worker: 2
  bulk_max_size: 3072
  loadbalance: true
  # Required for the NodePort service approach to give Filebeat a chance to recognize 
  # and use additional Logstash instances that have been provisioned. 
  # 5m was determined to be the best value for the highest possible throughput. 
  # Values lower than 5 minutes may cause an error in Filebeat described here:
  # https://www.elastic.co/guide/en/beats/filebeat/current/publishing-ls-fails-connection-reset-by-peer.html
  ttl: 2m
  # Required to make TTL working
  pipelining: 0
```

The NodePort Service without any Load-Balancer in between is the recommended approach for the best possible throughput. This has been tested with up to 1.000 TPS using 4 Logstash instances and a 5 Node-Elasticsearch cluster.

#### Load Balancer

If you prefer to use a Load-Balancer to have a single entry point it's also possible. You can re-configure the Logstash service from a NodePort to a LoadBalancer and use for instance your Public-Cloud Load-Balancer, from AWS, GCP, etc.
With that kind of setup, care must be taken to ensure that Filebeat is set with an appropriate TTL to improve the load is at least better distributed between the available Logstash instances. (See here for more details elastic/beats#661). However, it's still random to which Logstash instances connections are established. So, you might see situations where Logstash-Instances are on Idle and other under heavy load.

For example:

```bash
output.logstash:
  # Or as part of the .env:
  # LOGSTASH_HOSTS=172.31.51.209:32001,172.31.53.214:32001,172.31.54.120:32001
  hosts: ["logstash.on.load-balancer:5044"]
  worker: 2
  bulk_max_size: 3072
  # This parameter has not effect, as there is only one Logstash host configured
  loadbalance: true
  # The following two parameters drop & re-establish the connection to Logstash every 5 minutes
  # With that, you give the Service/LoadBalancer from time to time the chance to distribute the traffic. 
  # But even with that, it might be the case, that call traffic goes to one Logstash instance.
  # Do not set the ttl less than 1 minute, as it would increase the connection management overhead
  ttl: 2m
  # Required to make TTL working
  pipelining: 0
```

If you would like to read more, <https://discuss.elastic.co/t/filebeat-only-goes-to-one-of-the-logstash-servers-that-is-behind-an-elb/48875/5>.

### Enable user authentication

nabling user authentication in Elasticsearch is quite analogous to the Docker Compose approach. For a newly created Elasticsearch cluster, you generate passwords for the default Elasticsearch users and then store them in your myvalues.yaml or in your own secrets.

Run the following command to generate the passwords for the default users.

```bash
kubectl -n apim-elk exec axway-elk-apim4elastic-elasticsearch-0 -- bin/elasticsearch-setup-passwords auto --batch --url https://localhost:9200
```

This structure shows how to setup the Elasticsearch users in your myvalues.yaml and disable anonymous access.

```bash
apibuilder4elastic:
  secrets:
    elasticsearchUsername: "elastic"
    elasticsearchPassword: "XXXXXXXXXXXXXXXXXXXX"
logstash:
  logstashSecrets:
    # Used to send stack monitoring information
    logstashSystemUsername: "logstash_system"
    logstashSystemPassword: "AAAAAAAAAAAAAAAAAA"
    # Used to send events
    logstashUsername: "elastic"
    logstashPassword: "XXXXXXXXXXXXXXXXXXXX"
kibana:
  kibanaSecrets:
    username: "kibana_system"
    password: "ZZZZZZZZZZZZZZZZZ"
filebeat:
  filebeatSecrets: 
    beatsSystemUsername: "beats_system"
    beatsSystemPassword: "YYYYYYYYYYYYYYYYYYY"
  # Required for the internal stack monitoring to work with Filebeat
  elasticsearchClusterUUID: "YOUR-CLUSTER-UUID-ID"
# Required for the Elasticsearch readiness check, once users have been generated
elasticsearch:
  elasticsearchSecrets: 
    elasticUsername: "elastic"
    elasticPassword: "BBBBBBBBBBBBBBBBBBBBBB"
  anonymous: 
    enabled: false
```

### Customize your Helm chart

To customize the solution according to your needs, you can configure it using your own Secrets, ConfigMaps, etc.

We recommend that you create your own Helm chart that contains all the necessary resources.
You then link your custom resources in your myvalues.yaml for the final deployment of the solution. The following illustrates the recommended approach:

(**diagram**: <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#customize-the-setup>)

#### Create your own Helm-Chart

Customize the generated Helm chart according to your needs and remove stuff that is not needed. Based on a few examples it's explained below how to customize the solution.

```bash
helm create axway-elk-setup
cd axway-elk-setup
```

### Use a Secret for API Manager username and password

The following example explains how you can create a secret, that keeps the API-Manager username and password and use it with API-Builder. The same procedure applies for all confidential information. Please check values.yaml for more details.

## Next steps

After you have tried the solution in a single node elasticsearch scenario, ensure that you have read [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) then you can proceed to setup Operational insights in your production environment.
