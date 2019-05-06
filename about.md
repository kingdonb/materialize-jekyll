---
layout: page
title: About Team Hephy
permalink: /about/
---

#### Hephy Workflow is the open source fork of Deis Workflow

[Hephy Workflow](https://web.teamhephy.com)

#### Introduction

Hephy Workflow v2.21.0 (codename: "Helm Dep Fixer-Upper") is out!

### **Postgres Database Upgrade** Caution/Advice

The Database component supports one-way upgrades to Postgres v11, since v2.20.0.

A general advice for Hephy Workflow users at any version: platform admins should
remember to thoroughly understand backups on a staging cluster, and confirm on
your own details about how to test and verify them.

Test your platform database backups before performing an upgrade. Test upgrades
in your staging environment first, especially if you have important workloads,
and as always recommended according to general best-practices advice, remember
to test any important backups regularly (before you need them) in any case.

#### Current Patch Level - v2.21.0 (latest)

The [v2.21.0][] release includes fresh buildpacks from upstream, some important
security updates for nginx, as well as some new configuration headers in the
router component, e2e testing, and other changes as documented in the changelog
which you can find packaged in the chart.

The [2.21.0 changelog][] is baked into the chart tarball, and will be posted on
the docs site soon, with our next round of documentation updates as per usual.

This release is tested and known to work with K8s clusters on <b>Amazon Web Services</b>, <b>Azure Kubernetes Service</b>, we also support users on <b>Google Kubernetes Engine</b> and <b>Kubernetes on DigitalOcean Engine</b>.  Every release is always tested on <a href="https://github.com/kubernetes/minikube">Minikube</a>, too.

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

[2.21.0 changelog]: https://docs.teamhephy.com/changelogs/v2.21.0/
[v2.21.0]: https://github.com/teamhephy/workflow/releases/tag/v2.21.0
[v2.20.2]: https://github.com/teamhephy/workflow/releases/tag/v2.20.2
[v2.20.1]: https://github.com/teamhephy/workflow/releases/tag/v2.20.1
[v2.20.0]: https://github.com/teamhephy/workflow/releases/tag/v2.20.0
[v2.19.4]: https://github.com/teamhephy/workflow/releases/tag/v2.19.4
[This is a bug]: /blog/posts/announcements/release-v2-20-1-postmortem#description-of-the-bug
[take special steps]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug#the-fallout-from-automatic-upgrading-of-platform-database
[read this blog post for details]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug
