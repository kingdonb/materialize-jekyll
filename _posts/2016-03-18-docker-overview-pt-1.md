---
layout: post
title: Docker Overview, Part One
description: "New to Docker? In part one of our overview, take a look at why it’s special and how it’s built."
tags:
  - "Series: Docker Overview"
  - Docker
author: rimas
---

In my last miniseries, we gave [an overview of CoreOS](https://deis.com/blog/2016/coreos-overview-p1). In this miniseries, I want to take a look at Docker. In this post we’ll look at why it’s special and how it’s built. We’ll start at the beginning, so don’t worry if you’re totally new to this.

Docker is an [open source](https://en.wikipedia.org/wiki/Open-source) project that makes easier packaging of [applications](https://en.wikipedia.org/wiki/Application_software) inside [containers](https://en.wikipedia.org/wiki/Software_container) by providing an additional layer of abstraction and automation of [operating-system-level virtualization](https://en.wikipedia.org/wiki/Operating-system-level_virtualization) on [Linux](https://en.wikipedia.org/wiki/Linux).

Containers themselves are just an abstraction over Linux *cgroups*, which are a lower level kernel construct for [jailing and limiting](https://deis.com/blog/2015/isolation-linux-containers) the resources of a process and its children.

Docker was using [*LinuX Containers*](http://lxc.sourceforge.net/) (LXC) in the beginning, but then [switched](https://blog.docker.com/2015/06/runc/) to [runC](https://github.com/opencontainers/runc), formerly known as libcontainer. [runC](https://github.com/opencontainers/runc) runs in the same operating system as its host. This allows it to share a lot of the host operating system resources, such as RAM, CPU, networking, and so on.

<!--more-->

## Why Is Docker So Special?

Docker adds an application deployment engine on top of a virtualized container environment. It is a lightweight and powerful open source container virtualization technology combined with a workflow for building and containerizing applications.

Some benefits:

* Docker lets you quickly assemble applications from components and eliminates the friction that can come when shipping code. For example, you can have two Docker containers running two different versions of the same app on the same host.
* Docker lets you get your code tested and deployed into production as fast as possible.
* Docker is incredibly simple to use. You can get started with Docker on a minimal Linux, OS X, or Windows host running nothing but a compatible Linux kernel directly or in a *Virtual Machine* (VM) with a Docker binary.
* You can "dockerize" your application in minutes. Most Docker containers take less than a second to launch.
* Docker containers run (almost) everywhere. You can deploy containers on desktops, physical servers, virtual machines, into data centers, and up to public and private clouds. And, you can run exactly the same containers everywhere.

We could go on and on about Docker’s amazing containerization features! But let’s take a look at the main differences between VMs and containers.

## Differences Between VMs and Containers

Containers have similar resource isolation and allocation benefits as VMs, but a different architectural approach allows them to be much more portable and efficient. Let's look at two diagrams that illustrate the differences.

### Virtual Machines

Each VM includes the application, the necessary binaries and libraries, and an entire guest operating system—all of which may be tens of GBs in size.

![virtual machines](/images/blog-images/docker-overview-1-0.png)

### Containers

Containers include the application and all of its dependencies and use their own Linux distro of choice, but they share the kernel of host operating system with other containers.

They run as isolated processes in userspace on the host operating system. Additionally, they are not running all the processes that a guest OS would normally run. And they’re also not tied to any specific infrastructure! Docker containers run on any computer, on any infrastructure, and in any cloud.

![containers](/images/blog-images/docker-overview-1-1.png)

Imporantly: VMs can take minutes to boot and are resource intensive, while containers need only a few seconds or less and come with [much less resource overhead](https://deis.com/blog/2015/isolation-linux-containers). So whereas you might only be able to run a few VMs on your local computer for development purposes, you can run many, many containers. This effectively allows you to run a full duplicate of your production setup in dev. And in production, these resource savings translate to cost savings.

## Docker Components

There are a few top-level components to Docker that we need to understand.

Let’s take a look.

### Docker Client and Server

Docker is a client-server application.

![docker client and server](/images/blog-images/docker-overview-1-2.png)

The Docker client talks to the Docker server (or *daemon*) which, in turn, does all the work. Docker ships with a command line client binary (for Linux, OS X, and Windows) as well as a full RESTful API. You can run the Docker daemon and client on the same Linux host or connect your local Docker client to a remote daemon running on another Linux host.

You cannot run the Docker server natively on Mac OS X or Windows (because Docker containers needs LInux kernel) but you can run it inside a virtualized Docker-compatible host OS, e.g. CoreOS running on VirtualBox.

### Docker images

Docker *images* are the building blocks of Docker. Images are the "build" part of Docker's life cycle. They are a layered format, using Union file systems that are built step-by-step using a series of instructions.

A *Dockerfile* is a text document that contains all the commands a user could call on the command line to assemble an image. From using a base Docker image, to adding and copying files, running commands, and exposing ports.

You can consider the Dockerfile to be "source code" and images to be the "compiled code" for your containers, which are the "running code".

Dockerfiles are highly portable and can be shared, stored, and updated.

### Registries

Once you’ve build (compiled) an image, you store it in a *registry*.

There are two types of registries: public and private.

There are hosted registry services such as [Docker Hub](https://hub.docker.com/), [Quay.io](https://quay.io/) and [Container Registry by Google Cloud](https://cloud.google.com/container-registry/) with public and private repository options. You can also run your a private registry on your own hardware.

### Containers

A container is a running version of an image.

You can think about images as the building or compiling aspect of Docker and the containers as the running or execution aspect of Docker.

The life-time of a container matches the life-time of the command you run within it. A container can be run for a single command execution and terminated immidately afterwards, or you can keep them running with a long-running command.

## Wrap-Up

In this post we covered:

* What makes Docker so special
* The main differences between virtual machines and containers
* The primary components that make up Docker

In my next post, we’ll look at launching containers, building images, and working with data volumes.
