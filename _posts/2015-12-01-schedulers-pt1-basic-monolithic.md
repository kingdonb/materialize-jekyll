---
layout: post
title: "Schedulers, Part 1: Basic Monolithic"
description: "We take a look at two basic monolithic schedulers: fleet and Swarm."
tags:
  - "Series: Schedulers"
  - Docker
  - Fleet
  - Swarm
  - Overview
author: sivaram_mothiki
---

Scheduling is a method to to assign workloads to resources that can handle those workloads. In a distributed environments, there is a particularly important need for schedulers. Especially ones that provide scalability, are resource aware, and cost effective.

Monolithic schedulers are a single process entity that make scheduling decisions and deploy jobs to be scheduled. These *jobs* could be a long running server, a short living batch command, a MapReduce query, and so on.

For monolithic scheduler to make decision for scheduling a job, it should observe the resources available  in the cluster (e.g. CPU, memory, and so on), lock the resources, schedule the job, and update the available resources.

It’s hard for Monolithic schedulers to deal with more than one job at a time because there is a single resource manager entity a single scheduling entity. Avoiding concurrency makes the system easier to design and easier to understand.

Some examples of monolithic schedulers:

- [*fleet*](https://coreos.com/using-coreos/clustering/): a native scheduler to [*CoreOS*](https://coreos.com/) (not resource aware)
- [*Swarm*](https://www.docker.com/docker-swarm): a scheduling backend for Docker containers
- [*Kubernetes*](http://kubernetes.io/): an advanced type of monolithic scheduler for [*Pods*](http://kubernetes.io/docs/user-guide/pods/) (a collection of co-located containers that share same namespaces)

<!--more-->

In this post, we'll cover two basic monolithic schedulers.

First we'll look fleet, a scheduler for [*systemd*](https://docs.docker.com/v1.8/articles/systemd/) units. systemd is an advanced init system that that configures and boots Linux userspace. A unit is an individual systemd configuration file that describes a process you’d like to run. We’ll also look at Swarm, a scheduling backend for Docker containers.

## fleet

Essentially, fleet is systemd for your cluster.

fleet takes advantages of systemd and [*etcd*](https://github.com/coreos/etcd), a distributed key-value store used by CoreOS to create a distributed init system. Before scheduling a systemd unit, fleet figures out which nodes are running the least amount of unit files, and schedules the next unit file to that node.

However, fleet is not a resource aware scheduler. fleet doesn’t take available system resources like CPU and memory into consideration when scheduling a systemd unit. It only uses the simple algorithm described above.

We can specify some constraints in unit files. For example, we can say that a unit file should be run on every host.

Here’s an example Fleet unit file which deploys [the Deis store admin](http://docs.deis.io/en/latest/troubleshooting_deis/troubleshooting-store/):

```
[Unit]
Description=deis-store-admin

[Service]
EnvironmentFile=/etc/environment
TimeoutStartSec=20m
ExecStartPre=/bin/sh -c "IMAGE=`/run/deis/bin/get_image /deis/store-admin` \
    && docker history $IMAGE   >/dev/null 2>&1 || docker pull $IMAGE"
ExecStartPre=/bin/sh -c "docker inspect deis-store-admin >/dev/null 2>&1 \
    && docker rm -f deis-store-admin >/dev/null 2>&1 || true"
ExecStart=/bin/sh -c "IMAGE=`/run/deis/bin/get_image /deis/store-admin` \
    && docker run --name deis-store-admin --rm \
    --volumes-from=deis-store-daemon-data    \
    --volumes-from=deis-store-monitor-data \
    -e HOST=$COREOS_PRIVATE_IPV4 $IMAGE"
ExecStopPost=-/usr/bin/docker rm -f deis-store-admin
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

[X-Fleet]
Global=true
```

Constraints are set using `X-Fleet` section. These are instructions for fleet itself. The rest of the configuration file is for systemd. For example, in the above example, `Global=true` under `X-Fleet` section tells fleet to schedule the unit (i.e. deploy it to) every machine in the cluster.

Refer to [unit files and scheduling](https://github.com/coreos/fleet/blob/master/Documentation/unit-files-and-scheduling.md) for more option to customize a fleet unit file.

Typically, you use the `fleetctl` command to submit a fleet unit. If you’re using [Deis](https://github.com/deis/deis), you can use the `deisctl` command.

By setting `FLEETCTL_TUNNEL` for fleetctl (or `DEISCTL_TUNNEL` for deisctl) to any of the CoreOS IPs in the cluster, a user can talk to the fleet daemon on the host.

This command will install a fleet unit in the cluster:

```
deisctl install <unit name>
```

This command will run the unit:

```
deisctl start <unit name>
```

## Swarm

Swarm is a scheduling backend for Docker containers in a cluster.

Swarm works via the Docker API, so a container can be scheduled in a cluster by using the Docker client or any tool that talks to [Docker Engine](https://www.docker.com/docker-engine).

Swarm is an extension to the [libswarm](https://github.com/docker/swarm/) project. Swarm comes with a swarm manager which talks to swarm nodes on each node in the cluster. The advantage of Swarm is that it supports [filters](http://docs.docker.com/swarm/scheduler/filter) and strategies which act as rules to fine tune the way a container gets scheduled in a cluster.

Let’s take an example command:

```
docker run -d --name bar -e affinity:container==foo bar:v1
```

Here we’re specifying `affinity:container==foo`, which tells Swarm to deploy container `bar` on a node where there is a container by the name `foo`.

The user also has the option to specify soft affinities with a `~`, as in `affinity:container==~foo`. If foo container is not present on any node, Swarm will discard the affinity and schedule the container according to the scheduling strategy. Currently supported scheduling strategies with Swarm are: [*spread*](https://github.com/docker/swarm/tree/master/scheduler/strategy), [*bin packing*](https://github.com/docker/swarm/tree/master/scheduler/strategy), and [*random*](https://github.com/docker/swarm/tree/master/scheduler/strategy).

By default, [Deis](http://deis.io/) comes with fleet. Current version of Deis also support Swarm.

To install Swarm, run:

```
deisctl install swarm && deisctl start swarm
```

To switch the scheduling backend:

```
deisctl config controller set schedulerModule=swarm
```
## Conclusion

Monolithic scheduler design limits performance and throughput.

The resource manager and scheduler live in the same process, Monolithic schedulers are not sensitive to dynamic resource changes in the cluster.  Scheduling is tightly coupled, reducing extensibility. For example, fleet can only schedule systemd units and Swarm is only for docker containers.

fleet is a low-end scheduler and is not resource aware. It is not advisable to use fleet for clusters larger than 100 nodes.  Fortunately, Deis V2 will work with Kubernetes, a much more advanced scheduler. Swarm has it’s own weakness too. For example, as of version 0.2, Swarm doesn’t support node failover. So if a node becomes unavailable, Swarm doesn’t reschedule the containers on the failed node.

In [my next post](/posts/2016/schedulers-pt2-kubernetes/), I discuss advanced schedulers like Kubernetes, Apache Mesos, Omega, and Sparrow. I also talk about the ways these tools improve on the basic monolithic schedulers like Swarm and fleet.
