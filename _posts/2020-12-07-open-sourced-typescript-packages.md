---
layout: post
excerpt_separator: <!--more-->
---

At LiveRamp, I built a lot of TypeScript services. Coming from Scala, I was very hesitant at first to move to a less strict environment, and I was also wary about structural typing. However, after having used it as well as onboarded engineers to it, I can't recommend it enough.

To make development a little less finicky, I created two packages, which successfully were [open sourced last week](https://liveramp.com/engineering/enhancing-typescript-at-liveramp/)!

<!--more-->

Sadly, LiveRamp's blog doesn't seem to be mobile friendly, so I'll include a summary here, as well as some notebooks.

## Typed Mutations
[GitHub](https://github.com/liveramp/typed-mutations)

A surprising amount of engineering is really just translating between data structures. However, the translation often requires frequent type casts and repetitive code blocks. To reduce the risk of human error, and increase the fluency, I decided to build an abstraction, allowing for “map” and “flatMap” style interactions that play nicely with the type system.

The resulting package lends itself well to in-place mutation without intermediate variables. This empowers you to express business logic fluidly by abstracting away the boilerplate. Here’s a [RunKit notebook](https://runkit.com/conrs/typed-mutations) showing how we transform an object into a differently shaped object!

## Higher Order Promise
[GitHub](https://github.com/liveramp/higher-order-promise)

Another very common task is aggregating multiple asynchronous results. For example, calling multiple APIs in order to construct a unified response. TypeScript and JavaScript have already drastically improved this area, especially with the introduction of async/await. However, one finds themselves trading off readability for performance as the syntax for parallelizing calls is rather delicate.

I wrote Higher Order Promise to make dealing with multiple asynchronous requests a little easier to read, and a lot more robust. It exposes a simple abstraction allowing you to name your asynchronous tasks using simple Javascript objects, as well as letting you chain sequential calls using `then()` – and even better, the type information is kept up to date.

Here's a [RunKit notebook](https://runkit.com/conrs/higher-order-promise) with a few examples.

## Get Involved

Please feel free to submit pull requests or file issues! Starring the repositories would also be nice.

