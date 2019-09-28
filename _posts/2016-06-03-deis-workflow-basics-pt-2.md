---
layout: post
title: Deis Workflow Basics, Part Two
description: "Part two of our Deis Workflow overview, covering: Workflow architecture and components."
tags:
  - Deis Workflow
  - "Series: Deis Workflow Basics"
author: rimas
---

This is the second post in a series on Deis Workflow, the second major release of the Deis PaaS. Workflow builds on Kubernetes and Docker to provide a lightweight PaaS with a Heroku-inspired workflow.

In [part one](https://deis.com/blog/2016/deis-workflow-basics-pt-1/), we gave a brief conceptual overview, including Twelve-Factor apps, Docker, Kubernetes, Workflow applications, the "build, release, run" cycle, and backing services. We also explained the benefits of Workflow. In summary:

<blockquote>
Workflow is fast and easy to use. You can deploy anything you like. Release are versioned and rollbacks are simple. You can scale up and down effortlessly. And it is 100% open source, using the latest distributed systems technology.
</blockquote>

In this post, we'll take a look at the architecture of Workflow and how Workflow is composed from multiple, independent components.

<!--more-->

## Architecture

Deis Workflow is built using a service oriented architecture. All components are published as a set of container images which can be deployed to any Kubernetes cluster.

### Overview

Deis Workflow sits at the top of your stack:

![](/images/blog-images/deis-workflow-basics-pt-2-0.png)

Operators can use [Helm Classic](http://helm.sh/) to configure and install the Workflow components which interface directly with the underlying Kubernetes cluster.

Service discovery, container availability, and networking are all delegated to Kubernetes, while Workflow provides a clean and simple developer experience.

### Platform Services

Workflow provides additional functionality on top of your Kubernetes cluster:

![](/images/blog-images/deis-workflow-basics-pt-2-1.png)

In detail, these are:

* An [image builder](https://deis.com/docs/workflow/understanding-workflow/components/#builder-builder-slugbuilder-and-dockerbuilder) that compiles your applications via buildpacks or Dockerfiles
* [Cross-pod log aggregation](https://deis.com/docs/workflow/understanding-workflow/components/#logger-fluentd-logger) that gathers logs from all of your application processes
* A [simple REST API](https://deis.com/docs/workflow/understanding-workflow/components/#controller) that powers the CLI and any external integrations
* Application release and rollback
* Authentication and authorization to application resources
* [HTTP(S) edge routing](http://deis.com/docs/workflow/understanding-workflow/components/#router) for your applications

Check out the [components docs](https://deis.com/docs/workflow/understanding-workflow/components/) for more.

### Kubernetes Native

All platform components and applications deployed via Workflow expect to be running on an existing Kubernetes cluster.

This means that you can happily run your Kubernetes-native workloads next to applications that are managed through Deis Workflow.

Here's how you might partition a Kubernetes cluster:

![](/images/blog-images/deis-workflow-basics-pt-2-2.png)

### Application Layout and Edge Routing

By default Workflow creates per-application namespaces and services so you can easily connect your applications to other on-cluster services with standard Kubernetes mechanisms. For example:

![](/images/blog-images/deis-workflow-basics-pt-2-3.png)

The router component is responsible for routing HTTP(S) traffic to your applications as well as proxying `git push` and platform API traffic.

By default, the router component is deployed as a Kubernetes service with type `LoadBalancer`.  Depending on your configuration, this will provision a cloud-native load balancer automatically.

The router automatically discovers routable applications, SSL/TLS certificates, and application-specific configurations through the use of Kubernetes annotations. And any changes to router configuration or certificates are applied within seconds.

### Topologies

Deis Workflow does not dictate a specific topology or server count for your deployment anymore, which *was* the case when setting up version one of the Deis PaaS.

Workflow platform components will happily run on single-server Kubernetes configurations as well as multi-server production Kubernetes clusters.

## Workflow Components

Workflow is comprised of a number of small, independent services that combine to create a distributed PaaS. All Workflow components are deployed as services (and associated controllers) in your Kubernetes cluster.

All Workflow components are built with composability in mind. If you need to customize one of the components for your specific deployment or need the functionality (divorced from Workflow proper) in your own project, we invite you to give it a shot!

### Controller

The [controller component](https://github.com/deis/controller) is an HTTP API server which serves as the endpoint for the `deis` CLI.  The controller performs management functions for the platform, as well as interfacing with your Kubernetes cluster.

The controller persists all of its data to the database component.

### Database

The [database component](https://github.com/deis/postgres) is a managed instance of[ PostgreSQL](http://www.postgresql.org/), and holds a majority of the platform state. Backups and WAL files are pushed to Workflow’s object storage via [WAL-E](https://github.com/wal-e/wal-e). When the database is restarted, backups are fetched and replayed from object storage so no data is lost.

### Builder

The [builder component](https://github.com/deis/builder) is responsible for accepting code pushes via[ Git](http://git-scm.com/) and managing the build process of your [application](https://deis.com/docs/workflow/reference-guide/terms/#application).

The builder process:

1. Receives incoming git push requests over SSH
2. Authenticates the user via SSH key
3. Authorizes the user's access to push code to the application
4. Starts the application build phase
5. Triggers a new [release](https://deis.com/docs/workflow/reference-guide/terms/#release) via the controller

Builder currently supports both buildpack and Dockerfile based builds.

For buildpack deploys, the builder component will launch a one-shot pod in the `deis` namespace. This pod runs the [slugbuilder](https://github.com/deis/slugbuilder) component which handles default and custom buildpacks.

The compiled application is a *slug*, consisting of your application code and all of its dependencies, as determined by the buildpack. The slug is pushed to the cluster-configured object storage for later execution.

For Dockerfile applications (i.e. apps with a Dockerfile in the root of the repository) builder will instead launch the [dockerbuilder](https://github.com/deis/dockerbuilder) component to package your application.

Instead of generating a slug, dockerbuilder generates a Docker image (using the underlying Docker engine). The completed image is pushed to the managed Workflow Docker registry. For more information, see [the docs on using Dockerfiles](https://deis.com/docs/workflow/applications/using-dockerfiles/).

### Slugrunner

The [slugrunner component](https://github.com/deis/slugrunner) is responsible for executing buildpack applications. Slugrunner receives slug information from the controller and downloads the application slug from the Workflow’s object storage before launching your application processes.

### Object Storage

Some Workflow components ship their persistent data to cluster-level object storage.

Currently, three components use Object Storage:

1. The database stores its WAL files
2. The registry stores Docker images
3. The slugbuilder stores slugs

Workflow ships with [Minio](https://deis.com/docs/workflow/understanding-workflow/components/#object-storage) by default which is not configured for persistence, and provides in-cluster, ephemeral object storage. Hence: if the Minio server crashes, all data will be lost *and cannot be regenerated*. We recommend you use this for dev environments only.

If you feel comfortable using Kubernetes persistent volumes you may configure Minio to use the persistent storage available in your environment. However, if you tie Minio to your Kubernetes persistent storage, your Workflow object storage is now dependant on that volume being available, not having network issues, and so on.

Workflow also supports [off-cluster storage](https://deis.com/docs/workflow/installing-workflow/configuring-object-storage/), which should be more reliable, and this is our recommended production deployment configuration. Off-cluster object storage currently supports Google Cloud Storage, AWS S3, and Azure Storage.

### Registry

The [registry component](https://github.com/deis/registry) is a managed Docker registry which holds application images generated by the builder component. The registry persists Docker images to Workflow object storage.

### Router

The [router component](https://github.com/deis/router) is based on[ nginx](http://nginx.org/) and is responsible for routing inbound HTTP(S) traffic to your applications. The default Workflow charts provision a Kubernetes service in the `deis` namespace with a service type of `LoadBalancer`.

Depending on your Kubernetes configuration, this may provision a cloud-based load balancer automatically if that is supported by your cloud provider.

The router component uses Kubernetes [annotations](http://kubernetes.io/docs/user-guide/annotations/) for both application discovery and router configuration.

### Logger

The logging subsystem has two components:

1. The [fluentd](https://github.com/deis/fluentd) component handles log shipping
2. The [logger](https://github.com/deis/logger) component maintains a [ring-buffer](https://en.wikipedia.org/wiki/Circular_buffer) of application logs

The fluentd component is deployed to your Kubernetes cluster via [daemon sets](http://kubernetes.io/docs/admin/daemons/#what-is-a-daemon-set). fluentd subscribes to all container logs, decorates the output with Kubernetes metadata, and can be configured to drain logs to multiple destinations. By default, fluentd ships logs to the logger component, which powers `deis logs`.

The logger component receives log streams from fluentd, collating by application name. The logger does not persist logs to disk, instead maintaining an in-memory ring-buffer.

Such setup will allow the end user to configure multiple destinations such as [Elasticsearch](https://www.elastic.co/) and other syslog compatible endpoints like[ Papertrail](http://papertrailapp.com/). See the docs on how to set up [off-cluster logging](https://deis.com/docs/workflow/managing-workflow/platform-logging/#default-configuration).

### Workflow Manager

The [Workflow manager component](https://github.com/deis/workflow-manager) will regularly check your Workflow cluster against the latest stable release of Deis Workflow components. If components are missing due to failure or are out of date, Workflow operators will be notified.

### Workflow CLI

The [Workflow CLI component](https://github.com/deis/workflow-cli) provides the `deis` command line utility used to interact with the Workflow platform.

Running `deis help` will give you a up-to-date list of `deis` commands. To learn more about a command run `deis help <command>`.

## Wrap Up

In this post we gave an overview of the Deis Workflow architecture, including: platform services, Kubenetes native functionality, application layout, edge routing, and topologies. We also took a look at how Workflow is composed of many independent components.

In the next and final post in this series, we'll install Workflow.
