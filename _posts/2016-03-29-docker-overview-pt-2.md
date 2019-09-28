---
layout: post
title: Docker Overview, Part Two
description: "New to Docker? In this overview, we take a look at building images, launching containers, and persisting data."
tags:
  - "Series: Docker Overview"
  - Docker
author: rimas
---

In [part one](/blog/2016/docker-overview-pt-1/) of this miniseries looking at Docker, we looked at what makes Docker special, the difference between Virtual Machines and containers, and the primary components that make up Docker.

In this post, we'll work directly with some containers. Specifically, we'll show you hot to launch a container, how to build an image with a Dockerfile, how to work with registries, and the basics of data volumes.

## Launching Containers

Before launching a container you might pull it from the registry:

```
$ docker pull alpine
```

<!--more-->

Launching a container is as simple as running:

```
$ docker run <image name> <command>
```

The `command` here is the command you want to run inside the container.

If the image doesn't exist, Docker attempts to fetch it from the public image registry. This happens automatically, but you should expect a time delay.

It's important to note that containers are designed to stop after the command executed within them has exited. For example, if you run `/bin/echo hello world` as your command, the container starts, prints "hello world" and then stops.

For example, run:

```
$ docker run alpine /bin/echo hello world
```

You should see something like this:

```
$ docker run alpine /bin/echo hello world
Unable to find image 'alpine:latest' locally
latest: Pulling from alpine
31f630c65071: Pull complete
Digest: sha256:074de05bbd8554cf454a19094df34985c8674f54e14474731427a2a31d1970ec
Status: Downloaded newer image for alpine:latest
hello world
```

Let's launch an [Alpine Linux](http://www.alpinelinux.org/) based container and install `openssh` inside of it using the ash prompt. First, run this:

```
$ docker run -t -i alpine /bin/ash
```

The `-t` and `-i` flags allocate a pseudo-TTY and keep STDIN open even if not attached. This allows you to use the container like a traditional VM as long as the ash prompt is running.

Then install `openssh` like so:

```
apk update && apk add openssh
```

If we exit the container, the changes we made to the disk are not installed. So next time we launch the container, `openssh` will not be installed.

If we want to save our changes, we have to *commit* them.

But first we need to find the container ID. To do this, open a new shell in your terminal and look for the ID:

```
$ docker ps
```

You should see something like this:

```
$ docker ps
CONTAINER ID    IMAGE     COMMAND       CREATED           STATUS
2511d433bb01    alpine    "/bin/ash"    44 minutes ago    Up 44 minutes
```

Now we can commit our container:

```
$ docker commit 2511d433bb01 my/alpine
```

Here, `2511d433bb01` is our container ID, and `my/alpine` is our repository.

You can check this worked by running:

```
docker images
```

You should see something like this:

```
$ docker images
REPOSITORY     TAG        IMAGE ID         CREATED             VIRTUAL SIZE
my/alpine      latest     2511d433bb01     About a minute ago  9.501 MB
```

## Building Images With Dockerfiles

Committing a running container is fine for experimentation. But, for production scenarios, it is much better to use Dockerfiles where you can record the necessary commands as instructions for the `docker build` command. These files can then be versioned in Git. The Docker daemon runs your commands one-by-one, committing the result to a new image if necessary, before finally outputting the ID of your new image.

Create a file called Dockerfile, and put this in it:

```
# base image
FROM alpine

# grab ca-certificates and nginx
RUN apk update && apk add ca-certificates nginx

# expose ports 80 and 443
EXPOSE 80 443

# start up nginx
CMD ["nginx", "-g", "'pid /tmp/nginx.pid; daemon off;'"]
```

Now, let’s build the Docker image:

```
$ docker build -t mynginx .
```

Here, the `-t mynginx` argument names the image `mynginx`.

Now we can run it:

```
$ docker run -p 8080:80 mynginx
```

The `-p 8080:80` argument maps port 8080 on the local host to port 80 of the container.

In your browser, navigate to:

<pre>
http://<b>&lt;ip_of_your_docker_host&gt;</b>:8080
</pre>

And you should see:

![running nginx in docker](/images/blog-images/docker-overview-2-1.png)

We built a simple nginx image!

As the base image, we used Alpine Linux. We installed nginx, then we exposed ports 80 and 443. Finally, we ran nginx.

The reason for using Alpine Linux is that it has very small footprint. This makes the nginx image much smaller and quicker to build and also quicker to push and to pull from the registry.

