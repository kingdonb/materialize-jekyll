---
layout: page
title: About Team Hephy
permalink: /about/
---

#### Hephy Workflow is the open source fork of Deis Workflow

[Hephy Workflow](https://web.teamhephy.com)

#### Introduction

Hephy Workflow v2.20.1 (codename: "Semver and the Point Release") is out!

**This release comes with several *warnings!* NB** read more below

Users all should be aware that there have been some issues reported since this
was released on Thursday, February 14, and some known bugs may now affect users
of the current and previous release. Users please read all of this information
carefully, especially if you have a Hephy Workflow installation in production
and want to keep it safe.

### **GKE Warning** #1

**GKE users take special note of this upgrade**, as we have accidentally broken
some compatibility with the latest GKE releases, due to their lack of adherance
with SemVer standards. This break was an oversight on our part; we have not
made any emergency patch, as the issue is quite complex it will take time to
communicate with upstreams, and ascertain precisely how to correctly resolve
this safely and without causing any harm.

If you have already upgraded and are affected by this, you can [take special steps][] and [read this blog post for details][],
on how to safely roll-back the Controller to v2.20.0. This will hopefully
mitigate the issue which may be preventing the Hephy controller from becoming
healthy after the upgrade, or from normally answering Deis API requests when
Workflow is deployed on GKE.

### **Postgres Database Upgrade** Caution/Advice #2

There will be a deep-dive blog episode, coming soon, to explain exactly how
the beautiful postgres database backup works, *tl;dr:* **it is now _ever-so-slightly_
possible to accidentally (and potentially irreversibly) *wipe out your database*
during an upgrade.**

For users that installed v2.20.0 prior to February 14, 2019 - [This is a bug][]
that would only affect a narrow set of users and use cases. (Don't worry, it's
really not as bad as it sounds. *But gee you're right, it sounds awful bad at first!*)

Another general advice for Hephy Workflow v2.20.0 users: platform admins should
remember to thoroughly understand backups and retention schedules, and always
test your platform database backups before performing an upgrade. Test upgrades
in your staging environment first, if you have one, and as always recommended
according to best-practices advice, regularly test your backups in any case.

### Helm Repo Update **Warning** #3

Hephy users should run `helm repo update` first, to begin to safely mitigate
the most dangerous of the currently known issues.

The buggy chart referenced in v2.20.0 ([This is a bug][]) post linked above,
has been de-fanged and `https://charts.teamhephy.com`, our canonical charts
repo, is no longer actively dangerous. In this case we have actually
overwritten a slightly bad chart so that nobody will ever install it again.

For details about that issue, please read the [Release v2.20.1 Postmortem](/blog/posts/announcements/release-v2-20-1-postmortem).

**This upgrade is not reversible, so you must take care.** It would be wise,
and is stronly recommended, to take a snapshot of any `database` S3 buckets
first before upgrading, if feasible within your cluster's particular setup.

#### Current Patch Level - v2.20.1 (latest)

The [v2.20.1][] patch represents a major upgrade for [database](https://github.com/teamhephy/postgres)
users, and many important security fixes for all Hephy Workflow users. Users
should upgrade as soon as possible. GKE cluster users will have to [take special steps][]
to receive the upgrades safely, until v2.20.2 is made available.

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

[v2.20.1]: https://github.com/teamhephy/workflow/releases/tag/v2.20.1
[v2.20.0]: https://github.com/teamhephy/workflow/releases/tag/v2.20.0
[v2.19.4]: https://github.com/teamhephy/workflow/releases/tag/v2.19.4
[This is a bug]: /blog/posts/announcements/release-v2-20-1-postmortem#description-of-the-bug
[take special steps]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug#the-fallout-from-automatic-upgrading-of-platform-database
[read this blog post for details]: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug
