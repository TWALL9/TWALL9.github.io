---
layout: post
title:  "The Primordial Ooze: Part 1"
date:   2026-04-13 20:18:23 -0400
categories: otomo
tags: [otomo, embedded, rust]
---

# In the beginning...

There was me working on stuff, abandoning it, then restarting it in different ways. 

I wanted to make a robot. What that robot did changed pretty much every time I asked myself what I wanted it to do.

- I wanted an indoor mobile robot to fetch things around the house
- I wanted an outdoor mobile robot to mow the lawn
- I wanted a platform to learn on, that could evolve as my needs did

Sometime around 2019, I figured it would be fun to create an automatic lawnmower. I didn't have a yard to mow, but it seemed like a neat idea to create some kind of robot mower, driven by GPS or teleop.

At this point in my career (2018-2019), I really only had familiarity with C/C++ on embedded systems, and I really wanted to teach myself higher-level concepts such as SLAM and navigation. However, I was under the naive impression that should be done entirely on an embedded system. So I ended up ordering a simple 4WD toy platform for an Arduino, made the wheels spin, then got frustrated as I tried to create the embedded HAL of my dreams and forgot the forest for the trees.

[Just take a look at all the random commits just for me trying to figure out how to make a HAL in C++](https://github.com/TWALL9/otomo/commits/master/?after=d88cb727fee3f5864d59d1c71ea2f33024082b99+69)

[Or got bogged down around how to make Makefiles from first principles](https://github.com/TWALL9/otomo/commit/d88cb727fee3f5864d59d1c71ea2f33024082b99)

**Automatic Mower -> Auto-Mow -> Otomo**

It was a name that sounded cool at the time, and for sure has nothing to do with old [samurai clans](https://en.wikipedia.org/wiki/%C5%8Ctomo_clan) or [androids from RoboCop 3](https://robocop.fandom.com/wiki/Otomo). Once you see the robot you will very munderstand why I think these comparisons are hilarious.

The project started and stopped several times over the years, often going untouched for a year or more. But it was always there in my office, mocking me.

# It's a Journey

So really this blog is practically a self-admission that I can't finish the damn thing.

Which became the main feature.

I realized after the first iteration of otomo (which will be a later post) that what I really wanted was a platform to build upon, something that would be a foundation for which to add and remove stuff as I pleased with what I was realizing was my access to Grown-Up Money. The actual _application_ didn't matter so much as creating the tools with which to make any application work.

That and I decided that I would accept parts of the project as "just bad proof of concept stuff that has lived for too long" and _not_ go back and modify each and every detail. Doing that would keep me in an iterative loop and never progressing forward. So I accepted the following:

- I would not have the tools to make it pretty
- I would have to accept lots of warts in the implementation, and understand that it's not actual production code.
- Just get over the fact that the wiring will look like absolute ass.

I didn't follow these points very well, but over time, I'm quite pleased with having a little something to tinker on when I feel like it, and accepting that it's not something I'm beholden to has really freed me up from thinking that I _have_ to be working on it.

# So what are we doing here

This is all a long-winded way to say that I created a robot (or at least several versions of a robot) over the course of years and I'm finally getting around to documenting it. I won't have pictures for everything, unfortunately, but I will at least document the respective generations of the thing, what defined them, and what made me move on to the next thing. I'm also going to use this space to detail specifics of the latest generation of robot, and what I'm going to consider a "completed" robot, before actually building any new versions.
