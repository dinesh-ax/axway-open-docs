---
title: Configure a setup for production environments
linkTitle: Production setup
weight: 40
no_list: true
date: 2022-07-20
description: Configure a basic setup for Operational Insights to test Elasticsearch in a single instance.
---

This section covers advanced configuration topics that are required for a production environment.

{{< alert title="Note" >}}
It is assumed that you have already familiarized yourself with Operational Insights component by using its [basic setup](/docs/operational_insights/basic_setup/).
{{< /alert >}}

## Securing a kubernetes cluster

For a general overview of how to secure a production cluster please consult the kubernetes documentation [here:](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/). 

In this page we will also highlight 3 specific areas of security - network policies, secrets and encryption in transit

## Kubernetes Networking

In kubernetes, containers run inside pods. A typical pod will run only one container but it is also common to run more than one container in a pod if the other containers are helper or side-car containers.

In a kubernetes cluster, each pod gets its own ip address and this ip address is unique throughout the cluster. Each container within a pod uses the same ip address. There is no isolation between the pods and this means that all pods within a cluster can communicate with each other without Network Address Translation. This applies to pods running in different namespaces and/or on different nodes.

If you want to restrict access from some pods to others - especially in a multi-tenancy cluster - you need to configure network policies

### Network Policies

Network policies are similar to firewalls. With network policies in place you can control traffic flow between pods at the ip address or port level.
This allows control of traffic coming from other pods, other namespaces or from the outside world. This incoming traffic (ingress), which by default is disabled, can be opened up and controlled by use of a network policy. Outgoing traffic from the pods to the outside world (egress) which, by default is enabled, can also be controlled as part of a network policy.

In summary network policies work by:

* Controlling access from pod to pod
* Granting or denying pods access from or to a namespace
* Using IP CIDR blocks to restrict access to pods

### Container Network Interface (CNI) Plugins

Network policies cannot be enforced without a CNI Plugin. If running kubernetes on a managed service such as AKS (Azure Kubernetes Service) you can avail of the built-in Azure CNI. Otherwise you can install a 3rd party plugin such as Calico or Weaveworks.

For more information on kubernetes networking, including sample network policies, please refer to the Kubernetes documentation [here:](https://kubernetes.io/docs/concepts/services-networking/network-policies) or the Calico documentation [here:](https://projectcalico.docs.tigera.io/about/about-network-policy)

### Default Networking in AAOI helm chart

By default the AAOI helm chart does not use network policies but directly enables ingress traffic on the elasticsearch and kibana pods to allow inbound communication to take place. For the other pods ingress is either disabled or not configure which means that no inbound traffic is allowed into these pods.

## Kubernetes Secrets

In the current AAOI solution, passwords such as those for API Manager, kibana, logstash and elasticsearch are stored in clear text in environment variables. This makes them easily accessible to anyone who has access to the kubernetes cluster. 

Kubernetes secrets provide a further layer of security. They are stored as key-value pairs and can be created outside of the application code and referenced from within the code. Secrets are base-64 encoded and so are still accessible to anyone who has the credentials to query elements in the cluster.

In a production environment we recommend that you store your passwords using kubernetes secrets that are backed by a secrets manager such as Hashicorp Vault, AWS Secret Manager or Azure Vault. Access to secrets should also be restricted using RBAC (role-based access control) rules

For more information on kubernetes secrets please consult the kubernetes documentatiion [here:](https://kubernetes.io/docs/concepts/configuration/secret/)

## Elasticsearch encryption in transit

By default all data sent between nodes in the Elasticsearch cluster is sent in clear text. This may be acceptable if the cluster itself is sufficiently secured. However in a production environment we recommend that traffic between nodes is encrypted. 

To do this first locate the file *elasticsearch.yml* and follow the instructions [here:](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html#encrypt-internode-communication) 

If your cluster is running in a managed service such as on Azure Kubernetes Service (AKS) then you will have built-in security options such as enabling TLS which come as part of the service

For more details on encryption in transit see the following docs [Elastic:](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html#security-basic-setup) and [Azure:](https://docs.microsoft.com/en-us/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit)

