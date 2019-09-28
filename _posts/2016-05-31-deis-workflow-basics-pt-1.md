---
layout: post
title: Deis Workflow Basics, Part One
description: "Part one of our Deis Workflow overview, covering: reasons to use Workflow, and basic concepts."
tags:
  - Deis Workflow
  - "Series: Deis Workflow Basics"
author: rimas
---

Deis Workflow is an open source *Platform as a Service* (PaaS) that makes it easy to deploy and manage applications on your own servers. Workflow builds on [Kubernetes](http://kubernetes.io/) and [Docker](https://www.docker.com) to provide a lightweight PaaS with a Heroku-inspired workflow.

Deis Workflow is the second major release of the Deis PaaS.

In this miniseries we'll go over the basics of Deis Workflow. That includes: why you'd want to use Workflow, a conceptual overview, a look at architecture and components, and finally, how to install Workflow on a Kubernetes cluster.

## Why Use Workflow?

<!--more-->

There are five main reasons you might want to use Workflow:

1. **Fast and easy.** Supercharge your team with a platform that deploys applications as fast as you can create them. No Kubernetes knowledge is needed.

2. **You can deploy anything.** Deploy any language, framework, or Dockerfile with a simple `git push`. Use `deis pull` to move existing Docker images through your team's development CI/CD pipeline.

3. **Scale effortlessly.** Your application can be scaled up or down with a single command that handles everything for you: `deis scale`.

4. **Open source.** Avoid vendor lock-in with an open source platform that runs on public cloud, private cloud, or bare metal. Contribute to the project if you want to add features and help us set direction.

5. **The latest technology.** Benefit from the latest distributed systems technology thanks to a platform that is constantly evolving.

## Concepts

Workflow is a lightweight application platform that deploys and scales [Twelve-Factor apps](http://docs.deis.io/en/latest/understanding_deis/concepts/#concepts-twelve-factor) as containers across a [Kubernetes](http://kubernetes.io/) cluster.

### Twelve-Factor

The [Twelve-Factor app](http://12factor.net/) is a template for building modern applications that can be scaled across a distributed system. This approach is a distillation of years worth of experience working with *Software as a Service* (SaaS) apps on the Heroku platform.

Workflow, like most Platforms as a Service, is designed to run applications that adhere to the [Twelve-Factor App](http://12factor.net/) approach. However, [legacy apps](https://deis.com/blog/2015/pets-vs-cattle/) can still run on Deis. They just need some [extra attention](https://deis.com/blog/2015/why-your-app-wont-work-in-the-cloud/) to get things working.

### Docker

[Docker](https://www.docker.com/) is an open source project to build, ship, and run any application as a lightweight, portable, self-sufficient container.

If you have not yet converted your application to containers, Workflow provides a simple and straightforward "source to Docker image" capability.

Because Workflow supports multiple language runtimes via Heroku [buildpacks](https://deis.com/docs/workflow/applications/using-buildpacks/), building your app in a container can be as easy as `git push deis master`.

Applications packaged via a Heroku buildpack are run in Docker containers as part of the [slugrunner](https://github.com/deis/slugrunner) process. Applications which use either a Dockerfile or reference an external Docker Image are launched unmodified.

### Kubernetes

[Kubernetes](http://kubernetes.io/) is an open-source cluster manager developed by Google and donated to the[ Cloud Native Compute Foundation](https://cncf.io/). Kubernetes manages all the activity on your cluster, including: desired state convergence, stable service addresses, health monitoring, service discovery, and DNS resolution.

Workflow builds on Kubernetes, leveraging this functionality to provide simple, developer-friendly app deployment.

Workflow is shipped as a Kubernetes-native application. It's installable via [Helm Classic](https://helm.sh/) so systems engineers and operators familiar with Kubernetes will feel right at home running Workflow.

The platform and your applications run in dedicated Kubernetes namespaces, nicely separating workloads.

Using Kubernetes *services* and *replication controllers*, Workflow deploys new versions of your app with zero downtime.

If you have workloads that aren't managed by Workflow, you can easily connect them through Kubernetes' baked in service discovery.

### Applications

Workflow is designed around the concept of an [*application*](https://deis.com/docs/workflow/reference-guide/terms/#application), or app.

Apps can come in three forms:

1. A collection of *buildpack* source files stored in a Git repository
2. A *Dockerfile*, which describes how to build your app
3. A reference to an already built *Docker Image*, hosted on a remote Docker repository

Apps are identified by a unique name, for easy reference. If you do not specify a name when creating your app, Workflow generates one for you.

Workflow also tracks other app information, including domain names, SSL certificates, and other config provided via environment variables.

### Build, Release, Run

The diagram below shows the App's workflow:

![](/images/blog-images/deis-workflow-basics-pt-1-0.png)

#### Build Phase

The [builder](https://deis.com/docs/workflow/understanding-workflow/components/#builder-builder-slugbuilder-and-dockerbuilder) component processes incoming `git push deis master` requests and manages your application packaging. Like so:

1. If your application is using a[ buildpack](https://deis.com/docs/workflow/applications/using-buildpacks/), the builder will launch an ephemeral job to extract and execute the packaging instructions. The resulting application artifact is stored in [*Workflow Object Storage*](https://deis.com/docs/workflow/installing-workflow/configuring-object-storage/), and extracted by the platform for execution during the run stage.
2. If instead, you provide a[ Dockerfile](https://deis.com/docs/workflow/applications/using-dockerfiles/), builder will use the instructions in the Dockerfile to build a Docker Image. The resulting artifact is stored in a Deis managed private Docker registry which will be referenced during the run stage.
3. If you already have an external system building your application container you can simply reference that artifact. When using [external Docker images](https://deis.com/docs/workflow/applications/using-docker-images/), the builder component doesn't attempt to rebuild your image. But the image will be directly pulled onto Kubernetes nodes and used by your app.

#### Release Phase

During the release stage, a[ build](https://deis.com/docs/workflow/reference-guide/terms/#build) is combined with[ application configuration](https://deis.com/docs/workflow/reference-guide/terms/#config) to create a new numbered[ release](https://deis.com/docs/workflow/reference-guide/terms/#release).

New releases are created every time a new build is created or application configuration is changed.

So, for example, you could alter your config like so:

```
$ deis config:set REDIS_MASTER_SERVICE_HOST=redis-master.default -a guestbook
```

When you do this, Workflow will build, release, and run a new version of the app with that config. And by "version", we mean: every release is assigned a version number. Versioning every release makes it easy to roll back to any previous release.

#### Run Phase

The run stage is responsible for deploying the new release to the underlying Kubernetes cluster.

The run stage launches a new *replication controller* which references the new release. By managing the desired replica count, Workflow orchestrates a zero-downtime, rolling update of your application. Once successfully updated, Workflow removes the last references to the old release.

That during the deploy, your application will be running in a mixed mode. The old and the new replication controllers will run at the same. When the new version of your app is up and running, the old replication controller is stopped.

### Backing Services

Workflow treats all persistent services (such as databases, caches, storage, messaging systems, and other[ backing services](http://12factor.net/backing-services)) as resources managed separately from your application. This aligns with the [Twelve-Factor](http://12factor.net/backing-services) approach.

Applications are attached to backing services using[ environment variables](http://12factor.net/config).

Because applications are decoupled from backing services, apps are free to scale up independently, to use services provided by other apps, or to switch to external or third-party vendor services.

[Helm Classic](http://helm.sh/), the package manager for Kubernetes, is a great way to install and set up some of those backing services.

## Wrap Up

Deis Workflow is a Heroku-inspired Platform as a Service that builds on Kubernetes and Docker. It is fast and easy to use. You can deploy anything you like. Release are versioned and rollbacks are simple. You can scale up and down effortlessly. And it is 100% open source, using the latest distributed systems technology

In part two of this miniseries, we'll look at how Workflow is architected and composed from multiple independent components.
