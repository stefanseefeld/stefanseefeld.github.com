---
author: stefanseefeld
comments: true
date: 2017-07-21 18:40:34+00:00
layout: post
link: http://stefan.seefeld.name/2017/07/21/its-the-people-stupid/
slug: its-the-people-stupid
title: It's the people, stupid !
categories:
- software
---

The [boost community](http://www.boost.org/) has recently been shaken by an announcement of the steering committee to move from its current build infrastructure (based on an [in-house tool](http://www.boost.org/build/)) to [CMake](https://cmake.org/). Discussions about such a change have been going on for years, and the SC, noticing that these discussions didn't go anywhere, decided that it would be in the best interest of everyone to [impose a dicision](https://lists.boost.org/Archives/boost/2017/07/237248.php).
Now the "only" remaining question is how to get there, by asking volunteers to implement the change. This has expectedly resulted in a lot of heat, including the [departure of a key member](https://lists.boost.org/Archives/boost/2017/07/237250.php) of the team developing and maintaining the existing build tool, leaving the whole project in a disarray.

I'm myself maintainer of two Boost libraries, and thus have strong opinions about the best way forward. But what I'd like to note here is a more general pattern I have been observing in many different places. And it's about how people conceive of "software", even within the industry, people who should really know better.

Software is considered to be a thing, something that can be changed (broken, fixed, improved, etc.) by applying a "patch". So it's natural to think of "resources" and "jobs" as entities needed to move from state A to state B.
Others think they have done everything possible to make their software "optimal" (not to say "perfect") to solve a given problem, so there is no possible reason for wanting to change it, as that would by definition make the software "less good".

Now consider software as a reflection of our understanding of a given problem, a manifestation of processes and practices - short, a cultural artefact. There can be many reasons to change. The problem we are trying to solve may have changed, or our understanding of it may have evolved. In either case, it doesn't make sense to think of change as merely a patch to be applied to software. It's first and foremost a procedural change, a change that needs to be agreed on and embraced by the very people who work on and with the software, before it can be embodied into software.

Software is a means, not an end. And software engineering really is applied epistemology.


