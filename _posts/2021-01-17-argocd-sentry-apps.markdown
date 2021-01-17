---
permalink: /blog/posts/tutorials/argocd-sentry-apps.html
layout: post
title:  "Installing Sentry On-Premise with ArgoCD"
date:   2021-01-17
category: "tutorials"
tags: [blog, 2021, tutorial]
---

This post is a placeholder for a more in-depth tutorial about ArgoCD, Sentry, and Helm post-install hooks. I am publishing it early to capture the workaround, and to permit closing of [a bug report I filed][] upstream in the [adfinis-sygroup/helm-charts][] community-provided Helm chart repository (on a [self-imposed deadline][].)

The issue is explained in greater depth at the report linked above, but in summary: there is a subtle difference in the way that ArgoCD manages Helm's "Chart Hooks", specifically a `post-install` hook. As a result, there is a chicken-and-egg problem when installing Sentry On-Premise this way.

[Helm's post-install hook][] (when activated by Helm itself) starts executing "after all resources are loaded into Kubernetes." ArgoCD's Helm support does not use Helm itself, does not generate Helm release configmaps or leave a release footprint in the form of a Helm release; instead it basically runs `helm template` and captures the manifest yaml output, which it then syncs to the cluster at sync time. Argo [maps its own pre/post sync hooks][] to the Helm hooks, which is unfortunately [not a clean mapping][], in this case [PostSync waits][] until all resources are healthy before executing, which will never happen as the `deploy/sentry-relay` artifact cannot become healthy until post-install hooks executed, running (among other things) postgres database initialization and migrations.

Unfortunately this is unlikely to be fixed in ArgoCD, and [maintainers recommend Helm Operator][] when perfect helm installations are needed.

Fortunately this is easy to work around! (Placeholder: Screencast is forthcoming)

In tl;dw: switch your ArgoCD Application UI's filter to show resources with a Health status of `Progressing` or `Degraded` only. Sync the ArgoCD Application resource that was created via:

```
helm repo add adfinis https://charts.adfinis.com
helm upgrade --install sentry-apps adfinis/sentry-apps --values values.yaml
```

When `sentry-relay` is started but not ready, this prevents ArgoCD's PostSync hook from executing. [Three other deployments][] (`sessions-consumer`, `snuba-outcomes-consumer`, and `snuba-replacer`) also don't become healthy, but rather than using Readiness check, they terminate (and will land in CrashLoopBackOff status if this continues long enough.) This all means that `sentry-relay`'s failure prevents PostSync, but the others do not.

So, with the filter engaged, wait until everything is healthy except `sentry-relay` is not ready, and the three others mentioned are starting, but also stuck in Terminating loop. You can trick ArgoCD into proceeding easily at that point by scaling down `sentry-relay` for a brief moment.

```
kubectl -n sentry scale --replicas=0 sentry-relay
# ... wait for Jobs to start firing
kubectl -n sentry scale --replicas=1 sentry-relay
```

If it worked, you can tell PostSync has begun because Jobs start firing (and hopefully eventually completing.) When the `sentry-relay` deploy has been scaled back up and the `post-install` Jobs have all been completed, you should find ArgoCD reporting Sync succeeded and everything is Healthy.

I hope this helps someone! It took me a whole afternoon to figure this out myself. 😅

[a bug report I filed]: https://github.com/adfinis-sygroup/helm-charts/issues/143
[adfinis-sygroup/helm-charts]: https://github.com/adfinis-sygroup/helm-charts
[self-imposed deadline]: http://kb.commits.to/review-sentry-helm-on-argocd-blog-post
[Helm's post-install hook]: https://helm.sh/docs/topics/charts_hooks/#the-available-hooks
[maps its own pre/post sync hooks]: https://argoproj.github.io/argo-cd/user-guide/helm/#helm-hooks
[not a clean mapping]: https://github.com/argoproj/argo-cd/issues/355#issuecomment-482237019
[maintainers recommend Helm Operator]: https://github.com/argoproj/argo-cd/issues/4288#issuecomment-693701155
[Three other deployments]: https://github.com/adfinis-sygroup/helm-charts/issues/143#issuecomment-748361114
