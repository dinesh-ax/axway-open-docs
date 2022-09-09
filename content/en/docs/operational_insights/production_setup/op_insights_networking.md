---
title: Kubernetes Networking
linkTitle: Networking
weight: 10
date: 2022-09-08
nolist: true
description: How to set up kubernetes network policies
---

In kubernetes, containers run inside pods. A typical pod will run only one container but it is also common to run more than one container in a pod if the other containers are helper or side-car containers.

In a kubernetes cluster, each pod gets its own ip address and this ip address is unique throughout the cluster. Each container within a pod uses the same ip address. There is no isolation between the pods and this means that all pods within a cluster can communicate with each other without Network Address Translation. This applies to pods running in different namespaces and/or on different nodes.

If you want to restrict access from some pods to others - especially in a multi-tenancy cluster - you need to configure network policies

#### Network Policies

Network policies are similar to firewalls. With network policies in place you can control traffic flow between pods at the ip address or port level.
This allows control of traffic coming from other pods, other namespaces or from the outside world. This incoming traffic (ingress), which by default is disabled, can be opened up and controlled by use of a network policy. Outgoing traffic from the pods to the outside world (egress) which, by default is enabled, can also be controlled as part of a network policy.

In summary network policies work by:

* Controlling access from pod to pod
* Granting or denying pods access from or to a namespace
* Using IP CIDR blocks to restrict access to pods

#### Container Network Interface (CNI) Plugins

Network policies cannot be enforced without a CNI Plugin. If running kubernetes on a managed service such as AKS (Azure Kubernetes Service) you can avail of the built-in Azure CNI. Otherwise you can install a 3rd party plugin such as Calico or Weaveworks.

For more information on kubernetes networking, including sample network policies, please refer to the Kubernetes documentation [here:](https://kubernetes.io/docs/concepts/services-networking/network-policies) or the Calico documentation [here:](https://projectcalico.docs.tigera.io/about/about-network-policy)

#### Default Networking in AAOI helm chart

By default the AAOI helm chart does not use network policies but directly enables ingress traffic on the elasticsearch and kibana pods to allow inbound communication to take place. For the other pods ingress is either disabled or not configure which means that no inbound traffic is allowed into these pods.

