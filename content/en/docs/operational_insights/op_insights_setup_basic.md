---
title: Configure Elasticsearch for a single instance
linkTitle: Basic setup
weight: 30
date: 2022-06-09
description: Configure a basic setup for Operational Insights to test Elasticsearch in a single instance.
---

The basic setup explains the individual components, how they can be deployed and work together. This is a fully functional configuration to get you started as fast and as easy as possible, and which allows you to try out Operational Insights and gain experience, and be prepared for the deployment in your production environment.

{{< alert title="Note">}}This configuration is recommended for development environments.{{< /alert >}}

After completing the basic setup you will have a single node Elasticsearch instance including multiple Kibana dashboards. At this point, API Gateway Manager is not yet configured, therefore this single instance receives data from 1 to many API Gateways via Filebeat, Logstash, and API Builder, and is accessible via the Traffic Monitor.

You can also import and use the sample Kibana dashboard or create your own visualizations.

This basic setup uses minimal parameters to run and test Operational Insights on a machine with at least 16 GB of RAM including the API Management platform (for example, like the Axway internal API Management reference environment). Therefore, this is not suitable for a production environment without further configuration.

For a production environment, check the parameters mentioned in the [env-sample](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/7d91baaf8009ceb09ac9f3889752912b17b83736/env-sample#L604-L648) and set them if necessary. (**TO BE REVIEWED**)

You can deploy Operational Insights using a Helm chart or Docker Compose. This page is organized in three main sections describing the common setup for both Helm and Docker Compose options, and two specific sections for each Helm and Docker Compose.

## Setup for Helm and Docker Compose

The following sections are common for both Helm and Docker Compose options.

### Enable Open Traffic Event Log

You must [enable Open-Traffic Event log](/docs/apim_administration/apigtw_admin/admin_open_logging/#configure-open-traffic-event-logging) for your API Gateway instances to allow your log files to be created by default at `apigateway/logs/opentraffic`. This location is required for when configuring Filebeat later.

To avoid data loss, it is strongly recommended to increase the disk space for the Open Traffic logs from 1 GB to at least 8 GB, particularly if you have a lot of traffic. For example, if you have 100 TPS on one API Gateway instance, depending on your custom policies, the oldest log file will be deleted after approximately 30 minutes with only 1 GB OpenTraffic log configured. If for any reason Filebeat and Logstash are not running to process events for more than 15-20 minutes, you will have a loss of data as it also takes some time to catch up.

### Install Kibana dashboards

To install the Kibana dashboards, use the Kibana Menu **Stack Management > Saved Objects > Import saved objects** and select the following file from the release package, `kibana/dashboards/7/*.ndjson`.

In the **Import options** window, select **Automatically overwrite conflicts**. This is the default setting, and you must always select it regardless of whether you are installing the dashboards for the first time or after an update.

You can create customized visualizations and dashboards, but do not change the existing ones as they will be overwritten with the next update. If you have created your own visualizations and dashboards, they will not be changed by the import.

### Setup Admin Node Manager

Watch this video for an overview, [Traffic-Monitor and Kibana Dashboard](https://youtu.be/OZ0RNnqE6hs).

Configure Admin Node Manager to render log data provided by Elasticsearch instead of the individual API Gateway instances. The Admin Node Manager API listens to port 8090 for administrative traffic by default and itt is responsible for serving the Traffic Monitor. You must now configured node manager to use the API Builder REST API instead.

To configure the node manager to render log data from Elasticsearch, follow these steps:

1. Open the node manager configuration in Policy Studio. For more information, see [Authentication and RBAC with Active Directory](/docs/apim_administration/apigtw_admin/general_rbac_ad_ldap/#use-the-ldap-policy-to-protect-management-services).
2. Import the provided policy fragment (`nodemanager/policy-use-elasticsearch-api-7.7.0.xml`) from the release package you have downloaded. This imports the policy, "Use Elasticsearch API".
    Do not use the XML file downloaded directly from the GitHub project as it might contain different certificates.
3. [Update main policy](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/nodemanager) by inserting the "Use Elasticsearch API" policy as a callback policy (see filter, **Shortcut filter**) into the main policy, **Protect Management Interfaces**, and wire it like shown in the following image.
    **IMAGE**
4. Disable the audit log for **Failure** transactions to avoid not needed log messages in the node manager trace file.
5. Open the `apigateway-install-dir>/apigateway/conf/envSettings.props` file and add the following new environment variable, `env.API_BUILDER_URL=https://apibuilder4elastic:8443`.
6. If you are using [multiple regions](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#different-topologiesdomains), you must configure the appropriate region to restrict the data of node manager to the correct regional data, for example, `env.REGION=US`.
7. Copy the new configuration from the Policy Studio project folder (path on Linux, `/home/<user>/apiprojects/\<project-name\>`) back to the Admin Node Manager folder (`\<install-dir\>/apigateway/conf/fed`).
8. Restart the Admin Node Manager.

### Traffic Monitor view for API Manager users

Watch this video for an overview, [Traffic Monitor for API Manager users](https://youtu.be/X08bQPC1sc4).

In larger companies hundreds of API service providers are using the API Manager or the APIM-CLI to register their own services and APIs. While the service providers require access to the Traffic Monitor to monitor their own APIs independently, during registration, the corresponding APIs are assigned to API Manager organizations, which logically split them up, but, the standard traffic monitor does not know the organization concept and therefore, cannot restrict the view for a user based on the organization of an API.

Operational Insights solves this problem by storing the API transactions in Elasticsearch with the appropriate organization. Then, the API organization is used when reading the traffic data from Elasticsearch according to the following table.

| API Gateway Manager | API Manager | Restriction                       | Comment     |
|---------------------|-------------|-----------------------------------|-------------|
| Administrator       | N/A         | Unrestricted access               | A Gateway Manager user is considered as an Admin when they own the permission, `adminusers_modify`.  |
| Operator            | API Admin   | All APIs having a Service-Context | By default, each API processed by the API Manager has a Service-Context. Pure Gateway APIs, for example, `. /healthcheck`, will not be visible. |
| Operator            | Org Admin   | APIs of its own organization      | Such a user will only see the APIs that belong to the same organization as himself.  |
| Operator            | User        | APIs of its own organization      | The same rules apply as for the Org-Admin           |

#### Setup API Manager user in API Gateway Manager

To give API Manager users an restricted access to the API Gateway Traffic-Monitor, the user must be configured in the API-Gateway-Manager with the same login name as in the API-Manager. Here, for example, an LDAP connection can be a simplification.
Additionally you need to know, that by default, all API-Gateway Traffic-Monitor users get unrestricted access only if one of their roles includes the permission: `adminusers_modify`. But typically, only a full API-Administrator has this right and therefore only these users can see the entire traffic. All other users get a restricted view of the API traffic and will be authorized according to the configured authorization configuration described below.
However, with the parameter: UNRESTRICTED_PERMISSIONS you can configure which right(s) a user must have to get unrestricted access.

#### Customize user authorization

By default, the organization(s) of the API-Manager user is used for authorization to the Traffic-Monitor. This means that the user only sees traffic from his the organization(s) he belongs in API-Manager. From a technical point of view, an additional filter clause is added to the Elasticsearch query, which results in a restricted result set. An example: `{ term: { "serviceContext.apiOrg": "Org-A" }}`.

Since version 2.0.0, it is alternatively possible to use an external HTTP service for authorization instead of the API Manager organizations, to restrict the Elasticsearch result based on other criterias.

To customize user authorization, you need to configure an appropriate configuration file as described here:

```bash
# Copy the provided example 
cp ./config/authorization-config-sample.js ./config/my-authorization-config.js
# Customize your configuration file as needed
vi ./config/my-authorization-config.js
# Setup your .env file to use your authorization config file
vi .env
AUTHZ_CONFIG=./config/my-authorization-config.js
# Recreate the API-Builder container
docker stop apibuilder4elastic
docker-compose up -d
```

In this configuration, which also contains corresponding Javascript code, necessary parameters and code is stored, for example to parse the response and to adjust the Elasticsearch query. The example: <https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/config/authorization-config-sample.js> contains all required documentation.

Once this configuration is stored, the API Manager Organization based authorization will be replaced.

Note:

* Besides the API-Manager Organization autorization only `externalHTTP` is currently supported.
* Only 1 authorization method can be enabled
* It is also possible to disable user authorization completely. To do this, set the parameter: `enableUserAuthorization`: false.
* If you have further use-cases please create an issue describing the use-case/requirements.

## Setup for Helm

The following sections cover the configuration specific for deploying Axway API Management for Elastic solution on a Kubernetes or OpenShift cluster using Helm. The provided Helm chart is extremely flexible and configurable. You can decide which components to deploy, use your own labels, annotations, secrets, and volumes to customize the deployment to your needs.

Before you start, ensure that you have all [prerequisites](/docs/amplify_analytics/op_insights_prerequisites/) in place.

Watch [this video](https://youtu.be/w4n9JcBA-X4) to see how to deploy the solution on Kubernetes.

Here it is explained how you could start the solution with minimal setup. Nevertheless, you need sufficient resources (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/helm#required-resources>) for it, if you take over the standard resources defined by the values.yaml (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/helm/values.yaml>). In the further course of the document how you can include your own Secrets, Volumes or adjust the required resources.
Create your own `myvalues.yaml` based on the standard values.yaml and configure needed parameters. All of the parameters are explained in detail in the charts values.yaml.

The following represents the most simple`myvalues.yaml` assuming the API Management Platform and Filebeat are running externally to the Kubernetes cluster as indicated in the Helm architecture diagram (**add link**):

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

**to be continued...**

## Setup for Docker Compose

The following sections cover the configuration specific for using Docker Compose (virtual machines).

### Download and extract the release package

Download the relevant ELK package.

The community release always reflects the state of development. Check the [changelog](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/blob/develop/CHANGELOG.md) to ensure that you have select the correct version. For more information, see [Is this component officially supported by Axway](/docs/amplify_analytics/op_insights_faq/#is-this-component-officially-supported-by-axway).

We recommend that you run Operational Insights on different machines. Therefore, download and unpack the release package on each machine.

**Axway supported version**:

```bash
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.2.0/axway-apim-elk-v4.2.0.tar.gz -O - | tar -xvz
```

**Community version**:

```bash
wget --no-check-certificate https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/releases/download/v4.5.0/axway-apim-elk-v4.5.0.tar.gz -O - | tar -xvz
```

To simplify updates, it is recommended to create a symlink folder and rename the provided file `env-sample` to `.env` in each machine as follows:

```bash
ln -s axway-apim-elk-v1.0.0 axway-apim-elk
cd axway-apim-elk-v1.0.0
cp env-sample .env
```

You must store the `.env` file as a central configuration file in a version management system.

### Setup Elasticsearch

Watch this video for a demonstration, [Setup Single Node Elasticsearch cluster](https://youtu.be/x-OdAdV2N7I).

If you are using an existing Elasticsearch cluster, you can skip this section and go straight to sections Logstash/Memcached and APIBuilder4Elastic.

Open the .env file and configure the ELASTICSEARCH_HOSTS. At this point, configure only one Elasticsearch node. You can start with a single node and add more nodes later.

This URL is used by all Elasticsearch clients (Logstash, API-Builder, Filebeat) of the solution to establish communication. If you use an external Elasticsearch cluster, specify the node(s) that are given to you. Note that the hostnames must be resolvable within the docker containers. Some parameters to consider to change before starting the cluster:

```bash
ELASTICSEARCH_HOSTS=https://my-elasticsearch-host.com:9200
ELASTICSEARCH_CLUSTERNAME=axway-apim-elasticsearch-prod
ES_JAVA_OPTS="-Xms8g -Xmx8g"
```

Run the following command to initialize a new Elasticsearch Cluster which is going through an appropriate bootstrapping. Later you can add more nodes to this single node cluster. Please do not use the init extension when restarting the node:

```bash
docker-compose --env-file .env -f elasticsearch/docker-compose.es01.yml -f elasticsearch/docker-compose.es01init.yml up -d
```

Wait until the cluster has started and then call the following URL:

```bash
curl -k GET https://my-elasticsearch-host.com:9200
{
  "name" : "elasticsearch1",
  "cluster_name" : "axway-apim-elasticsearch",
  "cluster_uuid" : "nCFt9WhpQr6JSOVY_h48gg",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "d34da0ea4a966c4e49417f2da2f244e3e97b4e6e",
    "build_date" : "2020-09-23T00:45:33.626720Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

At this point you can already add the cluster UUID to the .env (ELASTICSEARCH_CLUSTER_UUID) file. With that, the Single-Node Elasticsearch Cluster is up and running.

### Setup Kibana

Watch this video for a demonstration, [Setup Kibana](https://youtu.be/aLODAuXDMzY).

If you are using an existing Elasticsearch cluster, you can skip this section and go straight to sections Logstash/Memcached and APIBuilder4Elastic.

For Kibana all parameters are already stored in the .env file. Start Kibana with the following command:

```bash
docker-compose --env-file .env -f kibana/docker-compose.kibana.yml up -d
```

You can address Kibana at the following URL Currently no user login is required.

```
https://my-kibana-host:5601
```

If Kibana doesn't start (>3-4 minutes) or doesn't report to be ready, use docker logs Kibana to check for errors.

### Setup Logstash, API-Builder, and Memcached

Watch this video for a demonstration, [Setup Logstash and API-Builder](https://youtu.be/lnSjF2tUS8Y).

It is recommended to deploy these components on one machine, so they are in a common Docker-Compose file and share the same network. Furthermore, a low latency between these components is beneficial. This allows you to use the default values for Memcached and API Builder. Therefore you only need to specify where the Admin-Node-Manager or the API manager can be found for this step. If necessary you have to specify an API-Manager admin user.

```bash
ADMIN_NODE_MANAGER=https://my-admin-node-manager:8090
API_MANAGER_USERNAME=elkAdmin
API_MANAGER_PASSWORD=elastic
```

If you are using an existing Elasticsearch cluster and have therefore skipped the previous two sections, please also configure the Elasticsearch hosts, any necessary users (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#activate-user-authentication>) and the Elasticsearch server (<https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#custom-certificates>) CA here.

```bash
ELASTICSEARCH_HOSTS=https://my-existing-elasticsearch-host1.com:9200, https://my-existing-elasticsearch-host2.com:9200, https://my-existing-elasticsearch-host3.com:9200
```

To start all three components the main Docker-Compose file is used:

```bash
docker-compose up -d
```

Check that the docker containers for Logstash, API Builder and Memached are running.

```bash
[ec2-user@ip-172-31-61-59 axway-apim-elk-v1.0.0]$ docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS                 PORTS                              NAMES
d1fcd2eeab4e        docker.elastic.co/logstash/logstash:7.12.2  "/usr/share/logstash…"   4 hours ago         Up 4 hours             0.0.0.0:5044->5044/tcp, 9600/tcp   logstash
4ce446cafda1        cwiechmann/apibuilder4elastic:v1.0.0        "docker-entrypoint.s…"   4 hours ago         Up 4 hours (healthy)   0.0.0.0:8443->8443/tcp             apibuilder4elastic
d672f2983c86        memcached:1.6.6-alpine                      "docker-entrypoint.s…"   4 hours ago         Up 4 hours             11211/tcp                          memcached
```

It may take some time (2-3 minutes) until Logstash is finally started.

```bash
docker logs logstash
Pipelines running {:count=>6, :running_pipelines=>[:".monitoring-logstash", :BeatsInput, :Events, :DomainAudit, :TraceMessages, :OpenTraffic], :non_running_pipelines=>[]}
Successfully started Logstash API endpoint {:port=>9600}
```

Notes:

* The Logstash API endpoint (9600) is not exposed outside of the docker container.
* Logstash is configured not to create indexes or index templates in Elasticsearch. These will be installed later by the API Builder application when the first events are received. The reason is that they may need to be created according to the region.

### Setup Filebeat

Watch this video for a demonstration, [Setup Filebeat](https://youtu.be/h0AdztZ2bSE).

Finally Filebeat must be configured and started. You can start Filebeat as Docker-Container using the Docker-Compose files and mount the corresponding directories into the container. Alternatively you can install Filebeat natively on the API gateway and configure it accordingly. It is important that the filebeat/filebeat.yml file is used as base. This file contains instructions which control the logstash pipelines.

The following instructions assume that you set up Filebeat based on filebeat/docker-compose.filebeat.yml.

This is an important step, as otherwise Filebeat will not see and send any Event data. Add the following configuration. At this point you can already configure the filebeat instances with a name/region if you like.

```bash
APIGATEWAY_LOGS_FOLDER=/opt/Axway/APIM/apigateway/logs/opentraffic
APIGATEWAY_TRACES_FOLDER=/opt/Axway/APIM/apigateway/groups/group-2/instance-1/trace
APIGATEWAY_EVENTS_FOLDER=/home/localuser/Axway-x.y.z/apigateway/events
APIGATEWAY_AUDITLOGS_FOLDER=/home/localuser/Axway-x.y.z/apigateway/logs
GATEWAY_NAME=API-Gateway 3
GATEWAY_REGION=US
```

Audit-Logs are optional. If you don't want them indexed just point to an invalid folder. You can find more information for each parameter in the env-sample.

To start Filebeat:

```bash
docker-compose --env-file .env -f filebeat/docker-compose.filebeat.yml up -d
```

Use docker logs filebeat to check that no error is displayed. If everything is ok, you should not see anything.
Check in Kibana (Menu --> Management --> Stack Management --> Index Management) that the indexes are filled with data.

If you encounter issues please see the Troubleshooting section.

## Next steps

After you have tried the solution in a single node elasticsearch scenario, ensure that you have read [size your infrastructure](/docs/amplify_analytics/op_insights_infra_size) then you can proceed to setup Operational insights in your production environment.