---
layout: post
title: Getting rusty
categories: rust
date: 2020-07-13 11:01 -0500
---
A few weeks ago I started to create my own custom C++ project template for embedded projects. It would be great. I would build it from the ground up and know exactly how it works. Dive into the nitty-gritty details and tune it to my liking. Integrate the latest tools, to monitor binary size and relate changes to specific commits, have nice code coverage statistics, a linter, and various static analyzer tools available at a single command. I would extend it gradually, adding the features as I see fit and mould it to fit my way of working. There was a small example application to show how it works. I determined and documentation relevant compiler and linker flags for GCC, Clang, and MSVC. All, such that I could have a single point from where to compile my embedded application for target or compile unit-tests on my PC. And from there build other, larger, more sophisticated applications within my workflow. Did I mention it would be great?

And then I decided to scrap it.

Why? Because it was not fun anymore. The more time I spend on it, the less I enjoyed working on it, becoming less motivated to work on it. I realized I was mimicking a production environment, which is not what I set out to do. I do these kinds of projects for myself, to learn, to explore, and to challenge myself and to have a bit of fun while I am at it. So, I went back to a simple question: what do I want?

For now, I decided to pick up something I have had on my wanting to learn list for some time: learning Rust. Over the last years, I started reading the Rust book a few times, but never actually made it past the first few chapters. Nor made anything. Why? I think because I missed a goal.

So, I have set myself the challenge to create my first embedded Rust application and properly understand it. So, not just combining a few libraries and be done with it. The application itself would be relatively simple, a data collector for an IMU development board that has been collecting dust on my desk for months.

From there on, I will use that data collector to experiment with some IMU related algorithms and estimation theory. But more on that later, for now, I take it one step at a time.
