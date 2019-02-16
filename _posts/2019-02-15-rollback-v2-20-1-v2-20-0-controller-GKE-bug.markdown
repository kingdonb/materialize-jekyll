---
permalink: /blog/posts/announcements/rollback-v2-20-1-v2-20-0-controller-GKE-bug.html
layout: post
title:  "Rollback: v2.20.1 to v2.20.0 solves Controller/GKE bug"
date:   2019-02-15
category: "announcements"
tags: [rollback, 2019]
---

The [issue was confirmed and triaged][] by users on our Slack channel on
February 15, one day after [v2.20.1][] was released in the wild. Read more in
this post for details about how this issue can be mitigated and recovered from.

If you have not upgraded yet, and wish to avoid this issue and wait for
the next release, simply use Helm to install the previous version, `v2.20.0`
– which works, and does not suffer from any (known) GKE compatibility issues.

```bash
helm install hephy/workflow -n hephy --namespace deis -f values.yaml --version v2.20.0
```

#### The GKE Issue

If you are not using GKE, there may be no need for you to roll back your
workflow Controller release from `controller-v2.20.1` to `controller-v2.20.0`.
You can upgrade to `v2.20.1` and everything is fine for you.

If you are using the platform database on-cluster, and have already upgraded,
then rolling back the entire `workflow` release to `v2.20.0` wholesale, **may no
longer be possible.**

Caveat emptor, or if you have already hit this issue, please keep reading to
understand everything before running any further upgrade or rollback commands:

#### The Rollback Issue

```bash
helm upgrade hephy hephy/workflow --version v2.20.0 -f values.yaml
```

Above, is the cleanest and simplest way to revert and resolve this kind of issue at
once, in this case it will work **only if your cluster uses an off-cluster, or
external, postgres database**. We always assume throughout documentation that
you've installed into a release named `hephy`.

**If your cluster uses an on-cluster postgres, then rolling back the workflow
chart to `v2.20.0` is *not* recommended, and will not work.** This is where
things get complicated, and we're sorry to all GKE users with clusters that
may be impacted.

