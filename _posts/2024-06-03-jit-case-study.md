---
layout: post
title: "Case Study - Shipping Just Enough, Just In Time"
excerpt_separator: <!--more-->
image: 2024_06_03-og-image.png
---
I [recently described](./2024-05-08-shipping-software-jit) how the Theory of Constraints and Toyota Production System can minimize wasted effort and improve the quality and flow of delivered software. However, I got advice from my network that having a concrete example would really help get the point across. Fortunately, I came across an example from work nearly immediately.
<!--more-->

## Context

We have a backlog of work that is sorted by dollar value impact to the business. We pull in the things that have the best value over effort based on our capacity.  Here's a visualization of that backlog:

[Image]

As you can see, we should work on improving the way we handle orders. It's decent effort, but has a high value associated with it. However, while refining this task, we realize we can simply reject the request if the order is beyond a certain status. This probably won't cover all the cases, so we split the tasks out and and recalculated their value over effort.

[Image]

Ok, great! We should do the quick fix right away, and then sequence the more sophisticated work right after it.

https://excalidraw.com/#json=eRcydjRE46Hp0DzqTMcif,aXbcV703PBFOxY5-wY5dTQ