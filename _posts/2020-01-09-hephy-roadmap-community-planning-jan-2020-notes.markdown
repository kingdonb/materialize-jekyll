---
permalink: /blog/posts/announcements/open-roadmap-community-planning-jan-2020-notes.html
layout: post
title:  "Meeting Notes: Open Roadmap Community Planning"
date:   2020-01-09
category: "announcements"
tags: [roadmap, 2020]
---

The January Hephy Roadmap, or "Open Roadmap/Community Planning" meeting for January 2020 was today. As usual, the Zoom link and Google Calendar invite are contained within this post, although our timing has become more fluid, and this month's Roadmap was not held at the usual time and place. At the end we discuss timing of our next Roadmap planning.

As always, we welcome and thank our team members and contributors who send us issue reports and pull requests! A pull request was merged which fixes a weird Python bug in the controller, [PR #114(controller)][], that only triggers when you have to run more than 256 pods off of a single deployment. Take note if you might be doing that. Among other things, this and more will find their way into our next patch release of Hephy Workflow, v2.21.5.

This month we have incorporated several upgrades for the go-dev package including Go v1.12, that docker go support image which is still being maintained by Deis upstream; we are also forking (at least temporarily) to accommodate some customizations that were needed to the built-in configuration, because of tight memory constraints on our CI environment, and a deadline limit in builder which was being reached during source code linting.

We are also upgrading all of the other components, likely we will also be making another point release prior to the the next minor bump, v2.22, in which we intend to provide the long-awaited final support for Kubernetes 1.16+ and stop using deprecated APIs, namely `v1beta1`! The extent of backward compatibility which we will be able to offer is still as of yet uncertain.

It may turn out to be much easier for Team Hephy to hard deprecate support for older Kubernetes releases that are no longer even still supported themselves. (People with strong opinions can certainly still have some time to share them with us on Slack, before our Final Kubernetes v1 support is landed, so there is still time right now! Do come talk to us, on any of Twitter, GitHub, or Slack.)

Either way this will mark a major achievement and milestone in Team Hephy's maintenance and command as well as understanding of the original Deis Workflow codebase. Thanks go to a lot of people who made drive-by or repeat contributions, spending time and effort to talk through and resolve issues.

It is still hopeful this news will be ready for next month's Open Roadmap. There are an awful lot of components to verify and check for dependencies to upgrade. Favored for more attention perhaps in another minor release (say, possibly v2.23), it is possible that we will see contributions upgrade Builder, especially `slugbuilder` and perhaps also `dockerbuilder`, with new alternate backends. Some contributors expressed an interest in replacing the slug builder with something compatible that is maintained by another team.

Although this was not actually discussed at the Roadmap today, but since it was mentioned earlier in December, I will re-share it now that Dokku contributed and maintains a library called [herokuish][], which integration attempt may be contributed by an interested friend of Team Hephy.

Nothing at all is final about this proposal, but it would be great to reduce the overall complexity and number of components that need to be maintained by Team Hephy and take advantage of similar projects. Our volunteers can work on anything they want, we are always open to proposals as described in our [contributor guide][], and we welcome adaptations of best-of-breed solutions that can or could integrate with Workflow.

We are also upgrading the Object CLI which has gone untouched for some time, that seems to be the source of a problem with support for the full modern set of AWS EC2 and storage regions like `eu-west-2`. Users who have been waiting for this support can expect that support is coming soon, likely in another patch release before the next minor release.

I have made plans to update our docs that are still in a minor state of disrepair. Keep an eye on [the docs][] and [the docs mirror][] for still more changes soon, to coincide with the next minor release. We expect to break things first in the `workflow-beta` Helm chart release, and perform our most extensive tests yet for the 2.22.0 workflow chart release, to verify that v2.22 `workflow` chart works as well on all supported Cloud platforms.

This news will help us keep using Workflow on stable Kubernetes for a long time with confidence, and full support for the latest Kubernetes releases.

---

Meeting time is regularly set for 2:00PM EST on the first Thursday of every month. In February 2020, that is February 2 - Groundhog Day! Tune in live next time, for the Groundhog Day release party and Open Roadmap Release Planning meeting on Zoom.

Adding this meeting directly to your calendar will create a recurring appointment for 2:00PM Eastern US Time, on the first Thursday of every month.

[Mark your calendars](https://calendar.google.com/event?action=TEMPLATE&tmeid=MTA5aG5qM2Zpa2NlcnZ2OXA4aHVvOTlxMzFfMjAxOTAyMDdUMTkwMDAwWiBrcXBkaDAzMzZsbmU5OTJsZDNnNjBuOTZ0Y0Bn&tmsrc=kqpdh0336lne992ld3g60n96tc%40group.calendar.google.com&scp=ALL)!

We are evaluating whether it would be helpful to engage more community members in the meetings, if we held the meeting at a different time of day or week! Please interested parties feel free to share your opinion on Slack, or send feedback privately to [team at teamhephy.com](mailto:team@teamhephy.com). We are not married to the 2:00PM Thursday format schedule for our meetings, and will consider your feedback.

Thanks!

– Kingdon

<a target="_blank" href="https://calendar.google.com/event?action=TEMPLATE&amp;tmeid=MTA5aG5qM2Zpa2NlcnZ2OXA4aHVvOTlxMzFfMjAxOTAyMDdUMTkwMDAwWiBrcXBkaDAzMzZsbmU5OTJsZDNnNjBuOTZ0Y0Bn&amp;tmsrc=kqpdh0336lne992ld3g60n96tc%40group.calendar.google.com&amp;scp=ALL"><img border="0" src="https://www.google.com/calendar/images/ext/gc_button1_en.gif"></a>

P.S. if you are just trying to join the meeting, here is the Zoom link: [https://zoom.us/j/566476564](https://zoom.us/j/566476564)

[the docs]: https://docs.teamhephy.com/
[the docs mirror]: https://teamhephy.info/docs/workflow
[contributor guide]: https://teamhephy.info/docs/contributing/overview/
[pull request]: https://teamhephy.info/docs/contributing/submitting-a-pull-request/
[herokuish]: https://github.com/gliderlabs/herokuish

[PR #114(controller)]: https://github.com/teamhephy/controller/pull/114
