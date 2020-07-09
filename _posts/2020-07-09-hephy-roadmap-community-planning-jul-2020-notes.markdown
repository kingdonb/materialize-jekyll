---
permalink: /blog/posts/announcements/open-roadmap-community-planning-jul-2020-notes.html
layout: post
title:  "Meeting Notes: Open Roadmap Community Planning"
date:   2020-07-05
category: "announcements"
tags: [roadmap, 2020]
---

The Hephy Roadmap, or "Open Roadmap/Community Planning" meeting for July 2020 was on July 9 at 2PM Eastern Time. Meeting notes are contained herein, please read more below!

---

Meeting time is regularly set for 2:00PM EST on the first Thursday of every month. In July this time, instead, we met on the 9th at 2:00PM EST. Adding this meeting directly to your calendar will enable you to receive the most current information about when our meeting is planned.

[Mark your calendars](https://calendar.google.com/event?action=TEMPLATE&tmeid=MTA5aG5qM2Zpa2NlcnZ2OXA4aHVvOTlxMzFfMjAxOTAyMDdUMTkwMDAwWiBrcXBkaDAzMzZsbmU5OTJsZDNnNjBuOTZ0Y0Bn&tmsrc=kqpdh0336lne992ld3g60n96tc%40group.calendar.google.com&scp=ALL)!

This month we opened with some discussion about "Technical Debt" and had some conversation about how to visualize and report on projects with outdated dependencies for a team that may own code repositories that are written in one or more different languages. I am a Rubyist and primarily work in Ruby, where there is `bundler-audit` and `bundle outdated` which each provide a part of the kind of data that I'm most interested in reporting on, eg. how many packages can be updated, and how many have updates that are related to security. We talked about spending time upgrading everything in a team's suite of project offerings, eg. `bundle update` to the latest version, and what the frequency of how often one does that process actually means in terms of project health; in general the consensus was that it's usually a bit harder to become current the further behind it has gotten.

Everyone agreed that it would be good to get notified when a dependency has been updated upstream, and that it could also be beneficial to collect metrics about how far behind code projects are tracking against their upstream dependencies when they are upgraded. There are certainly analogs to `bundler` in almost every language which could be monitored for outdated dependencies in a similar way, and for cross-disciplinary teams with code projects in several languages (like Team Hephy or any team really), we could benefit from an implementation of these ideas when seeking to understand more deeply about the project health.

Kingdon gave a demonstration of Hephy's pre-release v1.16 support in its current state, which has a few hiccups remaining to resolve before Hephy Workflow v2.22 can be released.

An inquiry was made about [object-storage-cli/pull/2](https://github.com/teamhephy/object-storage-cli/pull/2) and whether it will make it into the next release; since it has been merged, the answer is yes, it is certainly planned to be included in v2.22. (Thanks @jfuechsl!)

Another contributor has planned to provide an update to the `deis-cli` binary, so that it does not call itself Deis anymore! This PR is expected to land in [workflow-cli/pulls](https://github.com/teamhephy/workflow-cli/pulls) inside of the next 24 hours. (Thanks @ess!)

Another question was asked about [Deis Dash](https://github.com/olalonde/deisdash), the project for visualizing and controlling Workflow workloads interactively in a web browser. This has been mentioned [in the documentation](https://docs.teamhephy.com/managing-workflow/extending-workflow/) for some time, and we appreciate this contribution! However the support from original creator has lapsed, versions of all dependency packages are very old, and it could use a refresh from someone with JavaScript skills and interest. (Call for volunteers!)

Thanks everyone who joined us for the meeting today, it was good to hear from you all and we appreciate the spirit of community that everyone shares when we have these Open Roadmap Community Planning meetings most every month. Please check out our [contributor guide][] if you have questions about how to provide contributions to Team Hephy, and also remember that [the docs][] are still reasonably current, in spite of the appearance or any broken links you may find, (we will gladly [accept PRs](https://github.com/teamhephy/workflow/pulls) to resolve those issues too, if you find anything you can easily fix, please feel free to report them, or send a fix for us in the [teamhephy/workflow][] repository.)

We are evaluating whether it would be helpful to engage more community members in the meetings, if we held the meeting at a different time of day or week, or with a different format. Please interested parties feel free to share your opinion on Slack, or send feedback privately to [team at teamhephy.com](mailto:team@teamhephy.com). We are not married to the 2:00PM Thursday format or schedule for our meetings, and will consider your feedback.

Thanks!

– Kingdon

<a target="_blank" href="https://calendar.google.com/event?action=TEMPLATE&amp;tmeid=MTA5aG5qM2Zpa2NlcnZ2OXA4aHVvOTlxMzFfMjAxOTAyMDdUMTkwMDAwWiBrcXBkaDAzMzZsbmU5OTJsZDNnNjBuOTZ0Y0Bn&amp;tmsrc=kqpdh0336lne992ld3g60n96tc%40group.calendar.google.com&amp;scp=ALL"><img border="0" src="https://www.google.com/calendar/images/ext/gc_button1_en.gif"></a>

P.S. if you are just trying to join the meeting, here is the Zoom link, and it is hopefully stable from month to month: [https://zoom.us/j/566476564](https://zoom.us/j/566476564)

[the docs]: https://docs.teamhephy.com/
[the docs mirror]: https://teamhephy.info/docs/workflow
[contributor guide]: https://teamhephy.info/docs/contributing/overview/
[teamhephy/workflow]: https://github.com/teamhephy/workflow
