---
permalink: /blog/posts/tutorials/running-workflow-without-any-loadbalancer.html
layout: post
title:  "Running Workflow Without Any LoadBalancer"
date:   2019-05-19
category: "tutorials"
tags: [blog, 2019, tutorial]
---

When I learned that my favorite Nginx-Ingress [provides an option][] to do TCP load balancing, I decided to try and configure it with Hephy Workflow v2.21.0's `experimental_native_ingress` support, in order to obviate Builder's requirement of a separate L4 LoadBalancer under this configuration. As a Hephy Workflow user and long-time Nginx user this seemed like a natural extension that Native Ingress Support, when configured with nginx-ingress, should be capable of routing SSH traffic to Builder as part of the package.

This post is meant to show briefly, just how it can be done today.

### Ingress Controllers Serving SSH Traffic

This is full-circle really, since Deis Workflow already provided router, which
also is nginx, that routed traffic to Builder as well and had this capability
built-in long before Ingress was ever a thing. Deis Router already provides a
configuration like this by default, and it seems a natural facet of Ingress
would be to facilitate a similar support, except Kubernetes Ingress spec
`networking.k8s.io/v1beta1` still doesn't include any provisions for TCP load
balancing, or multiplexing any other connection type than L7/HTTP.

This is a hack — it's not the formal enhanced support for Ingress that you've
been waiting for. But depending on which Kubernetes provider you are using, you
probably pay something for a LoadBalancer, and if you're like me, you'd rather
not pay for two if you don't need them!

Why not use the nodes?  A HostPort DaemonSet-based mesh of Ingress controllers
could open up their nodes' :2222 SSH, the port for Workflow's Builder, on the
same IP address that hosts deis.controller.your.dns.whatever, spare me from an
extra LoadBalancer and DNS record please!

Good news, if you use `nginx-ingress`, it's supported.

Here's how: and the bad news is, if you've used any decent kind of automation
to deploy Workflow, this will almost certainly break your CI for now. (Sorry,
maybe I'll try and make this better in a later release! For now, it's a hack.)

```shell
helm repo add hephy https://charts.teamhephy.com
helm repo update
helm fetch --untar hephy/workflow

vi workflow/charts/builder/templates/builder-service.yaml
```

### First: Reconfigure Builder to use NodePort

First you're going to change this chart in-place, according to this diff:
```diff
{% raw %}diff --git a/charts/builder/templates/builder-service.yaml b/charts/builder/templates/builder-service.yaml
--- a/charts/builder/templates/builder-service.yaml
+++ b/charts/builder/templates/builder-service.yaml
@@ -12,5 +12,5 @@ spec:
   selector:
     app: deis-builder
 {{ if .Values.global.experimental_native_ingress }}
-  type: "LoadBalancer"
+  type: "NodePort"
 {{ end }}{% endraw %}
```

That was the part that breaks CI. Not going to go into that right now, I'll add
it in a footnote[[1]](#footnotes). I assume you have this chart untarred, and you
edited the template files on disk. If you've done so and still trust me, run:

```
kubectl --namespace=deis delete deis-builder
helm upgrade --install hephy --namespace deis workflow/
```

Now I'm assuming you've already installed Workflow and configured it with
Experimental Native Ingress, which provisioned a `service/deis-builder` of type
LoadBalancer. You delete the service because a `Service` with type LoadBalancer
is immutable and can't be changed into NodePort or ClusterIP. (It needs to be
deleted and created again, the next Helm release will create it new for you.)

This tutorial does not cover setting up nginx-ingress or cert-manager, if you
need help with that you may find it on the [wiki page][], on which I did some
research preparing for this post. Caveat emptor, learning you up on Ingress is
unfortunately something that's out of scope for today's blog post. If any of
this is unfamiliar, you might stop what you're doing and [read the docs][] on
Native Ingress. (You could also, [try web root][] or [reach us on slack][]!)

Next, I am going to show you how to configure the nginx-ingress chart to
forward port `:2222` toward your `deis/deis-builder:2222`:

### Second: Configure Nginx to forward :2222

```diff
diff --git a/nginx-ingress/values.yaml b/nginx-ingress/values.yaml
--- a/nginx-ingress/values.yaml
+++ b/nginx-ingress/values.yaml
@@ -380,8 +380,8 @@ imagePullSecrets: []
 # TCP service key:value pairs
 # Ref: https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples/tcp
 ##
-tcp: {}
-#  8080: "default/example-tcp-svc:9000"
+tcp:
+  2222: "deis/deis-builder:2222"

 # UDP service key:value pairs
 # Ref: https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples/udp
```

If you haven't untarred the chart and are just installing from the live helm
repo, you could say:

`--set tcp.2222="deis/deis-builder:2222"` as in,

```shell
helm upgrade --install ingress stable/nginx-ingress \
  --namespace nginx-ingress \
  --set tcp.2222=deis/deis-builder:2222,controller.hostNetwork=true,controller.daemonset.useHostPort=true,controller.kind=DaemonSet,controller.service.type=NodePort
```

Now you have to be sure that Nginx is reachable on port 2222, however that may
come to have been protected on `$cloudProvider` of your choice: by Security
Group, Firewall, or something else. If you copied settings above, nothing else
surprising has happened, and you created a DNS record pointed correctly at some
of your nodes by the name of `deis-builder.your.platform_domain.example.com`
then you should be able to `git push` to the Deis Builder component now!

### Synopsis

The SSH traffic flows through the nginx-ingress just as easily as HTTP traffic.

Hope this note saves some other cheapskates like me a couple dollars, --Kingdon

#### Footnotes

[1]* it breaks CI because any automation for a helm chart is likely to run
`helm dep build` before installing the chart and its dependencies, which means
your change to the file `charts/builder/templates/builder-service.yaml` right
there in-place on your filesystem, is going to be overwritten by the copy from
upstream before it gets installed.

The solution is, we have to make this an actually supported configuration of
Builder. In a later release, maybe `values.builder.service_type: NodePort` is
a new option in the workflow chart.

[wiki page]: https://wiki.hephy.pro/books/deishephy-workflow-pad/page/deploying-workflow-with-the-helm-operator
[provides an option]: https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples/tcp
[read the docs]: https://teamhephy.info/docs/installing-workflow/experimental-native-ingress/
[try web root]: https://web.teamhephy.com
[reach us on slack]: https://slack.teamhephy.com
