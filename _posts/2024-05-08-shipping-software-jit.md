---
layout: post
title: "Shipping Software Just-in-Time"
excerpt_separator: <!--more-->
---
There's always been a certain disdain in software circles for management techniques from other industries. With things like the mythical man month, the Agile industrial complex, and the myriad of failed attempts to measure productivity, most software engineers develop a bias early in their career, then as they gain experience, also gain significant blind spots for techniques and learnings from other industries that would help substantially with the problems they face.

<!--more-->

Today I'd like to talk about two blind spots in particular:
- The Theory of Constraints
- The Toyota Production System

These are not new concepts, but despite their focus on manufacturing, they fit nicely when talking about the flow of software development, and show how to minimize wasted effort and achieve a higher quality product.

## The Theory of Constraints

The idea behind the theory of constraints is that there's always going to be a step in the manufacturing process that is the bottleneck. This means it takes longer than the other steps, and work tends to pile up in front of it as a result.

### The WIP Dip

No matter what you do to the other, faster steps, your throughput will remain the same. This leads to an unintuitive conclusion - that it's **better to have those faster steps sit idle than it is to have them continue to produce work**. This is because Work in Process (WIP) costs money, but isn't sellable.

When you think about work like an investment, this makes more sense. All that WIP is money spent, either in raw materials or in labor, and you can't get a return until the bottleneck is finished with it. This turns the faster steps into "expenditure machines" generating more and more WIP while the completion rate still stays the same thanks to the bottleneck - meaning you're waiting longer and longer before you make any money.


![Visualization of WIP](/assets/images/2024_05_08-wip.png){:style="display:block; margin-left:auto; margin-right:auto"}
Up and to the... left? Uh oh! [[1]](https://excalidraw.com/#json=QCcQVsQqpSOn0ZstXACjE,DA2NHgbCoZy_nfA0_kjSKQ){:target="_blank"}
{: style="font-style: italic; text-align: center;"}

Building software after a certain scale is inherently collaborative, and since work will never be perfectly distributed, one team will always end up becoming a bottleneck. To maximize returns, the bottleneck should be working on the highest value item possible at all times, and the bottleneck's capacity should be made clear to the other teams so they can avoid generating WIP.

Generally, you tackle this by having a **refined backlog**. This is a fancy to-do list with the highest value opportunities on the top, and the lowest value opportunities on the bottom. This ties in nicely with the **Backlog Refinement** agile ceremony, and when done continuously, lets us be as sure as we can that the bottleneck is being optimally utilized.

Further, since the Theory of Constraints tells us that non-bottlenecked teams will have spare capacity, having a refined backlog lets us split up the work - either by moving some work to a different team, or by moving people to the bottlenecked team (keeping in mind that mythical man month!). Remember, *anything* is better than the non-bottlenecked teams continuing to generate WIP!

## Toyota Production System

One of the two pillars of the [Toyota Production System (TPS)](https://global.toyota/en/company/vision-and-philosophy/production-system/) is Just-in-Time production. This means orienting your production system around the order flow and not stocking completed inventory at all, rather focusing on making it at the same rate that it sells through. This gets carried left, with any components (e.g. seats, engine) being replenished at the rate which they are sold. Systems like this minimize waste, ensuring you only expend when you expect to make a profit.

Waste in engineering is any effort expended on something that doesn't end up being used. If you write a generalized, highly configurable system but only have one use-case, chances are you have overbuilt and generated some waste. Tackling the problem of generalization or configuration when it's needed will be roughly the same effort, but critically, be validated by real-world use cases.

Most engineering teams are sensitive to the risk of producing an unmaintainable system, and so this section may already be spiking their stress levels. However, it is worth pointing out that **maintenance is a nice problem to have** - it means your software is getting used. Further, while maintainability is qualitative, maintenance *tasks* are quantitative, which helps build a justification for fixing the underlying issues.

Definition of the minimum viable product aside, the rollout strategy is also crucial. If you build a product too soon, you risk it never selling. Further, this isn't very Agile - you'll often find yourselves reworking the solution when you do sell it, as you are unlikely to have gotten everything perfect during up-front planning.

![Visualization of building the product before selling it](/assets/images/2024_05_08-jit-1.png){:style="display:block; margin-left:auto; margin-right:auto"}
High risk of waste, and substantial reworking cost. [[2]](https://excalidraw.com/#json=GQ7KH1gFP1mI8Lh_SsYV_,r-7Eh1gO9QzdvBRvpWqhRw){:target="_blank"}
{: style="font-style: italic; text-align: center;"}

Aligning your production schedule with your first user reduces the risk of waste and speeds up the development process. The risk of rework is also dropped substantially as there is less reliance on the original plan due to immediate iteration with the user before calling the product complete.


![Visualization of building the product while selling it](/assets/images/2024_05_08-jit-2.png){:style="display:block; margin-left:auto; margin-right:auto"}
Reduced risk of waste, tighter feedback loop
[[2]](https://excalidraw.com/#json=GQ7KH1gFP1mI8Lh_SsYV_,r-7Eh1gO9QzdvBRvpWqhRw){:target="_blank"}
{:style="font-style: italic; text-align: center;"}

# Recap

Focusing on identifying your bottleneck and aligning its pace with your other teams will minimize waste caused by the development process at your company.

Aligning feature delivery with launch minimizes mistakes in planning, and helps tighten the minimum viable product. This reduces waste caused by planning mistakes and rework while leading to a better overall customer experience and perception of quality.

