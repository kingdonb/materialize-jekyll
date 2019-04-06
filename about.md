---
layout: page
title: About Team Hephy
permalink: /about/
---

#### Hephy Workflow is the open source fork of Deis Workflow

[Hephy Workflow](https://web.teamhephy.com)

#### Introduction

Hephy Workflow v2.20.2 (codename: "Fixes, Documentation, Maintenance") is out!

### **Postgres Database Upgrade** Caution/Advice

There will be a deep-dive blog episode, coming soon, to explain exactly how
the beautiful postgres database backup works, and how we kind-of broke it
during v2.20.0, just a little bit, just so nobody noticed over 3 months.

For users that installed v2.20.0 prior to the February 14, 2019 release - **NB:**
[This is a bug][] that would only affect a narrow set of users and use cases.

*tl;dr:* **it has been _ever-so-slightly_ possible to accidentally (and
potentially irreversibly) *wipe out your database* during an upgrade.**

Those who ran `helm repo update` on or after February 14, 2019 are not affected.

(Don't worry, it's really not as bad as I made it out to sound. *But gee Kingdon,
**wipe out your database** does sound kind of serious, I guess you're right, it
sounds awful bad the way you started to explain it at first!*)

Another general advice for Hephy Workflow users at any version: platform admins should
remember to thoroughly understand backups and retention schedules, and always
test your platform database backups before performing an upgrade. Test upgrades
in your staging environment first, if you have one, and as always recommended
according to best-practices advice, regularly test your backups in any case.

### Helm Repo Update **Warning**

Hephy users should run `helm repo update` first, to begin to safely mitigate
the most dangerous of the currently known issues.

For details about that issue, please read the [Release v2.20.1 Postmortem](/blog/posts/announcements/release-v2-20-1-postmortem).

**This upgrade is not reversible, so you must take care.** It would be wise,
and is stronly recommended, to take a snapshot of any `database` S3 buckets
first before upgrading, if feasible within your cluster's particular setup.

The upgrade is not reversible because Postgres database upgrades are not
reversible.  (Of course with the right kind of backup you could do anything)

#### Current Patch Level - v2.20.2 (latest)

The [v2.20.2][] patch resolves the issue that caused GKE and some other cluster
users to have to [take special steps][] to receive the database upgrades
safely, prior to the v2.20.2 being made available.

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

[v2.20.2]: https://github.com/teamhephy/workflow/releases/tag/v2.20.2
[v2.20.1]: https://github.com/teamhephy/workflow/releases/tag/v2.20.1
[v2.20.0]: https://github.com/teamhephy/workflow/releases/tag/v2.20.0
[v2.19.4]: https://github.com/teamhephy/workflow/releases/tag/v2.19.4
[This is a bug]: /blog/posts/announcements/release-v2-20-1-postmortem#description-of-the-bug
[take special steps]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug#the-fallout-from-automatic-upgrading-of-platform-database
[read this blog post for details]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug
