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

## Kubernetes Networking

In kubernetes, containers are run inside pods. A typical pod may run more than one container but it is common to configure a pod to run only one container, unless the second is a helper or side-car container.

In a kubernetes cluster, each pod gets its own ip address and this ip address is unique throughout the cluster. There is no isolation between the pods and this means that all pods within a cluster can communicate with each other without Network Address Translation. This applies to pods running in different namespaces and/or on different nodes.

If you want to restrict access from some pods to others - especially in a multi-tenancy cluster - then you need to configure network policies

### Network Policies

Network policies are similar to firewalls. With network policies in place you can control traffic flow between pods at the ip address or port level (OSI Layer 3/4).
This allows control of traffic coming from other pods, other namespaces or from the outside world. This incoming traffic (ingress) can be controlled by use of a network policy. Outgoing traffic from the pods to the outside world (egress) which, by default is allowed can also be controlled as part of a network policy.

There are three kinds of policies that can be applied:

* Drop communication by default
* Allow connection to an application from specific namespaces
* Allow connection to pods from certain applications

Different policies can apply to different pods in your application. For more information refer to kubernetes documentation [here:](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

### Network Policy Agents

In order for a network policy to be enforced we need a Network Policy Agent such as Calico or Weaveworks to be installed. Note: on some managed kubernetes services such as AKS (Azure Kubernetes Service) you can avail of the built-in Azure Network Policy Manager if you don't wish to install a 3rd party agent such as Calico.

### Default Networking in AAOI helm chart

By default the AAOI helm chart does not use network policies but directly enables ingress traffic on the elasticsearch and kibana pods to allow inbound communication to take place. For the other pods ingress is either disabled or not configure which means that no inbound traffic is allowed into these pods.

## Kubernetes Secrets

In the current AAOI solution, passwords such as those for API Manager, kibana, logstash and elasticsearch are stored in clear text in environment variables. This makes them easily accessible to anyone who has access to the kubernetes cluster. 

Kubernetes secrets provide a further layer of security. They are stored as key-value pairs and can be created outside of the application code and referred to from within the code. The secrets are base-64 encoded and so are still accessible to anyone who has the credentials to query elements in the cluster.

In a production environment we recommend that you store your passwords using kubernetes secrets that are backed by a secrets manager such as Hashicorp Vault, AWS Secret Manager or Azure Vault. Access to secrets should also be restricted using RBAC (role-based access control) rules

For more information on kubernetes secrets please consult the kubernetes documentatiion [here:](https://kubernetes.io/docs/concepts/configuration/secret/)