For more information about Dockerfile commands, see [the documentation](https://docs.docker.com/articles/dockerfile_best-practices/).

## Working With a Registry

Now that we know how to build Docker images, let’s learn how to push images to the Docker registry.

In this example, we’ll use the [Docker Hub](https://hub.docker.com) registry.

You need to open an account there to be able to push Docker images. For more information, see [the user guide](https://docs.docker.com/userguide/dockerrepos/).

In the previous example, we used this command to build the Docker image:

```
docker build -t mynginx .
```

That command builds the Docker image, but because it has no repository name, it can only be used on the host it was built on.

To push to the Docker Hub registry (or any other hosted registry) you need to add your username to the Docker image name:

<pre>
$ docker build -t <b>yourname</b>/mynginx .
</pre>

Before using docker push, make sure that you have done `docker login`. This will save your connection settings to the `.dockercfg` file in your home folder.

To push to the Docker registry, run:

<pre>
$ docker push <b>yourname</b>/mynginx
</pre>

To pull from the Docker registry, run:

<pre>
$ docker pull <b>yourname</b>/mynginx
</pre>

If you have self-hosted a private Docker registry, the commands look like this:

<pre>
$ docker push <b>your_registry</b>:5000/mynginx
</pre>

<pre>
$ docker pull <b>your_registry</b>:5000/mynginx
</pre>

## Data Persistence

Docker containers are stateless. However, in some cases, such as when using a database, you need to to have persistent data. Docker supports two ways to persist data: data volumes and data volume containers. We'll cover both now.

### Data Volumes

A data volume is a specially designated directory within one or more containers that bypasses the [Union file system](https://docs.docker.com/reference/glossary#union-file-system).

Data volumes provide several useful features for persistent or shared data:

* Volumes are initialized when a container is created. If the container’s base image contains data at the specified mount point, that existing data is copied into the new volume upon volume initialization.
* Data volumes can be shared and reused among containers.
* Changes to a data volume are made directly.
* Data volumes are uneffected by updates to the image, even if those updates modify paths that would live at the same location as the mounted data volume.
* Data volumes persist even if the container itself is deleted.

Data volumes are designed to persist data, independent of the container’s life cycle. Docker therefore never automatically deletes volumes when you remove a container, nor does it "garbage collect" volumes that are no longer referenced by a container.

You can add a host folder as a data volume into a container using the `-v` flag with the `docker run` command. You can use the `-v` multiple times to mount multiple data volumes.

Let’s mount a single volume in our web application container:

<pre>
$ docker run -p 8080:80 \
    -v <b>/data/share/nginx/html</b>:<b>/usr/share/nginx/html</b> mynginx
</pre>

The command above mounts a host folder `/data/share/nginx/html` into the container at `/usr/share/nginx/html` folder. Anything we put to the host folder is seen instantly by the container.

Note: if the path `/usr/share/nginx/html` already exists inside the container’s image, its contents are replaced by the contents of `/data/share/nginx/html` on the host in order to stay consistent with the expected behavior of mount.

### Data Volume Containers

If you have some persistent data that you want to share between containers or want to use from non-persistent containers, it’s best to create a named *data volume container*, and then to mount the data from it. This avoids potential permissions problems that might arise from mounting a host directory on some Linux OSes.

Let’s create a new, named container with a volume to share. We use the busybox image because the other official images, like the postgres image, are themselves ​based​ off of the busybox image. Because our images then have Union FS layers in common, Docker can save disk space by only keeping one copy of those layers.

Run this command:

```
$ docker create -v /dbdata --name dbdata busybox true
```

This creates a data volume container called dbdata, using the busybox base image, and creates a data directory at `/dbdata`.

You can then use the `--volumes-from` flag to mount the `/dbdata` volume in another container:

```
$ docker run -d --volumes-from dbdata --name postgres postgres
```

In this case, if the postgres image contained a directory called `/dbdata` then mounting the volumes from the dbdata container hides the `/dbdata` files from the postgres image. The result is only the files from the dbdata container are visible.

You can use multiple `--volumes-from` parameters to bring together multiple data volumes from multiple containers.

If you remove containers that mount volumes, including the initial dbdata container, the volumes will not be deleted. To delete the volume from disk, you must explicitly call `docker rm -v` against the last container with a reference to the volume. This allows you to upgrade, or effectively migrate data volumes between containers.

## Conclusion

In [part one](/blog/2016/docker-overview-pt-1/) of this miniseries we took a high level view of Docker, what makes it special, and how it's architected. In this post, we looked at building images and launching containers, layered filesystems, and how to persist data.

Stay tuned for a Kubernetes overview next!
