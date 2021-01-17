---
layout: page
title: About Team Hephy
permalink: /about/
---

#### Hephy Workflow is the open source fork of Deis Workflow

[Hephy Workflow](https://web.teamhephy.com)

#### Introduction

Hephy Workflow v2.22.1 is out! The v2.22 series brings support current with Kubernetes v1.16+
which deprecated APIs and GKE upgrades and forced us to finally move on from
the venerable `extensions/v1beta1` APIs for deployments and daemons, in order
to stay in support. We have worked out some kinks and our first patch release
in the series, v2.22.1, is out now too, notably resolving some chart surprises
on AWS and storage issues, as well as addressing problems relating to Helm 3.

#### Current Patch Level - v2.22.1 (latest)

Go ahead and install the [v2.22.1][] release to get the latest Hephy, there are
lots of package upgrades which have been abbreviated in the GitHub changelog.

The [full 2.22.1 changelog][] is [on the docs site][], as usual. Find out more
details there, and remember to read the blog for more awesome Team Hephy news!

This release is tested and known to work with K8s clusters on <b>Amazon Web Services</b>, <b>Azure Kubernetes Service</b>, we also support users on <b>Google Kubernetes Engine</b> and <b>Kubernetes on DigitalOcean Engine</b>.  Every release is tested on <a href="https://github.com/kubernetes/minikube">Minikube</a>, too.

#### License

Copyright&nbsp;&copy;&nbsp;{{ site.author }}

- - -

Team Hephy accepts <b>Issues</b> and <b>Pull Requests</b> from any interested contributors!  If there are any <b>questions</b> or <b>problems</b>, please reach us directly on <b><a href="https://slack.teamhephy.info">slack</a></b>
or just open an <b>issue</b> on the <a href="https://github.com/teamhephy/workflow">github repository.</a>

Under [CC Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/), you are free to <b>share</b> and <b>adapt</b> this document for any purpose.

- - -

#### Personal information

Email: <a href="mailto:{{ site.email }}">{{ site.email }}</a>

##### Team:

Kingdon: <a href="mailto:{{ site.maint1_mail }}">{{ site.maint1_mail }}</a><br/>
Anton: <a href="mailto:{{ site.maint2_mail }}">{{ site.maint2_mail }}</a>

[on the docs site]: https://docs.teamhephy.com/changelogs/v2.22.1/
[full 2.22.1 changelog]: https://github.com/teamhephy/workflow/blob/master/src/changelogs/v2.22.1.md
[2.22.0 changelog]: https://docs.teamhephy.com/changelogs/v2.22.0/
[v2.22.1]: https://github.com/teamhephy/workflow/releases/tag/v2.22.1
[v2.22.0]: https://github.com/teamhephy/workflow/releases/tag/v2.22.0
[v2.21.6]: https://github.com/teamhephy/workflow/releases/tag/v2.21.6
[v2.21.5]: https://github.com/teamhephy/workflow/releases/tag/v2.21.5
[v2.21.4]: https://github.com/teamhephy/workflow/releases/tag/v2.21.4
[v2.21.3]: https://github.com/teamhephy/workflow/releases/tag/v2.21.3
[v2.21.0]: https://github.com/teamhephy/workflow/releases/tag/v2.21.0
[v2.20.2]: https://github.com/teamhephy/workflow/releases/tag/v2.20.2
[v2.20.1]: https://github.com/teamhephy/workflow/releases/tag/v2.20.1
[v2.20.0]: https://github.com/teamhephy/workflow/releases/tag/v2.20.0
[v2.19.4]: https://github.com/teamhephy/workflow/releases/tag/v2.19.4
[This is a bug]: /blog/posts/announcements/release-v2-20-1-postmortem#description-of-the-bug
[take special steps]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug#the-fallout-from-automatic-upgrading-of-platform-database
[read this blog post for details]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug
