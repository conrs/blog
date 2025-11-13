---
layout: post
series: standardization
title: "\"What\" vs. \"Why\""
---

Standardization is one of the hardest things an engineering organization can do, especially when it's also trying to ship new products. 

Small and medium sized companies often face this - they have a lot of market share to win, so they need a lot of features. They need a lot of features, so they hire a lot of engineers. They hire a lot of engineers, so they want to keep them aligned. 

The trouble is, while a lot of engineers are familiar with standards, chances are they mostly enjoyed the "what" without understanding the "why". And, by nature, standardization is a cross-cutting concern, which means its impact is amplified - good or bad.

<!--more-->

Let's take a look at the pitfalls an organization will face if they miss the "why" behind the "what".

## Common Mistake: Starting with a Standards Doc

#### What the company wanted

Consistent technologies to be used.

#### How they tackled the problem

Senior engineers catalog current technologies, debate merits, and produce a list of approved technologies. Any new technologies must be approved by a group of senior engineers.

#### Where they ended up

The list of standard technologies proves inadequate for ever-evolving product needs. 

However, individual contributors (IC's) all still adhere to the list, because the risk and cost of proposing new technologies is not something any one IC can take on. Those who try typically burn out, often over something clearly supported by the industry at large. New products ship with compromises and workarounds, and generally teams trend towards learned helplessness.

#### What'd they miss? 

- They assumed that companies with standards started with a standardization doc.
- They thought the reason code was consistent there was due to enforcement, not enablement.
- They eroded autonomy and trust in their service and product owners.

Standardization when done incorrectly looks a lot like building software using Waterfall principles. This is because the planning is all up-front, and then once the plan is approved, it's very hard to change. In more business-y words, the company failed to validate its standardization product in the market.

This situation is further complicated by the incorrect belief that standards should not change. So a lot of the time, this rigidity is seen as a feature. However, the pitfalls of Waterfall planning are still applicable here, which means you can be sure that your rigid standards have not anticipated all the needs of the engineering organization.

If we start with the code and work backwards, an alternative path to consistency emerges, one that is proven by concepts like Economics, Science, and Evolution. The gist is that useful ideas should proliferate, and that when they do, consistency will follow. Thus emerges the "why" behind the "what" -- **Companies with strong proliferation of ideas will naturally converge towards consistency, and new products they ship will maximize consistency without intervention.**

This leaves autonomy and trust in the hands of the teams building products. They are capable of making tradeoffs given the data they have, which we know will be much more than any central planning authority would have. Choices they make are communicated with adjacent teams, and generally bad tradeoffs will be caught at that step, while good ones gain further adoption. These good ideas continue to percolate upwards, eventually themselves being standard -- only this time, instead of petitioning a board, the idea has gotten there because enough IC's have found it to be useful.

## Conclusion 

Rather than enforcing consistency, we should enable it. What this means is, don't start with the goal, start with the steps that feed into the goal. Hold internal conferences, encourage tech talks - essentially, get your engineers collaborating - and the standards will follow.




