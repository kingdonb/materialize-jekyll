---
layout: post
title: Bootstrap a GlusterFS Cluster on GCE
description: "We demo a few scripts to quickly launch a GlusterFS cluster on GCE for scalable, persistent Kubernetes storage."
tags:
  - GlusterFS
  - GCE
  - GKE
  - Kubernetes
author: rimas
---

As we've covered before, shared file systems are [a tricky problem](https://deis.com/blog/2015/scale-everything/) in the cloud. One solution to that problem is a distributed file system. Something each one of your app nodes can read from and write to. When it comes to distributed file systems, [GlusterFS](https://www.gluster.org) is one of the leading products.

With a few simple scripts on your Mac OS X or Linux machine, you can deploy a multi-zone *High Availability* (HA) GlusterFS cluster to [*Google Compute Engine*](https://cloud.google.com/compute/) (GCE) that provides scalable, persistent shared storage for your GCE or [*Google Container Engine*](https://cloud.google.com/container-engine/) (GKE) [Kubernetes](http://kubernetes.io) clusters.

In this post, I will demo these scripts and show you how to do this. By default, our GlusterFS cluster will use three GlusterFS servers, one server per Google Cloud zone in the same chosen region.

<!--more-->

## Prerequisites

Before continuing, please make sure you have:

* A [Google Cloud](https://cloud.google.com) account
* The [Google Cloud SDK](https://cloud.google.com/sdk/) installed
* Git installed

## Clone

The first thing you need to do is clone [my GitHub repo](https://github.com/rimusz/glusterfs-gce):

````
$ git clone https://github.com/rimusz/glusterfs-gce
```

Then, open the `cluster/settings` file in your text editor. Find the section marked `your cluster region and zones zones` and set `REGION` and `ZONES` to whatever matches your preferred setup. The rest of settings in this file are probably fine, but can be adjusted if need be.

## Bootstrap Your Cluster

There's nothing more you need to do. You should already be authenticated with Google from when you installed the Google Cloud SDK.

You can go right ahead and create the cluster by running:

```
$ ./cluster/create_cluster.sh
```

This command will create three servers.

Each server will have:

* A static IP
* The GlusterFS server package installed
* A Google Cloud persistent disk to be used as a GlusterFS *brick*, that is: storage space made available to the cluster

## Create Your First Volume

A GlusterFS *volume* is a collection of bricks. A volume can store data across the bricks in three basic ways: distributed, striped, or replicated.

In summary:

* A distributed volume stores each file on one brick
* A striped volume stores a single file, in chunks, across multiple bricks
* A replicated volume stores a copy of each file on every brick

My script configures the replicated volume store. This provides you with faul-tolerance, should one of your GlusterFS nodes go down.

Here's how to create a replicated volume on all three servers:

```
$ cd ..
$ ./cluster/create_volume.sh VOLUME_NAME
```

At this point, your GlusterFS cluster should be fully set up and operational.

Let's test it.

## Testing Your Volume

Spin up a new GCE virtual machine, or use one of your existing non-GlusterFS virtual machines, and grab a shell on that machine.

Then mount your GlusterFS volume (replacing `VOLUME_NAME` with the actual volume name you chose) to `/mnt/gfs` by running this command:

```
mount -t glusterfs gfs-cluster1-server-1:/VOLUME_NAME /mnt/gfs
```

The `gfs-cluster1-server-1` is the name your cluster is given by my script.

Then copy a bunch of files into the mount point:

```
$ for i in `seq -w 1 100`; do cp -rp /var/log/messages /mnt/copy-test-$i; done
```

You don't have to run this exact command. Any files will do.

Now, grab a shell on one of your GlusterFS virtual machines, and take a look inside the brick volume to see the files you just created.

Do that like so:

```
$ ls -lA /data/brick1/VOLUME_NAME
```

Again, replace `VOLUME_NAME` with the volume name you chose.

## Wrap-Up

In this post, we saw how to use [a few scripts](https://github.com/rimusz/glusterfs-gce) to quickly launch a GlusterFS cluster on GCE for scalable, persistent storage for your apps.

In `cluster` folder of this repo, there are two more scripts:

* The `upgrade_glusterfs.sh	` script upgrades GlusterFS on all servers
* The `upgrade_servers.sh` script upgrades your distro (via APT) on all servers

To learn more about using GlusterFS with Kubernetes, check out the [GlusterFS examples](https://github.com/kubernetes/kubernetes/tree/release-1.2/examples/glusterfs/) in the official Kubernetes repository.

We plan to cover more on this topic in future posts!
