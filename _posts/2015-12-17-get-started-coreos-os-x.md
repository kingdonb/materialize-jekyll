---
layout: post
title: Get Started With CoreOS on OS X
description: "Learn how to run CoreOS on OS X without using VirtualBox."
tags:
  - CoreOS
author: rimas
---

In one of our recent posts we have covered how to install [CoreOS](https://deis.com/blog/2015/coreos-on-virtualbox) with VirtualBox.

In this post, we learn how to run CoreOS *without* VirtualBox.

This setup is specific to OS X as it uses *xhyve*, which is built on top of `Hypervisor.framework` [introduced in OS X 10.10](https://developer.apple.com/library/mac/releasenotes/MacOSX/WhatsNewInOSX/Articles/MacOSX10_10.html).

## Install

We’re going to use the *CoreOS VM* application, which is an independent open source project that bundles CoreOS into a VM for running on OS X.

The CoreOS VM app uses the new [*corectl*](https://github.com/TheNewNormal/corectl) tool to manage [xhyve](https://github.com/mist64/xhyve)-based VMs.

First, download the [CoreOS VM app](https://github.com/TheNewNormal/coreos-osx).

The CoreOS VM App does not have many dependencies to download. Everything needed is already included. However, the app will perform some automatic downloads on your behalf., they include CoreOS ISO file needed to run the VM, as well as Docker and [fleetctl](https://coreos.com/fleet/docs/latest/using-the-client.html) clients for OS X.

<!--more-->

## Run

You can double click the app to run it. 

When you run the app for first time it checks to see if you have [iTerm2](http://www.iterm2.com) installed. If not, this software gets installed automatically into your Applications folder.

You’ll then be asked what disk size you want for the VM and which CoreOS release channel to use. Don’t worry. You can change this later.

That’s it. You’re good to go.

All app’s files get installed under "~/coreos-osx" folder. No other files or folders are changed.

Look for this icon in your task bar:

![coreos logo](/images/blog-images/coreos-vm-0.png) 

Click it, and you should see a menu like this:

![coreos menu](/images/blog-images/coreos-vm-1.png) 

You can do lots of things from here, including: starting, halting, reloading, ssh, and opening an OS shell with some pre-set environment variables.

Here's the booted VM with the opened pre-set shell window in iTerm:

![vm console](/images/blog-images/coreos-vm-2.png) 

The OS shell presets these variables:

```
DOCKER_HOST=tcp://192.168.64.3:2375
FLEETCTL_ENDPOINT=http://192.168.64.3:2379
ETCDCTL_PEERS=http://192.168.64.3:2379
```

When you start your VM, [Docker Registry](https://github.com/docker/distribution) gets started (outside of the VM) on `192.168.64.1:5000`. You can then push you Docker images to that IP and port number so you can share them with other VMs you might be running on your Mac.

If you destroy your VM (via the menu) Docker registry will persist the images. Docker registry is running as an OS X background service, with Docker images stored directly on your Mac’s disk, and not inside the VM.

Selecting [*Fleet-UI*](http://fleetui.com) from menu will open the list of currently running CoreOS fleet units in your default web browser:

![fleet menu](/images/blog-images/coreos-vm-3.png) 

Selecting [DockerUI](https://github.com/crosbymichael/dockerui) (the web interface for Docker) will similarly show running Docker containers and other info about your Docker images:

![docker dashboard](/images/blog-images/coreos-vm-4.png) 

You can check your running VM with `corectl` utility:

```
$ corectl ps 
[corectl] found 1 running VMs, summing 2 vCPUs and 2048MB in use.
- core-01, beta/877.1.0, PID 1502 (detached=true), up 6m47.506766498s
```

Running `corectl -h` will show all available commands:

```
Usage:
  corectl [flags]
  corectl [command]

Available Commands:
  rm          Removes one or more CoreOS images from local fs
  kill        Halts one or more running CoreOS instances
  ls          Lists locally available CoreOS images
  load        Loads CoreOS instances defined in an instrumentation file.
  version     Shows corectl version information
  ps          Lists running CoreOS instances
  pull        Pulls a CoreOS image from upstream
  run         Starts a new CoreOS instance
  ssh         Attach to or run commands inside a running CoreOS instance
  put         copy file to inside VM
```

Other useful things can be done from the application menu:

- You can upload Docker images to your VM
- You can update your local fleetctl and Docker OS X clients after you have downloaded the lastest CoreOS ISO version

## Conclusion

We have learned how it is easily to install and use CoreOS VM on OS X without needing to use VirtualBox.

For more about Docker, see:

- [Get Started With Docker on Your Non-Linux PC](https://deis.com/blog/2015/get-started-with-docker-on-your-non-linux-pc)
- [Dockerfile Instructions and Syntax](https://deis.com/blog/2015/dockerfile-instructions-syntax)
- [Create and Share Your First Docker Image](https://deis.com/blog/2015/creating-sharing-first-docker-image)

For more about CoreOS, see:

- [Schedulers, Part 1: Basic Monolithic](https://deis.com/blog/2015/schedulers-pt1-basic-monolithic)
- [Run Self-Sufficient Containers on CoreOS](https://deis.com/blog/2015/run-self-sufficient-containers-coreos)

