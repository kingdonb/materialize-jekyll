---
layout: post
published: false
title: "Deis Workflow Final Release"
description: "Deis Workflow's final release will be v2.18 in September."
tags:
  - Announcement
author: matt_boersma
---

Deis Workflow v2.18 will arrive in September 2017 as the last official monthly release. After
that, maintainers will accept critical fixes only for six months.

For more detail on the end-of-life plan for Deis Workflow, please read the timeline below
and watch the video of the [August community meeting][].

# Timeline

- 08/08/2017 Deis Workflow [v2.17][v2.17.0]
- 09/07/2017 Deis Workflow [v2.18][v2.18.0] **final release** before entering maintenance mode
- 03/01/2018 End of Workflow maintenance: critical patches no longer merged

<!--more-->

# History of Workflow

Engine Yard released the first alpha version of Deis Workflow on December 30, 2015. After several
months of community testing and feedback, [v2.0.0][] was unleashed on June 9, 2016.

"Deis" was already a popular open source PaaS, based on CoreOS and `fleet`, and Workflow worked
just the same, but ran on this Kubernetes thing everyone was starting to talk about... Neither
Kubernetes nor Workflow were easy to install at first, but that got better (thanks in part to
[Helm][], the Kubernetes package manager we started at about the same time).

A growing number of users started typing `deis create`. And many of those users helped each other
out with Workflow and Kubernetes questions in the Slack channel we made. This community found the
bugs and contributed the code...lots of awesome code! Together with the dedicated Engine Yard
maintainers, we released a new, backward-compatible Workflow version every month like clockwork.

# The Future

As Workflow became a mature product over seventeen releases (so far), the team began working
harder on [Helm][], on [Service Catalog][] machinery, and on Kubernetes itself.

Microsoft demonstrated how serious it is about Kubernetes by acquiring the Deis team from
Engine Yard in early 2017. (And if you like things like Linux, Go, and containers:
[we are hiring][]!) Microsoft also [joined the CNCF][] to help further adoption of open,
cloud-native technologies.

The former Deis / Engine Yard engineers are now part of the Azure Container Services team. We are
putting containers first by improving Kubernetes in Azure and focusing on new Kubernetes-native
open source tools.

# New Tools

In addition to the Kubernetes projects already mentioned, there was one "skunkworks" project that
we couldn't wait to see the light of day: [Draft][]. Draft distills the core ideas of Workflow
into the simplest possible way to deploy your app code directly into Kubernetes. It's also
[Helm][]-compatible, for when your idea moves into production. Join us at the [Draft][] project
and help build the future of app development on Kubernetes!

The Microsoft Azure team plans to build other open source pieces that fill in the gaps around Draft
and Helm. We want to ensure that each tool adheres to the UNIX philosophy of "do one thing well."

Another fresh idea, [Azure Container Instances][], is a perfect example of the kind of
future-facing, container-oriented technology we're focused on at Microsoft. If your app runs in
a container, you can launch it directly into the cloud in just seconds with
[ACI][Azure Container Instances].

# You Are All Workflowers

Deis Workflow was and is a team and community effort. Nearly everyone who worked at Deis, Inc. or
latter-day Engine Yard worked on Workflow. Most of the user community contributed back in some way,
by reporting bugs, by helping other users, or by contributing fixes and features in code.

You did this. You helped. You made this. We can't thank you enough. Long live the Workflow forks!

Crewmembers Vaughn Dice, Matt Fisher, and Matt Boersma would like to extend a salute.
Well done, all of us.

[v2.0.0]: https://github.com/deis/workflow/releases/tag/v2.0.0
[v2.17.0]: https://teamhephy.info/docs/workflow/changelogs/v2.17.0/
[v2.18.0]: https://teamhephy.info/docs/workflow/changelogs/v2.18.0/
[August community meeting]: https://www.youtube.com/watch?v=hNohQYi8nzg
[Azure Container Instances]: https://azure.microsoft.com/en-us/services/container-instances/
[Draft]: https://github.com/Azure/draft
[Helm]: https://github.com/kubernetes/helm
[joined the CNCF]: https://www.cncf.io/announcement/2017/07/26/microsoft-joins-cloud-native-computing-foundation-platinum-member/
[Service Catalog]: https://github.com/kubernetes-incubator/service-catalog
[we are hiring]: https://careers.microsoft.com/jobdetails.aspx?ss=&pg=0&so=&rw=1&jid=303843&jlang=EN&pp=SS
