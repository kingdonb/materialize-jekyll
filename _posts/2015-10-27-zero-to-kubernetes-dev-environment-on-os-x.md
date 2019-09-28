---
layout: post
title: Zero to Kubernetes Dev Environment on OS X
description: "How to quickly setup a local OS X development environment for Kubernetes."
tags:
  - Kubernetes
  - Development
  - Containers
author: rimas
date: "2015-10-27 00:00:00"
---

Many people we talk to are interested in experimenting with Kubernetes but find that putting together a development environment is daunting.

[*Kubernetes Solo OSX*](https://github.com/rimusz/kube-solo-osx) (Kube Solo) provides a lightweight, simple Kubernetes enviroment that is easy as a few clicks.

Kube Solo wraps [coreos-xhyve](https://github.com/coreos/coreos-xhyve) and runs in your Mac's status bar. With a few clicks Kube Solo provisions a CoreOS server and boostraps Kubernetes development environment.

Since Kube Solo is based on [xhyve](https://github.com/mist64/xhyve) there is no need to have VirtualBox and Vagrant installed on your Mac.

<!--more-->

## Getting Started

Kube Solo's only external dependency is [iTerm2](http://www.iterm2.com/#/section/downloads). So make sure you download and install iTerm first.

Download the latest release from [Kube Solo Releases](https://github.com/rimusz/kube-solo-osx/releases).

On first launch, Kube Solo will prompt for configuration information then automatically download the appropriate CoreOS ISO image and bootstrap a Kubernetes cluster. After installation completes you will have a fully functional local Kubernetes environment. More information about what's done during the install is available in the [README](https://github.com/rimusz/kube-solo-osx/blob/master/README.md).

## Using Kube Solo

From the status bar icon you can:

1. Start, stop or reboot the underlying CoreOS VM
1. SSH to the CoreOS VM
1. Attach to the CoreOS VM's console
1. Launch a pre-configured shell session with the appropriate environment and binaries in place
1. Upgrade your Kubernetes environment when a new version is available
1. View your fleet units with Fleet-UI
1. View the Node, Pods and Replication Controllers via Kubernetes-UI

Here's a screenshot of the menu:

![Kube Solo Screenshot](/images/blog-images/kube-solo-osx.png)

Now you're set up, check out the  [official Kubernetes examples](https://github.com/kubernetes/kubernetes/tree/master/examples) to get you started.

Happy Kuberneting!

P.S. If you are in Barcelona on November 20th, join us at [Distributed Matters Barcelona](https://2015.distributed-matters.org/bcn/training-day/). I will be teaching an introductory class on CoreOS, Docker and the Deis PaaS. I'd love to see you there!