Please join us in [#support][] on [the slack][] to talk to a person. We will
gladly try and help you get sorted, in case this post is unclear or confusing.

**Roll back the `deis-controller` deployment *only* instead,** to mitigate the
GKE issue. Read on to learn how. We will release a new `v2.20.2` workflow as
soon as possible so that GKE users can avoid this minor calamity in the future.

##### Preserve Chart `values.yaml` data

Before you take action, please take care to understand the confluence of
issues at play. Remember to preserve, and always provide values data when
upgrading, in `workflow/values.yaml` when doing `helm upgrade` commands with
Hephy charts. Do this in order to keep the same settings from before. An
excerpt from the [helm upgrade #synopsis][] anchor in the Helm docs:

```
If no chart value arguments are provided on the command line, any existing customized values are carried forward.
```

While this is true in general, some users may be upgrading from previous
versions of Deis Workflow, and so this "carry forward" behavior can be
unreliable. Provide a `values.yaml` for extra safety; otherwise, the defaults
in `values.yaml` may be used (and that probably isn't what you want to happen.)

##### The Amazing New Fully-Auto Platform Database Upgrades

Astonishingly, for clusters that are using an on-cluster postgres database, and
regardless of your Minikube or any Cloud/Kubernetes vendor hosting arrangement,
if you have already upgraded to `database-v2.7.2` and your database is healthy,
then your database and backups are at Postgres v11.1 now.

Amazed? We hope that feature worked well for all of our users, and that it was
so easy you didn't even need to notice that it happened.

The automatic upgrade was first merged and contributed in `database-v2.7.1`,
but it was judged not ready yet, so was held back until now. In
`workflow-v2.20.1` and `database-v2.7.2`,
[automated postgres upgrades](https://github.com/teamhephy/postgres/commit/ae89bf46d77fb3fd798b1361f4709793e5d029e3)
have been implemented, tested, and released. Unfortunately there is some risk
associated with the change, so take care and please test upgrading in a staging
environment before upgrading any important or Production clusters to v2.20.1.

Additional testing has been performed, and Team Hephy believes this feature is
now ready for prime-time, so it has finally been released! 

##### The Fallout from Automatic Upgrading of Platform Database

**Unfortunately these changes mean that database upgrades cannot easily be
un-done (rolled back or reverted) to the previous release anymore (*unless you
have previously taken a snapshot of the database bucket prior to upgrading.*)**

An effort is underway to update the documentation with best-practices advice
around database backups, taking snapshots, and upgrading safely.

So the above helm rollback strategy will not always work, for clusters with the
postgres platform database hosted on-cluster, since `database-v2.7.2` included
this non-reversible upgrade that has converted your database to Postgres 11.1;
without the ability to restore an older database backup, you will simply have
to forge ahead from now.

Cluster administrators will need full Kubernetes API access and permission to
edit the manifests and make `kubectl rollout undo` API calls in order to
resolve the situation after a failed upgrade.

A strategy that you can try instead of helm, rolling back your controller only:

```bash
kubectl --namespace=deis rollout undo deployment deis-controller --to-revision=5
```

To determine the correct revision to use for roll-back, you must use `kubectl`
to describe the deployment of `deis-controller` (here I have `alias kd='kubectl -n deis'`):

```bash
$ kd describe deployment deis-database
Name:               deis-database
Namespace:          deis
CreationTimestamp:  Sat, 18 Aug 2018 11:49:20 -0400
Labels:             heritage=deis
Annotations:        component.deis.io/version: v2.7.2
                    deployment.kubernetes.io/revision: 6
Selector:           app=deis-database
...

$ kd describe deployment deis-controller
Name:                   deis-controller
Namespace:              deis
CreationTimestamp:      Sat, 18 Aug 2018 11:49:20 -0400
Labels:                 heritage=deis
Annotations:            component.deis.io/version: v2.20.1
                        deployment.kubernetes.io/revision: 5
Selector:               app=deis-controller
...
```

On my upgraded cluster, you can see the now-broken v2.20.1 controller is at revision 5, so I would use
`kubectl --namespace=deis rollout undo deployment deis-controller --to-revision=4` to get back
to the previously installed revision.

If you use `helm ls` and `helm history hephy` at this point, you will see that
`rollout undo` does not create any new revision in helm, as you might have
expected. Our apologies for that. Instead, you must use `kubectl --namespace=deis rollout history deployments`
to inspect the history of ReplicaSet revisions for the `deis-controller`
component's Deployment object.

```
$ kd rollout history deployments deis-controller
deployment.extensions/deis-controller
REVISION  CHANGE-CAUSE
3         <none>
6         <none>
7         <none>
```

In my case, I am not using GKE, but I have performed the rollback in order to
document this process for information. The `v2.20.1` controller is fine for me,
so after I rolled-back from Revision 5 to 4, I rolled forward again to 5. The
revision `4` becomes `6`. Above, I have already rolled-forward again:

```
kubectl --namespace=deis rollout undo deployment deis-controller --to-revision=5
```

Revision 4 and 5 no longer appears in rollout history, with the normal behavior
of `rollout undo` in the Kube API, they are renumbered to appear as revision 6
and 7. Your revision numbers will likely not match mine, so take care.

[issue was confirmed and triaged]: https://teamhephy.slack.com/archives/C6Z7XLEAW/p1550258853140000
[v2.20.1]: /blog/posts/announcements/release-v2-20-1-postmortem
[#support]: https://teamhephy.slack.com/messages/CG908TB52/
[the slack]: https://slack.teamhephy.com/
[helm upgrade #synopsis]: https://github.com/helm/helm/blob/master/docs/helm/helm_upgrade.md#synopsis
