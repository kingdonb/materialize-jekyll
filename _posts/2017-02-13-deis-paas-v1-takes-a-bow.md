---
layout: post
published: false
title: "Deis PaaS v1 Takes a Bow"
description: "Deis Workflow Carries the Torch into the Future of Cloud Computing"
tags:
  - Deis Workflow
  - Announcement
author: matt_boersma
---

The original v1 PaaS based on CoreOS and fleet is at the end of its support lifetime today,
February 13, 2017. [Deis Workflow][] provides the same Heroku-inspired experience on
[Kubernetes][], the future of production-grade container orchestration.

The v1 code remains available at [deis/deis][], but we will stop testing and merging fixes
and updates. All focus now is on Workflow and Kubernetes.

## Many Moons Have Passed

When the new Platform-as-a-Service product was first released as a "[public preview][]," it leaned
on Chef. You could `git push deis master` a buildpack-based app and have it scheduled to run on
multiple Amazon EC2 nodes. That was more than three and a half years ago!

<!--more-->

And there began a relentless evolution of the platform we just called "Deis." For years,
the v1 PaaS had a new release every month, only pumping the brakes a bit last year with a
stable, long-term support ([LTS][]) release. v1 has transformed from a set of custom EC2 AMIs with
a primitive job scheduler based on Chef databags to a robust system built from Docker containers
that runs anywhere [CoreOS][] Container Linux does.

## The Scheduler Wars

But as much value as tools like [`etcd`][etcd] and [`fleet`][fleet] provided, more ambitious
container scheduling solutions loomed on the horizon. The Deis v1 PaaS took each of these new
speedboats out for an aggressive test drive, hoping to find a more stable and scalable solution
with a small footprint.

By the v1.9.0 release, intrepid Deis users could flip the switch on one of the "experimental"
schedulers: Docker Swarm (v1), Mesos with Marathon, and early Kubernetes. Commands such as
`deis scale web=3` would then be handled as API calls to the intended scheduler instead of routed
to `fleet` as usual.

## Kubernetes Becomes Ubiquitous

Feedback from brave testers on our IRC freenode channel showed strong interest in the
Kubernetes-based scheduler. Questions about Kubernetes and bug fixes to the "k8s" backend
started to flow. Eventually we removed the experimental schedulers and began the real effort to
move the user experience of Deis v1 PaaS to Kubernetes. That's [Deis Workflow][].

Since then, Kubernetes has grown like kudzu. In announcing that [fleet will be deprecated][fleet],
CoreOS agreed:

> Kubernetes is the best tool for managing and automating container infrastructure at massive scale.

Kubernetes is everywhere, and wherever it is, Workflow works.

## Steady as She Goes

The v1 PaaS saw 5,793 commits from 163 contributors over sixty-eight releases. It spawned a
number of community add-ons and integration tools, all created by you: the user community. Thanks
to the generosity and dedication of numerous DevOps wizards and cloud hackers, each component got
better, faster, stronger.

v1 is still in use in a a lot of places, and the good news is that it's open source and it's always
yours. (Deis, Inc. just won't maintain it any longer.) If Kubernetes isn't in your
plans, we also recommend [dokku][], a mini-PaaS being actively improved. But really you should move
to Kubernetes with [Deis Workflow][].

## Workflow Clears a Path to Kubernetes

Deis Workflow lets you keep using the same `deis` commands you know and love, backed by the
power of a Kubernetes cluster. Your developers can switch over with minimal disruption. And you
can trust that this developer-friendly experience will continue to improve steadily.

As demonstrated by almost four years of standing behind the v1 PaaS, we are serious about support
and maintenance, even as we add features that the community needs. [Semantic versioning][] is not
a joke here. You can always upgrade to a new 2.x version of Workflow, for example, and be confident
it is backward-compatible.

Deis Workflow has a public [planning process][] that ensures decisions about the future [roadmap][]
are out in the open. Join us at our next [Workflow Community Meeting][] and help steer the ship.

Deis Workflow has seen thousands of commits from hundreds of contributors over twelve releases
(and two patch releases). Community add-ons and integrations have already grown up
to [extend Workflow][]. Our #community slack channel is busy with questions being answered and
Workflow users helping others.

And there's no *"waiting for builder..."* installation step. v1 users will know what I mean.


[CoreOS]: https://coreos.com/
[deis/deis]: https://github.com/deis/deis/
[Deis Workflow]: https://deis.com/workflow/
[dokku]: https://github.com/dokku/dokku
[etcd]: https://coreos.com/etcd
[extend Workflow]: https://teamhephy.info/docs/workflow/managing-workflow/extending-workflow/
[fleet]: https://coreos.com/blog/migrating-from-fleet-to-kubernetes.html
[Kubernetes]: https://kubernetes.io/
[LTS]: https://github.com/deis/deis/releases/tag/v1.13.4
[planning process]: https://teamhephy.info/docs/workflow/roadmap/planning-process/
[public preview]: https://github.com/deis/deis/releases/tag/v0.0.4
[roadmap]: https://teamhephy.info/docs/workflow/roadmap/roadmap/
[Semantic versioning]: http://semver.org/
[Workflow Community Meeting]: https://goo.gl/pPZMhW
