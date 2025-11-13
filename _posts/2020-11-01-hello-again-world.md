---
layout: post
series: mash
title: Hello Again, World
---

I finally took the time to put the rewrite of mash [online](https://conrs.github.io/mash).

It's still not ready to replace the main homepage, but I've made pretty decent progress, and also managed to support composition of commands.

<!--more-->
## Progress

Since last, I've pulled the following off the todo list:
- Composability
- Command history support
- Paste support
- An actual thought out bio

Still outstanding is:
- Mobile support (touch events instead of keystrokes)
- Tab completion
- HTML-friendly buffer that line breaks based on _visible_ text
- "Say" command (obviously)
- Analytics update

And I've added:
- "interactive mode" knowledge for commands
- stderr stream
- clean up [Command](https://github.com/conrs/mash/blob/5a955c5dd083969ea5a5ae6c22a9e59d3b16bec2/package/src/commands/baseCommand.ts) class structure
- break up commands into smaller ones, compose them

## Challenges

### Buffers

Multiline buffers ended up taking quite a bit of time to get right. Ultimately I went with a doubly linked list representing characters, and also had to have the buffer be in charge of hard wraps (as otherwise the cursor position becomes inaccurate).

The doubly linked list helped with the "insertion of text" problem, as now it is as simple as adding some links. My first stab had some 2 dimensional arrays and was generally a pain, as you'd have to reindex after each insert.

I think I could still simplify this quite a bit, but decided to ship something usable in the meanwhile.

### HTML and Hard Wrapping

Hard wrapping and HTML are a little bit at odds. Consider the following link:
```
<a href="https://blog.con.rs">Blog</a>
```

When you render the HTML, only 4 characters show up. But let's say our buffer wraps at 10 characters wide - well now everything goes out the window. Your cursor will have adjusted for the hard wrap, but then when you actually render the text, two things might happen:
- The visible portion has a line break in it (e.g. `Bl[linebreak]og`)
- The invisible portion has a line break in it (e.g. `https://co[linebreak]n.rs`)
  - In this case, the cursor position will look incorrect, as it will have adjusted for a newline that is no longer there.

For now, the simplest fix was just to remove links (lol).

## Next up

The plan is to keep iterating and add the remaining outstanding functionality. I will probably not spend too much time supporting HTML yet, as it is not the most pressing problem, so first up will likely be better mobile support, followed by analytics support.

There is also a decent amount of cleanup needed, starting with decomposition of some of the beast commands.

I'd also like to take the time to document things right, as well as capture some of the nuance better, which will likely feed into another blog post here. In the meantime, feel free to follow my progress on [the repository](https://github.com/conrs/mash)!