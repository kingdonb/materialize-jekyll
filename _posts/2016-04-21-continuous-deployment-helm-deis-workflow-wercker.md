---
layout: post
title: Continuous Deployment With Helm Classic, Deis Workflow, and Wercker
description: "Build a simple, multi-tier web application using Helm Classic, Deis Workflow, and Wercker for continuous deployment."
tags:
  - Deis Workflow
  - Helm
  - Wercker
  - Continuous Deployment
author: rimas
---

[Deis Workflow](https://deis.com/workflow/) is already in GA for a while. But what is it like to work with? Well, I created [an example repository](https://github.com/deis/example-guestbook-wercker) on GitHub to demo some functionality.

Using this example, we'll build a simple, multi-tier web application using [Helm Classic](https://helm.sh), [Deis Workflow](https://deis.com/workflow/), and [Wercker](http://wercker.com) for continuous deployment.

When we finish, we'll have:

- A backend [Redis](http://redis.io/) cluster (for storage)
- A web frontend (installed as a Deis Workflow app) that interacts with Redis via JavaScript
- [Wercker](http://wercker.com) for continuous deployment of your Docker image to Deis Workflow

<!--more-->

## Prerequisites

Before we continue, you'll need a few things set up.

Firstly, you need a running [Kubernetes](https://kubernetes.io) cluster that is accessible remotely so Wercker can deploy new versions of your Docker image.

Next, you'll need [Helm Classic](https://helm.sh) installed. Helm Classic is the Kubernetes package manager developed by the Deis folks.

You can install Helm Classic by running:

```
curl -s https://get.helm.sh | bash
```

Then you'll need to install Deis Workflow. Consult [the Deis docs](https://github.com/deis/workflow/blob/master/src/installing-workflow/installing-deis-workflow.md) for that!

We're also going to use [Docker Hub](https://hub.docker.com/) for hosting our Docker repository. So you'll need an account there.

And finally, head over to [Wercker](http://wercker.com) and set up an account.

## App Setup

### Install Redis With Helm Classic

As a quick refresher, a *chart* in Helm Classic lingo is a packaged unit of Kubernetes software. For the purposes of this demo, I wrote a chart that sets up Redis for you.

Point Helm Classic at the demo repository by running:

```nohighlight
$ helmc up
$ helmc repo add demo-charts https://github.com/deis/demo-charts
$ helmc up
```

Now install the chart I wrote for this demo that sets up Redis:

```
$ helmc fetch demo-charts/redis-guestbook
$ helmc install redis-guestbook
```

And done!

### Create the Deis App

To work with Deis Workflow, we need to create a Deis app.

You can do that by running:

```
$ deis create guestbook --no-remote
```

We're creating the app with `--no-remote` because Deis only needs a Git remote when we’re using it for building Docker images. But we’re using Wercker for that.

Once that's done, the only thing we need to do is specify some environment variables so that our app knows how to contact the Redis cluster.

Run this command:

```
$ deis config:set GET_HOSTS_FROM=env REDIS_MASTER_SERVICE_HOST=redis-master.default REDIS_SLAVE_SERVICE_HOST=redis-slave.default -a guestbook
```

Now that's set up, we can can set up the code.

### Set Up the Code

This bit's easy.

Fork [my demo repo](https://github.com/deis/example-guestbook-wercker.git) to your GitHub account.

Then, clone your fork locally:

<pre>
$ git clone https://github.com/<b>USERNAME</b>/example-guestbook-wercker.git
</pre>

Now, we can set up continuous deployment with Wercker so changes made to your fork will result in automatic deployments to your Deis Workflow.

### Set Up Continuous Deployment

Log into [your Docker Hub account](https://hub.docker.com/) and create a new repository. You can do that via the *Create* menu, then *Create Repository*, then call it something like `example-guestbook-wercker`, or whatever you want.

Log in to [your Wercker account](https://app.wercker.com/) and select *Create*, then *Application*. Connect Werker to your GitHub account and select your repository. When configuring access, select the default option: check out the code without using an SSH key.

Once the app is created, go to *Settings*, then *Environment Variables*, and create the following key-value pairs:

- DOCKER_USERNAME: Your Docker Hub or other hosted docker registry account name
- DOCKER_PASSWORD: Your Docker registry account password (you'll probably want to set this *protected*, which hides it from the UI)
- DOCKER_REPO: Your Docker Hub URL e.g. <code><b>USERNAME</b>/example-guestbook-wercker</code>
- DEIS_CONTROLLER: Your Deis controller URL (see your `~/.deis/client.json` file)
- DEIS_TOKEN: Your Deis user token (see your `~/.deis/client.json` file)
- TAG: A tag for tagging your Docker image, e.g. `latest`

These environment variables will then be passed into the [wercker.yml](https://github.com/deis/example-guestbook-wercker/blob/master/wercker.yml) file before it is evaluated by Wercker.

## Test It

Make some changes to your code. Then, push to GitHub.

Wercker will see this, and do the following:

- Build and tag your Docker image
- Push to your Docker registry
- Deploy your Docker image to Deis Workflow

There we have it.

Continuous deployment with Helm Classic, Deis Workflow, and Werker.

Stay tuned for more posts like this.
