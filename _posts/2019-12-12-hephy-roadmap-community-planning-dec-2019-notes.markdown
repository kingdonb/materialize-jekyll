---
permalink: /blog/posts/announcements/open-roadmap-community-planning-dec-2019-notes.html
layout: post
title:  "Meeting Notes: Open Roadmap Community Planning"
date:   2019-12-12
category: "announcements"
tags: [roadmap, 2019]
---

The December Hephy Roadmap, or "Open Roadmap/Community Planning" meeting for December 2019 was today. What follows are some notes about what was discussed this time. As usual, the Zoom link and Google Calendar invite are contained within this post, although our timing has become more fluid, and this month's Roadmap was not held at the usual time and place. At the end we discuss timing of our next Roadmap planning.

We were glad to welcome some new contributors this month, some of whom joined us for the Roadmap and others contributed via PR on GitHub! Some of our contributors are from different time zones around the world, and although our schedules don't always line up, if you are out there and interested to contribute, we welcome you too. Our top priorities for next release were identified as CI and Documentation updates.

Many of our PRs and issue reports from contributors have taken the form of pointing out issues with our documentation, [the docs][] and [the docs mirror][] which are mostly unmodified original work of Deis Labs and sometimes even still have their branding. We are not shy about having forked from Deis' original work and it is our proud history to follow in the footsteps. But in some ways we have not lived up to the high bar that was set by their team, as evidenced for example by some broken links which still remain within the docs sites. The original documentation is of mostly very high quality copy, but today there remain some broken links which were pointing toward the now defunct Deis Blog. We aim to have all of that fixed by this time next month.

There is more good news though, many of the individual contributors who published there have already granted permission to republish their work, and we continue to look for opportunities to meet up with those "Deis Alumni" and see where their stories have taken them. Many thanks to Rimusz and Bacongobbler who I had the pleasure of meeting at Helm Summit, both regular contributors to the old Deis blog. In the near future you will be able to find most or all of their articles republished here, as they were originally published, now on Hephy Blog!

Our [contributor guide][] was a topic of discussion. As noted above, many references to the original Deis team still remain here. While we have kept most things the same, repeating references to the Deis team do not inspire confidence that Team Hephy follows all of this strictly. This is an example of what we are prioritizing for next release. Many of the [pull request][] guidelines have a foundation in the idea that there is a CI system monitoring Pull Requests and providing leased clusters to a pull request builder when something needs to be packaged for inclusion in the next release.

The Deis team had a fantastic Continuous Integration deployment configuration built on Jenkins, which was included as part of their Open Source gift to our community. This included continuous delivery of the Deis blog and website, as well as robots to vet each site and scan for bad link references before they would be published. None of this works has been permanently lost, but much has been temporarily shed from our Team Hephy as excess weight, for a team which is smaller and at first much less familiar with the codebase than its original creators.

Another topic of discussion was Sponsorship and Infrastructure. Team Hephy has received support from multiple cloud providers, and we have distributed our test harnesses and permanent infrastructure across clusters on those various cloud providers – namely Amazon Web Services, Azure Kubernetes Service, and Digital Ocean who have all granted some cloud infrastructure or infrastructure credits to Team Hephy. A future roadmap meeting may include a presentation including a visual guide, or graphical map, of our very supportive test and production infrastructure providers, and of the teamhephy.info internal test and production environments.

There was also discussion about the next releases that are upcoming and coming soon. Check on GitHub for more details, or join us on Slack anytime!

Meeting time is regularly set for 2:00PM EST on the first Thursday of every month.

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
