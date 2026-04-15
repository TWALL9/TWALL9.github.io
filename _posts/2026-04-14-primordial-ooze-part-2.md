---
layout: post
title:  "The Primordial Ooze: Part 2"
date:   2026-04-13 20:18:23 -0400
categories: otomo
tags: [otomo, embedded, rust]
---

# Otomo v1 (2020-2021) - Teleop via bluetooth, protobuf and embedded Rust

Just before the pandemic, I was working on a project at work that involved transmitting/receiving protobuf-encoded messages between an embedded system and an Android app. I had extremely minimal Android development experience, so I used the opportunity to create a simple Android app to drive around the little toy rover that I had purchased a year before. 

The other thing I really wanted to get into at the time was Rust. I'd been bound to legacy firmware development in C for several years at that point, and I desperately wanted something new. Something exciting, that also fit into my embedded niche.

So the first version of "otomo" that actually moved was a teleop system on a toy chassis with some custom additions and a scarily bad Android app.

The core of the robot was the venerable STM32F4-Discovery board, married to an expansion board that I received from a professor years before. That F4-discovery/expansion board setup has been in every successive generation of otomo since.

