---
layout: post
title: "Internal Load Balancing on Google Container Engine"
description: "Learn how to set up a load balancer on Google Container Engine that watches your cluster for IP changes."
tags:
  - Kubernetes
  - Google Cloud Platform
  - Google Compute Engine
author: rimas
---

Internal load balancing is important for many infrastructures. But, if you've tried to do it for Google Container Engine, you'll know there's no prepackaged solution. Well, fear not. I've written a tool to help you out. So keep reading.

To quote the [*Google Cloud Compute Engine* (GCE) docs](https://cloud.google.com/solutions/internal-load-balancing-haproxy):

> An internal load balancer distributes network traffic on a private network that is not exposed to the Internet. Internal load balancing is useful not only for intranet applications where all traffic remains on a private network, but also for complex web applications where a frontend sends requests to backend servers via a private network.

<!--more-->

Here's a diagram of the HAProxy setup they recommend, to give you an idea of what we're talking about:

![HAProxy screenshot](/images/blog-images/ilb-haproxy-high-level-diagram.png)

Thing is. I don't want to have to do the manual work to set all that up.

Also, if you read through the instructions, you'll see this recommended setup only works for VMs with a static IP. But that's a problem, because the *Google Container Engine* (GKE) nodes sometimes get new IPs when Kubernetes is upgraded.

That's where my new tool comes in.

## gke-internal-lb

*[gke-internal-lb](https://github.com/rimusz/gke-internal-lb)* is a tool that bootstraps a HAProxy VM, following the recommended Google setup. However, this HAProxy VM now also watches your GKE cluster and updates the HAProxy configuration file and restarts HAProxy when it detects a new IP address.

### How to Use It

Grab a local copy:

```
git clone https://github.com/rimusz/gke-internal-lb
```

Open `create_internal_lb.sh` in your editor.

Find this section of the file:

```
##############################################################################
# GC settings
# your project
PROJECT=_YOUR_PROJECT_
# your GKE cluster zone or which ever zone you want to put the internal LB VM
ZONE=_YOUR_ZONE_
#

# GKE cluster VM name without all those e.g -364478-node-sa5c
SERVERS=gke-cluster-1

# static IP for the internal LB VM
STATIC_IP=10.200.252.10

# VM type
MACHINE_TYPE=g1-small
##############################################################################
```

Update this information with your GCE project and zone, and any other settings you want to change.

Now run the script:

```
./create_internal_lb.sh
```

### What It Does

Running this script performs the following actions:

1. Checks for an existing setup and deletes it, along with all dependencies
2. Creates a temporary VM, and sets it up
3. Deletes the temporary VM keeping its boot disk
4. Creates the custom image from the boot disk
5. Creates a HAProxy instance template based on the custom image
6. Creates a managed instance group with the new VM

After that, you'll have an internal load balancer VM running HAProxy that forwards all received HTTP traffic to all GKE cluster nodes on port 80. You can configure the port number by editing the `get_vms_ip.tmpl` file.

This load balancer VM is watched by the Instance Group Manager. If the VM stops it gets restarted. Similarly, the HAProxy service running on the load balancer VM is set to always be running. So systemd restarts the service if it stops.

And here's the best bit: `/opt/bin/get_vms_ip` gets run by cron every two minutes. This script checks for cluster IP changes. If it detects any, it updates HAProxy configuration file as needed, and restarts HAProxy.

You can configure the cron frequency by editing the `create_internal_lb.sh` file.

## Conclusion

In this post, we looked at how to use *[gke-internal-lb](https://github.com/rimusz/gke-internal-lb)* to set up an internal load balancer on Google Container Engine that watches your cluster and updates the HAProxy configuration file when backend node IP addresses change.

This is an open source tool. If you'd like to make changes, or request changes, please send a ticket or a pull request! Thank you!
