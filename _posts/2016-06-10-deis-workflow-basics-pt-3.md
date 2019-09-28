---
layout: post
title: Deis Workflow Basics, Part Three
description: "Part three of our Deis Workflow overview. We show you how to install Workflow and deploy your first app."
tags:
  - Deis Workflow
  - "Series: Deis Workflow Basics"
author: rimas
---

This is part three of a three part miniseries looking at Deis Workflow, the open source Platform as a Service built on top of Kubernetes.

In [part one](https://deis.com/blog/2016/deis-workflow-basics-pt-1/), we took a look at some basic concepts: Twelve-Factor apps, Docker, Kubernetes, and the basics of Workflow. In [part two](https://deis.com/blog/2016/deis-workflow-basics-pt-2/), we took a look at Workflow as a system. Both it's architecture and modular composability.

In this post, we're going to install and use Workflow.

<!--more-->

## Cluster Setup

Before you continue, you need a Kubernetes cluster.

We're going to be using the [Helm Classic](https://helm.sh/) package manager for Kubernetes. Check out my previous post for [more information about Helm Classic](https://deis.com/blog/2016/kubernetes-overview-pt-2/).

We'll use Helm Classic to install Deis Workflow on a Kubernetes cluster. If you don't have a cluster up and running that you can use for this tutorial, you can use [the GKE cluster I show you how to set up](https://deis.com/blog/2016/first-kubernetes-cluster-gke/) in my previous post.

Got a cluster you can use?

Cool. Let's install some stuff on it.

## Helm Classic CLI

First, we must install the Helm Classic CLI.

There are two different sets of commands you must run, depending on how you're approaching this tutorial. If you're using GKE, you have access to the Google Cloud Shell. If you're not using GKE you'll just be using your regular shell.

If you're using Google Cloud Shell, run:

```
$ mkdir ~/bin && export PATH=${HOME}/bin:$PATH
$ cd ~/bin
$ curl -sSL https://get.helm.sh | bash
```

Google Cloud Shell puts `~/bin` on your path, so all we did here is is create that directory and install Helm there.

If you're *not* using Google Cloud Shell, run:

```
$ curl -s https://get.helm.sh | bash
```

## Workflow CLI

Next, we install the Deis Workflow CLI.

Using Google Cloud Shell, run:

```
$ cd ~/bin
$ curl -sSL http://deis.io/deis-cli/install-v2.sh | bash
```

Otherwise, run:

```
$ curl -sSL http://deis.io/deis-cli/install-v2.sh | bash
```

## Workflow

Now we need to add the Deis [chart repository](http://helm.readthedocs.io/en/latest/chart_tables/) to Helm, which will allow us to install the `workflow-v2.4.2` chart with a single command.

Add the chart repository like so:

```
$ helmc repo add deis https://github.com/deis/charts
---> Checking things locally…
---> Creating /home/user/.helmc/config.yaml
---> Everything looks good! Happy helming!
---> Continuing onwards and upwards!
---> Cloning into '/home/user/.helmc/cache/deis'...
---> Hooray! Successfully added the repo.
```

Update your charts:

```
$ helmc up
---> Checking repository charts
---> Cloning into '/home/user/.helmc/cache/charts'...
Already up-to-date.
---> Checking repository deis
Already up-to-date.
---> Done
```

Now fetch the Workflow chart:

```
$ helmc fetch deis/workflow-v2.4.2
---> Fetched chart into workspace /home/user/.helmc/workspace/charts/workflow-v2.4.2
---> Done
```

Note: for production deployments be sure to [configure off-cluster object storage](http://docs-v2.readthedocs.org/en/latest/installing-workflow/configuring-object-storage/) before you run the `helmc generate` command. If you don't, your object storage will be ephemeral and you could lose data.

Generate the [Kubernetes secrets](http://kubernetes.io/docs/user-guide/secrets/) Workflow needs:

```
$ helmc generate -x manifests workflow-v2.4.2
---> Ran 18 generators.
```

Now, we can install Workflow:

```
$ helmc install workflow-v2.4.2
---> Running `kubectl create -f` ...
namespace "deis" created
secret "builder-ssh-private-keys" created
secret "builder-key-auth" created
secret "django-secret-key" created
secret "database-creds" created
secret "objectstorage-keyfile" created
secret "deis-router-dhparam" created
serviceaccount "deis-builder" created
serviceaccount "deis-controller" created
serviceaccount "deis-database" created
serviceaccount "deis-logger-fluentd" created
serviceaccount "deis-logger" created
serviceaccount "deis-monitor-telegraf" created
serviceaccount "deis-registry" created
serviceaccount "deis-router" created
serviceaccount "deis-workflow-manager" created
service "deis-builder" created
service "deis-controller" created
service "deis-database" created
service "deis-logger" created
service "deis-monitor-grafana" created
service "deis-monitor-influxapi" created
service "deis-monitor-influxui" created
service "deis-monitor-stdout" created
service "deis-registry" created
service "deis-router" created
service "deis-workflow-manager" created
replicationcontroller "deis-builder" created
replicationcontroller "deis-controller" created
replicationcontroller "deis-database" created
replicationcontroller "deis-logger" created
replicationcontroller "deis-monitor-grafana" created
replicationcontroller "deis-monitor-influxdb" created
replicationcontroller "deis-monitor-stdout" created
replicationcontroller "deis-registry" created
replicationcontroller "deis-router" created
replicationcontroller "deis-workflow-manager" created
daemonset "deis-logger-fluentd" created
daemonset "deis-monitor-telegraf" created
========================================
# Workflow 2.0.0
```

Awesome. Workflow is installed.

Let's take a look at what Kubernetes is up to.

```
$ kubectl --namespace=deis get pods -w
NAME                          READY     STATUS              RESTARTS   AGE
deis-builder-ljlgm            0/1       Running             0          49s
deis-controller-lty7q         0/1       Running             0          48s
deis-database-hbjmj           0/1       ContainerCreating   0          48s
deis-logger-fluentd-7xbzo     0/1       ContainerCreating   0          46s
deis-logger-fluentd-ijx5j     0/1       ContainerCreating   0          46s
deis-logger-fluentd-ud8n4     0/1       ContainerCreating   0          46s
deis-logger-jlo2u             1/1       Running             0          48s
deis-monitor-grafana-67f17    0/1       ContainerCreating   0          47s
deis-monitor-influxdb-wawqb   0/1       ContainerCreating   0          47s
deis-monitor-stdout-a4ko1     0/1       ContainerCreating   0          47s
deis-monitor-telegraf-eshsg   0/1       ContainerCreating   0          47s
deis-monitor-telegraf-sbnif   0/1       ContainerCreating   0          47s
deis-monitor-telegraf-vzcl3   0/1       ContainerCreating   0          47s
deis-minio-jy9rx              0/1       ContainerCreating   0          47s
deis-registry-u3xip           0/1       ContainerCreating   0          47s
deis-router-go5im             0/1       ContainerCreating   0          47s
deis-workflow-manager-rdkfy   0/1       ContainerCreating   0          47s
```

The `-w` flag we used here allows us to watch for status updates.

Watch until all Workflow components are in the `Running` state. This means Workflow is ready to use.

After a while, you should see this:

```
NAME                          READY     STATUS    RESTARTS   AGE
deis-builder-ljlgm            1/1       Running   0          2m
deis-controller-lty7q         1/1       Running   1          2m
deis-database-hbjmj           1/1       Running   0          2m
deis-logger-fluentd-7xbzo     1/1       Running   0          2m
deis-logger-fluentd-ijx5j     1/1       Running   0          2m
deis-logger-fluentd-ud8n4     1/1       Running   0          2m
deis-logger-jlo2u             1/1       Running   0          2m
deis-monitor-grafana-67f17    1/1       Running   0          2m
deis-monitor-influxdb-wawqb   1/1       Running   0          2m
deis-monitor-stdout-a4ko1     1/1       Running   0          2m
deis-monitor-telegraf-eshsg   1/1       Running   0          2m
deis-monitor-telegraf-sbnif   1/1       Running   0          2m
deis-monitor-telegraf-vzcl3   1/1       Running   0          2m
deis-minio-jy9rx			  1/1       Running   0          2m
deis-registry-u3xip           1/1       Running   0          2m
deis-router-go5im             1/1       Running   0          2m
deis-workflow-manager-rdkfy   1/1       Running   0          2m
```

Now we need to get the Workflow router's external IP address. We'll use this to access our Workflow cluster.

```
$ kubectl --namespace=deis get svc deis-router
NAME          CLUSTER_IP       EXTERNAL_IP      PORT(S)
deis-router   10.175.247.246   104.197.212.37   80/TCP,443/TCP,2222/TCP,9090/TCP
```

Specifically here, we need the `EXTERNAL_IP` value. When you run this command, you will see a different IP address here. For the rest of this post, I will refer to your value of this as `YOUR_EXTERNAL_IP`. When copying a command that uses this reference, be sure to replace the value!

## Register and Set Up SSH Keys

To use Workflow, you must first register a user with the [controller](http://docs.deis.io/en/latest/reference/terms/controller/#controller).

For the purposes of this post, we're not going to set DNS records for our Workflow router's external IP address. But we are going going to use [nip.io](http://nip.io/), which is a simple wildcard DNS for any IP address.

We will use `deis register` with the controller URL (`YOUR_EXTERNAL_IP` we got above) to create a new account.

Do that like so:

```
$ deis register http://deis.YOUR_EXTERNAL_IP.nip.io
username: USERNAME
password:
password (confirm):
email: USERNAME@EXAMPLE.COM
```

If the registration is successful you should see:

```
Registered USERNAME
Logged in as USERNAME
```

Now you need to upload your SSH public key so you can use `git push` to deploy applications to Deis Workflow.

If you do not have an SSH key you must generate one.

This includes Google Cloud Shell users, as Google Cloud Shell does not come with an SSH key pre-generated for you.

Generate a key like so:

```
$ ssh-keygen -t rsa -b 4096 -C "USERNAME@EXAMPLE.COM"
```

Press *enter* to accept the default location and filename. You can then press *enter* two more times if you want to use an empty passphrase. Or set a passphrase. Up to you.

Once you're done, change the file permissions:

```
$ chmod 600 ~/.ssh/id_rsa.pub
```

You're done!

Now, upload your SSH public key:

```
$ deis keys:add
Found the following SSH public keys:
1) id_rsa.pub your_email@example.com
0) Enter path to pubfile (or use keys:add <key_path>)
Which would you like to use with Deis? 1
Uploading /Users/myuser/.ssh/id_rsa.pub to Deis... done
```

## Deploying Applications

Let's make a folder where we are going to store our applications:

```
$ mkdir ~/myapps
$ cd ~/myapps
```

Workflow can deploy any application or service that can run inside a Docker container and uses HTTP port 80.

To be scaled horizontally, application containers must be stateless. Apps can store state in an external backing services. This is one component of the [Twelve-Factor](http://12factor.net/) approach.

Workflow supports three ways of building applications:

1. [Heroku Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
2. [Dockerfiles](https://docs.docker.com/reference/builder/)
3. [Docker Images](https://docs.docker.com/introduction/understanding-docker/)

We'll look at one example of each.

### Buildpacks

Heroku buildpacks are useful if you want to follow Heroku's best practices for building applications. They're also useful if you're porting an application from Heroku.

We have an example buildpack app to demonstrate this workflow.

First, clone the app:

```
$ git clone https://github.com/deis/example-go
$ cd example-go
```

Now we can create the app:

```
$ deis create example-go
Creating Application... done, created example-go
Git remote deis added
remote available at ssh://git@deis.YOUR_EXTERNAL_IP.nip.io:2222/example-go.git
```

And finally, deploy the app:

```
$ git push deis master
Counting objects: 80, done.
Compressing objects: 100% (75/75), done.
Writing objects: 100% (80/80), 17.83 KiB | 0 bytes/s, done.
Total 80 (delta 33), reused 0 (delta 0)
Starting build... but first, coffee!
...
...
2016/04/25 18:06:43 Successfully copied home/example-go:git-779ca3d7/tar to /tmp/slug.tgz
-----> Go app detected
-----> Checking Godeps/Godeps.json file.
-----> Installing go1.6... done
-----> Running: go install -v -tags heroku .
github.com/deis/example-go
-----> Discovering process types
       Procfile declares types -> web
-----> Compiled slug size is 2.2M
2016/04/25 18:06:52 Successfully copied home/example-go:git-779ca3d7/push/slug.tgz to /tmp/slug.tgz
2016/04/25 18:06:52 Successfully copied home/example-go:git-779ca3d7/push/Procfile to /tmp/build/Procfile
Build complete.
Launching App...
Done, example-go:v2 deployed to Deis

Use 'deis open' to view this application in your browser
To learn more, use 'deis help' or visit http://deis.io
To ssh://git@deis.YOUR_EXTERNAL_IP.nip.io:2222/example-go.git
 * [new branch]      master -> master
```

If you're using the Google Cloud Shell, you cannot use `deis open` as suggested by the output, because there is no available browser in that environment. But you can open the URL by copying it into your regular browser or by using `curl`, like so:

```
$ curl -s http://example-go.YOUR_EXTERNAL_IP.nip.io
Powered by Deis!
```

Because Workflow detected a Heroku-style application, we have one web process running by default on the first deploy.

You can scale up to three web processes like so:

```
$ deis scale web=3
Scaling processes... but first, coffee!
done in 116s
=== example-go Processes
--- web:
example-go-v2-web-0lktd up (v2)
example-go-v2-web-2974g up (v2)
example-go-v2-web-jmloh up (v2)
```

Scaling a process type directly changes the number of Kubernetes replicas running that process. You can check this directly:

```
$ kubectl --namespace=example-go get rc
NAME                DESIRED   CURRENT   AGE
example-go-v2-web   3         3         11m
```

Ok, now let's change an environment variable.

This will trigger a new release.

```
$ deis config:set POWERED_BY=Kubernetes
Creating config... done
=== example-go Config
POWERED_BY      Kubernetes
```

This example app is configured to display a different message depending on the value of the `POWERED_BY` environment variable. So the new version of our app should have a new message.

Let's check it out:

```
$ curl -s http://example-go.YOUR_EXTERNAL_IP.nip.io
Powered by Kubernetes!
```

Great!

Each time Workflow makes a deploy, it versions the release.

Here's how to see app releases:

```
$ deis releases
=== example-go Releases
v3      2016-04-25T18:28:05UTC  rimas added POWERED_BY
v2      2016-04-25T18:06:53UTC  rimas deployed 779ca3d
v1      2016-04-25T18:02:55UTC  rimas created initial release
```

We can revert back to the previous version easily:

```
$ deis releases:rollback
Rolling back one release... done, v4
```

Why does this say "v4"? Because our app rolled back to the same state as v2, but this counts as a new release. Release v4 is identical to v2, except for the timestamp.

We can also specify which release to roll back to:

```
$ deis releases:rollback v3
Rolling back to v3... done, v5
```

If we check for releases now, we should see 5 versions there:

```
$ deis releases
=== example-go Releases
v5      2016-04-29T14:59:58Z    rimas rolled back to v3
v4      2016-04-29T14:56:13Z    rimas rolled back to v2
v3      2016-04-29T14:53:00Z    rimas added POWERED_BY
v2      2016-04-29T14:45:44Z    rimas deployed 779ca3d
v1      2016-04-29T14:44:02Z    rimas created initial release
```

Cool, huh?

We created an application, scaled it, changed our config, redeployed it, rolled it back, and so on. And no Kubernetes knowledge was needed at all!

To learn more about this workflow, check out [the official docs](https://deis.com/docs/workflow/applications/using-buildpacks/).

### Dockerfile Application

Workflow supports deploying applications via Dockerfiles. A[ Dockerfile](https://docs.docker.com/reference/builder/) automates the steps for crafting a[ Docker Image](https://docs.docker.com/introduction/understanding-docker/). Dockerfiles are very powerful but require some extra work to define your application runtime environment.

Let's deploy a Dockerfile based application with a backend service.

We will install the backend service with Helm.

#### Backend

Add the chart repository to Helm:

```
$ helmc repo add demo-charts https://github.com/deis/demo-charts
---> Cloning into '/home/user/.helmc/cache/demo-charts'...
---> Hooray! Successfully added the repo.
```

Update Helm:

```
$ helmc up
---> Checking repository charts
Already up-to-date.
---> Checking repository deis
Already up-to-date.
---> Checking repository demo-charts
Already up-to-date.
---> Done
```

Fetch and install the backend chart:

```
$ helmc fetch demo-charts/redis-guestbook
---> Fetched chart into workspace /home/rimas/.helmc/workspace/charts/redis-guestbook
---> Done
```

Install the chart:

```
$ helmc install redis-guestbook
---> Running `kubectl create -f` ...
service "redis-master" created
service "redis-slave" created
deployment "redis-master" created
deployment "redis-slave" created
---> Done
========================================
# Redis Cluster for Guestbook App
This is a cluster of Redis master with a replicated set of slaves
========================================
```

This should have installed a Redis cluster for us.

Check the Redis cluster is running:

```
$ kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-4q5c0   1/1       Running   0          21m
redis-slave-2zo1t    1/1       Running   0          21m
redis-slave-prkcj    1/1       Running   0          21m
```

Everything looks great!

Let's move on to installing the frontend.

#### Frontend

Clone the app repository:

```
$ cd ~/myapps
$ git clone https://github.com/deis/example-guestbook.git
$ cd example-guestbook
```

Create the app:

```
$ deis create guestbook
Creating Application... done, created guestbook
Git remote deis added
remote available at ssh://git@deis.YOUR_EXTERNAL_IP.nip.io:2222/guestbook.git
```

Before we continue, we need to set some environment variables so our app knows where to find the backend Redis cluster.

```
$ deis config:set GET_HOSTS_FROM=env -a guestbook
Creating config... done
=== guestbook2 Config
GET_HOSTS_FROM                 env
```

```
$ deis config:set REDIS_MASTER_SERVICE_HOST=redis-master.default  -a guestbook
Creating config... done
=== guestbook2 Config
REDIS_MASTER_SERVICE_HOST      redis-master.default
```

```
$ deis config:set REDIS_SLAVE_SERVICE_HOST=redis-slave.default -a guestbook
Creating config... done
=== guestbook2 Config
REDIS_SLAVE_SERVICE_HOST       redis-slave.default
```

Now that's done, you can push the app:

```
$ git push deis master
Counting objects: 28, done.
Compressing objects: 100% (27/27), done.
Writing objects: 100% (28/28), 8.20 KiB | 0 bytes/s, done.
Total 28 (delta 12), reused 0 (delta 0)
Starting build... but first, coffee!
...
...
...
download tar file complete
extracting tar file complete
Step 1 : FROM php:5-apache
 ---> 1ed0a64c9c1b
Step 2 : RUN apt-get update &&     apt-get install -y php-pear
 ---> Running in 81fb6bd0a928
Get:1 http://security.debian.org jessie/updates InRelease [63.1 kB]
Get:2 http://security.debian.org jessie/updates/main amd64 Packages [295 kB]
Ign http://httpredir.debian.org jessie InRelease
Get:3 http://httpredir.debian.org jessie-updates InRelease [142 kB]
Get:4 http://httpredir.debian.org jessie Release.gpg [2373 B]
Get:5 http://httpredir.debian.org jessie Release [148 kB]
Get:6 http://httpredir.debian.org jessie-updates/main amd64 Packages [9283 B]
Get:7 http://httpredir.debian.org jessie/main amd64 Packages [9034 kB]
Fetched 9694 kB in 3s (3171 kB/s)Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:  libjson-c2 libonig2 libperl4-corelibs-perl libqdbm14 lsof php5-cli  php5-common php5-json php5-readline psmisc ucf
Suggested packages:  php5-dev php5-user-cache
The following NEW packages will be installed:  libjson-c2 libonig2 libperl4-corelibs-perl libqdbm14 lsof php-pear php5-cli  php5-common php5-json php5-readline psmisc ucf
0 upgraded, 12 newly installed, 0 to remove and 2 not upgraded.Need to get 4007 kB of archives.After this operation, 15.0 MB of additional disk space will be used.
Get:1 http://httpredir.debian.org/debian/ jessie/main libjson-c2 amd64 0.11-4 [24.8 kB]
Get:2 http://httpredir.debian.org/debian/ jessie/main libonig2 amd64 5.9.5-3.2 [118 kB]
Get:3 http://httpredir.debian.org/debian/ jessie/main libperl4-corelibs-perl all 0.003-1 [43.6 kB]
Get:4 http://httpredir.debian.org/debian/ jessie/main lsof amd64 4.86+dfsg-1 [316 kB]
Get:5 http://httpredir.debian.org/debian/ jessie/main ucf all 3.0030 [69.7 kB]
Get:6 http://httpredir.debian.org/debian/ jessie/main libqdbm14 amd64 1.8.78-5+b1 [118 kB]
Get:7 http://httpredir.debian.org/debian/ jessie/main psmisc amd64 22.21-2 [119 kB]
Get:8 http://httpredir.debian.org/debian/ jessie/main php5-common amd64 5.6.19+dfsg-0+deb8u1 [719 kB]
Get:9 http://httpredir.debian.org/debian/ jessie/main php5-json amd64 1.3.6-1 [18.6 kB]
Get:10 http://httpredir.debian.org/debian/ jessie/main php5-cli amd64 5.6.19+dfsg-0+deb8u1 [2178 kB]
Get:11 http://httpredir.debian.org/debian/ jessie/main php-pear all 5.6.19+dfsg-0+deb8u1 [268 kB]
Get:12 http://httpredir.debian.org/debian/ jessie/main php5-readline amd64 5.6.19+dfsg-0+deb8u1 [12.5 kB]
...
remote: ":"git-44664369: digest: sha256:5fe3981c2a8387dc7eacb327d3c791d1a03424d51abf67eaa18ce34f263896bd size: 56767"}
Build complete.
...
...
Launching App…
...
Done, guestbook:v3 deployed to Deis
Use 'deis open' to view this application in your browser
To learn more, use 'deis help' or visit http://deis.io
To ssh://git@deis.YOUR_EXTERNAL_IP.nip.io:2222/guestbook.git
 * [new branch]      master -> master
```

The docker image was built and pushed to Workflow's Docker registry.

Now, visit this URL, using the IP address in the command output:

```
http://guestbook.YOUR_EXTERNAL_IP.nip.io/
```

You should see something like this:

![](/images/blog-images/deis-workflow-basics-pt-3-0.png)

All done!

We deployed our frontend app with Workflow and our backend service with Helm. To learn more about this workflow, check out [the official docs](https://deis.com/docs/workflow/applications/using-dockerfiles/).

### Docker Image Application

Workflow supports deploying applications via an existing[ Docker Image](https://docs.docker.com/introduction/understanding-docker/) as well.

To demo this, we will reuse the same Redis backend.

We create the app with Workflow as before, but we use the `--no-remote` flag, as we do not need a Git remote set if we're using pre-built Docker images.

```
$ cd ~/myapps
$ deis create guestbook2 --no-remote
Creating Application... done, created guestbook2
If you want to add a git remote for this app later, use `deis git:remote -a guestbook2`
```

Because we're using the same backend Redis cluster, we set the same environment variables needed by our frontend app.

```
$ deis config:set GET_HOSTS_FROM=env -a guestbook2
Creating config... done
=== guestbook2 Config
GET_HOSTS_FROM                 env
```

```
$ deis config:set REDIS_MASTER_SERVICE_HOST=redis-master.default  -a guestbook2
Creating config... done
=== guestbook2 Config
REDIS_MASTER_SERVICE_HOST      redis-master.default
```

```
$ deis config:set REDIS_SLAVE_SERVICE_HOST=redis-slave.default -a guestbook2
Creating config... done
=== guestbook2 Config
REDIS_SLAVE_SERVICE_HOST       redis-slave.default
```

Now, pull the pre-built Docker image:

```
$ deis pull quay.io/deis/example-guestbook -a guestbook2
Creating build... done
```

Now our Workflow app is using the pre-built Docker image we pulled from [Quay](https://quay.io/). This is an alternative to to working with apps that need to be built from Git repositories. To learn more about this workflow, check out [the official docs](https://deis.com/docs/workflow/applications/using-docker-images/).

Open this URL, using your external IP address:

```
http://guestbook2.YOUR_EXTERNAL_IP.nip.io/
```

And you should see something like this:

![](/images/blog-images/deis-workflow-basics-pt-3-0.png)

It worked!

Because both your frontend apps are using the same Redis backend cluster, you will see the same guestbook messages in both apps.

## Wrap Up

In this series of posts we looked at the [basic concepts](https://deis.com/blog/2016/deis-workflow-basics-pt-1/) needed to understand Deis Workflow. We looked at the [architecture and modular composability](https://deis.com/blog/2016/deis-workflow-basics-pt-2/) of Workflow. And we got our hands dirty, installing and using Workflow.

Workflow provides a simple and easy to use Kubernetes-native PaaS with a Heroku-inspired workflow. With very little Kubernetes experience, you can manage, deploy, version, and scale apps with ease.

And with Helm, the package manager for Kubernetes, you can install common backend services without the hassle of manual setup.

