---
layout: post
title: "Schedulers, Part 2: Kubernetes"
description: "We take a look at how Kubernetes improves on the basic monolithic scheduler design."
tags:
  - "Series: Schedulers"
  - Schedulers, Kubernetes
author: sivaram_mothiki
---

In my [previous post](//posts/2015/schedulers-pt1-basic-monolithic) I introduced the concept of *scheduling* and took a look at two basic monolithic schedulers: *fleet* and *swarm*. In summary: schedulers are responsible for distributing jobs across a cluster of nodes. However, basic monolithic schedulers, by design, have limits on performance and throughput.

In this post we take a look at how Kubernetes improves on the basic monolithic design.

## Intro to Kubernetes

[Kubernetes](http://kubernetes.io/) is a tool for managing Linux containers across a cluster.

<!--more-->

Originally developed by Google, Kubernetes is lightweight, portable, and massively scalable. The design is highly decoupled and can split into two main components: a *control plane* and *worker node services**.* The controle plane which takes care of assigning containers to nodes and manages cluster configuration. Worker node services, which run on the individual machines in your cluster, manage the local containers.

Within Kubernetes, we have the concept of [*pod*](http://kubernetes.io/docs/user-guide/pods/). This is a group of colocated containers, like a pod of whales, or a pod of peas. Containers in the same pod share the same *namespace*. Namespaces are used for service discovery and segregation.

Kubernetes requires a dedicated subnet for each node—or an [overlay network](https://www.wikiwand.com/en/Overlay_network). This is so that each pod can get a unique IP address in the cluster. This can be achieved by deploying Kubernetes on a cluster with Flannel, OpenVSwitch, Calico, OpenContrail, or Weave. For more details about Kubernetes networking, see [the docs](http://kubernetes.io/docs/admin/networking/).

Let’s take a more detailed look at all this.

## Core Concepts

### Pods

A [pod](http://kubernetes.io/docs/user-guide/pods/) is the fundamental unit of Kubernetes. A pod is a group of containers that live on the same host and share the same set of [Linux namespaces](//posts/2015/linux-containers-isolation/).

When a pod is deployed, Kubernetes starts a container that locks the resources and namespace for the actual containers inside that pod. This is called a *pause container*.

Pods can support vertical stack applications. For example, a backend and a front end of tightly coupled application can live in the same pod.

A pod is fault tolerant on single worker node but not across a cluster. This can be enabled with the `--restart=on-failure` flag. With this, whenever the pod fails, it automatically restarts. But if a node goes down, the pods on that node are not rescheduled to another node.

Pods are not designed to scale across multiple nodes or surive a node failure. For this, we need a replication controller.

### Replication Controllers

A [replication controller](https://github.com/kubernetes/kubernetes/blob/master/docs/user-guide/replication-controller.md) takes care of scaling, updating, and rescheduling a pod. It also ensures the number of pods running at any point in time is equal to the number of replicas specified in the config. It does this by killing or starting pods to meet the replica number.

### Labels

[Labels](http://kubernetes.io/docs/user-guide/labels/) are key-value pairs that can be applied to pods. Labels are used to group a set of pods together and are used for routing network traffic.

### Namespaces

[Namespaces](http://kubernetes.io/docs/user-guide/namespaces/) are a way to segregate pods. Two pods from different namespaces cannot participate in service discovery. With this, we can create multiple environments in the same cluster, thereby providing application level isolation if we want it.

### Services

A [service](http://kubernetes.io/docs/user-guide/services/) acts as a point of contact for a set of related pods. The service then routes traffic to all the pods it is configured to work with.

Services let you address groups of pods without worrying about individual pod availability. Instead of accessing pods directly, access the service. The service will take care of routing traffic in a round-robin fashion to all pods that match the corresponding labels.

Services also helps in service discovery. When a pod gets created in a namespace, the host and port number of every service in the same namespace is provided via environment variables.

For example: in the `deis` namespace, I could create a service with name `db`. Every pod subsequently created in this namespace will then have `DB_SERVICE_HOST` and `DB_SERVICE_PORT` environment variables set.

### Daemon Sets

A [daemon set](http://kubernetes.io/docs/admin/daemons/) set ensures that a specific pod is running on every node (or set of nodes)  in the cluster. It's similar to a [global unit](https://coreos.com/fleet/docs/latest/unit-files-and-scheduling.html#unit-scheduling) for fleet. When an extra node is added to the cluster, Kubernetes adds the required pods onto that node. And if the cluster size decreases, pods on those nodes are garbage collected.

### Jobs

[Jobs](http://kubernetes.io/docs/user-guide/jobs/) are similar to pods but are meant to terminate. They are particularly useful for batch processing workloads.

Jobs and daemons sets are not available in the Kubernetes v1 API. If you can, switch to Kubernetes 1.2. Otherwise, enable the API extensions by starting the API server with `--runtime-config=extensions/v1beta1/daemonsets=true` or by exporting `ENABLE_DAEMONSETS=true` before running the `kube-up.sh` script.

## The Control Plane

The control plane has four major components: the API server, controller manager server, scheduler, and etcd.

### API Server

The API server is the face of your Kubernetes cluster. It is a standalone server providing a REST endpoint that does authentication, authorization, and handles all call from kubectl, the CLI utility for managing a Kubernetes installation. The API Server stores the state of the Kubernetes cluster in etcd.

A more detailed explanation of Kubernetes API can be found [in the docs](http://kubernetes.io/docs/api/) .

### Controller Manager Server

The *controller manager server* is a collection of several *controller managers*, which collectively ensure the cluster maintains the desired state.

For example, If you deploy a pod with the replication controller and set replica count to three, the *replication controller manager* ensures that three replicas will be running at any given point in time. If one of your replicas dies, the replication controller manager will schedule a new replica on a different node or the same node.

Kubernetes supports custom plugins for controller managers.

### Scheduler

The scheduler is responsible for tracking all the resources in the cluster and scheduling pods according to the resource requirements.

Whenever the API server receives a request to schedule a pod, it places the request in a queue and this queue is continuously read by scheduler, which eventually schedules the pod. The scheduler is pluggable and supports multiple cluster scheduling algorithms.

The scheduler is [monolithic](//posts/2015/schedulers-pt1-basic-monolithic/), but it’s decoupled from the API server and is uneffected by the availability of the API server. Even if the API server is unavailable, the scheduler can still repare pods, and can still read from the queue, and deploy pods in the cluster.

This decoupling helps Kubernetes overcome [the shortcomings of monolithic schedulers](//posts/2015/schedulers-pt1-basic-monolithic/).

More info about the scheduler can be found [in the ](http://kubernetes.io/docs/admin/kube-scheduler/)[docs](http://kubernetes.io/docs/admin/kube-scheduler/).

### etcd

etcd is a highly available [distributed key value store](//posts/2016/etcd-on-coreos/). It is designed to be simple, fast, reliable, and fault tolerant. etcd uses the [raft consensus algorithm](https://raft.github.io/) for leader election and logging.

## Worker Node Services

Individual Kubernetes nodes run two programs: *kubelet* and *kube-proxy*.

Together, the kubelet and a proxy service allow individual nodes to fully manage themselves and the containers running on the local host. This, in turn, reduces the amount of things the Kubernetes control plane is responsible for.

### kubelet

From a Docker point of view, the kubelet daemon manages images, containers, and volumes on the local host. From a Kubernetes point of view, the kubelet daemon functions as a single point of contact between Kubernetes API server and the local Kubernetes node it is running on, and is responsible for deploying pods and carrying out tasks as instructed.

### kube-proxy

The kube-proxy acts as a network proxy connecting locally running pods to outside world. It [round robins](https://www.wikiwand.com/en/Round-robin_DNS) incoming requests to pods according to [labels](http://kubernetes.io/docs/user-guide/labels/). It also functions as a load balancer for groups of pods sharing the same label.

## Conclusion

In my [last post](//posts/2015/schedulers-pt1-basic-monolithic/) we looked at the design limitations of monolithic schedulers. Kubernetes improves on this design in several ways.

Improvements over monolithic schedulers:

* Kubernetes decouples various scheduling and monitoring components, which improves system availability.
* Kubernetes uses etcd as a distributed queue, meaning state is stored in etcd and updated continuously via different components.
* Pods help with container colocation.
* Replication controllers help with scaling and fault tolerance.
* Services help with high availability and  load balancing.
* Labels let you group and isolate applications.

The design of Kubernetes makes it more than just a scheduler or an orchestration system for containers. Kubernetes neatly solves many difficult problems associated with clustering microservices and containerized applications.

You can find more about continuously evolving state of Kubernetes in this [blog post](http://blog.kubernetes.io/2016/03/Kubernetes-1.2-even-more-performance-upgrades-plus-easier-application-deployment-and-management-.html).

If you’re on Linux, [kube-up.sh](https://github.com/kubernetes/kubernetes/tree/master/cluster) is a good way to get started with Kubernetes. On OS X, [kube-solo](//posts/2015/zero-to-kubernetes-dev-environment-on-os-x) runs a single node CoreOS under [xhyve](https://github.com/mist64/xhyve) with Kubernetes installed in it.

In my next post we will talk about Apache Mesos, a two level centralized scheduler. We’ll also look at how to deploy a Kubernetes framework in Mesos.
