---
layout: post
title: "Never build for the future"
---

After a couple years of programming are under your belt, it's likely you'll fall into the trap of building for the future. This feels like a good thing to do, especially when you are dealing with technical debt from folks who _didn't_ build for the future. However, if this resonates with you, let me save you some grief: **It's a waste of time!**

<!--more-->

The reason you're forming this intuition is because of the hindsight bias you have on the technical debt you come across. You know for a fact that your job today would have been easier if whoever wrote this code thought through how it might be used or extended. 

However, consider the perspective of that original author. To them, there's multiple possible futures, and no guarantee any of them will need additional sophistication. In fact, in most of them, the code will get discarded.

By building for the future, you're gambling on your additional effort eventually having a use. Doing this only makes sense when you're sure about what the future holds, **AND** when the payoff from making the bet now is substantially higher than making it later. 

It gets worse - effort is spent on more than just writing the code. For example, writing unit tests for the extra functionality, or having a QA team validate it all before release. With the stakes that high, do you still want to roll the dice?

Remember, if you weren't spending your effort on this, you'd be spending it on something else that has a more qualified, immediate use. Thus, it's almost always the smarter play to only build what you need today - tomorrow can wait.

![GIF from Community episode "Remedial Chaos Theory"](/assets/images/2024_09_25-community-yahtzee.gif){:style="display:block; margin-left:auto; margin-right:auto"}
Don't make this your darkest timeline.
{: style="font-style: italic; text-align: center;"}
