---
layout: post
series: mash
---

I decided recently to refactor [my homepage](http://con.rs), so as to better represent the way a bash shell works. This is because my site, while cute, didn't capture the best part of doing things [the Unix Way](https://homepage.cs.uri.edu/~thenry/resources/unix_art/ch01s06.html) -- composability. 

<!--more-->

## Before

The site used to accept commands using a hidden `input` DOM element, then echoed those to a "command line" `div`, which had a blinking cursor anchored to its right (so as the command grew, the cursor would move). Further, the system printed a command prompt. 

When `enter` was detected, the full line from the "command line" `div` was copied to the "output buffer `div`. Then the system would try and interpret the command, potentially outputting more information.

Commands would write to the output DOM element directly. Thus, it was impossible to compose commands together, and further, commands had a hard dependency on the DOM existing, along with the correct element, meaning changing our display strategy was needlessly complex.

The system didn't really handle asynchronicity well, often it was simpler to have the command return immediately, but possibly populate the output buffer later on when the data arrived. So it wasn't always clear to the user they were waiting for anything.

That said, it was a great proof of concept, and also boasted some cool functionality, like the `wall` command, the `say` command, command history, as well as a fake filesystem (the blog portion no longer works, sadly).

## After

[[Source Code]](https://github.com/conrs/mash/)


![Data Flow Diagram](/assets/images/2020_09_11-new-way.png)

The orange arrow represents user input actions. For the browser, this is either key strokes or a paste event. For mobile, tap events would be used.

The shell parses the user input into commands, which it then executes, redirecting all output to a buffer, which is colored in purple because the shell will also output a prompt, along with the command as it is typed.

This buffer is responsible for hard-wrapping lines that exceed a configured width, as well as managing cursor position.

The frontend is now simply a wrapper over a node package which captures all the shell behavior. For now, it cheats a bit and has access to the buffer, which it uses to populate a `pre`, and it also accesses the cursor position directly rather than moving it around based on ASCII codes, which it uses to calculate an appropraiate `top` and `left` for our blinking cursor, which floats above the output `pre`. 

The workarounds are due to reading character-by-character being too slow when compared against typing or echoing lots of output. While it's a neat effect, it shouldn't be the default - we can always introduce a delay as a stream filter instead. You can see it in action in the gif below:

![Stream slowness](/assets/images/2020_09_11-slow-stream-demo.gif)


## Next Steps

Here's the things I have to get done before I deploy to [con.rs](http://con.rs).
- Composability - still haven't actually supported the pipe operator
- Command history support
- Paste support
- Mobile support (touch events instead of keystrokes)
- Tab completion
- HTML-friendly buffer that line breaks based on _visible_ text
- "Say" command (obviously)
- An actual thought out bio
- Analytics update


Here's the things rattling around my brain it would be nice to fix:
- Split some of the responsibilities of the shell out into their own commands.
	- e.g. command history is a very simple script on its own.
- Clean up my buffer tech debt, make an elegant implementation for multiline
- More silly web things to make a command abstraction over
 	- Any thoughts there are welcome!

## Notes from Development

### Buffers are hard

Before, I could use a text input which let me position my cursor for free (it would scale with the size of the inputted text). I could also store the rest of the console output in the div, and treat that as a buffer more or less. 

Now, the cursor position is managed by a buffer command, which keeps track internally of its X and Y. The site as it stands now subclasses the buffer command, and uses that X and Y to move around a blinking div, which represents the cursor. It populates the site body based on the contents of the buffer.

But how do you signal moves using only streams? The real answer is using ANSI escape codes, but I didn't want to go that far, so for now I just used special ASCII codes. The buffer command is smart enough to know when a "arrow key" press by the user should be allowed - if it is, it will emit it to `STDOUT`, otherwise it will simply eat the keystroke. 

However, all this aside, what happens if you want to insert text in the middle of a sentence? Well, the buffer is also smart enough to "delete" characters back to your cursor position, then write your new entered character, then write all the other characters over again. This happens fast enough that the eye doesn't notice anything at all.

For example, imagine the buffer currently contains:
```
Hello world, this is a buffer
```

You decide you want to edit this to be 
```
Hello world, this is a great buffer
```

So you move the cursor left, by pressing the left arrow key 6 times. Then you type `great `. Done! 

After the first keystroke, `g`, the buffer actually outputs something like 
```
[backspace]
[backspace]
[backspace]
[backspace]
[backspace]
[backspace]
g
b
u
f
f
e
r
```

After `r`, 
```
[backspace]
[backspace]
[backspace]
[backspace]
[backspace]
[backspace]
r
b
u
f
f
e
r
```

and so on. for each keystroke until you are done. 

This seems non-intuitive, which leads us to: 


### Ascii codes capture typewriter operations 

A lot of the "cursor position" control codes you'll see in [ASCII codes](https://www.ascii-code.com/) are this way because of typewriters or mechanical printers. 

For example, symbols `CR` and `LF` are inspired by "Carriage Return" (move the cursor to the start of the current line) and "Line Feed" (move the cursor to the next line). These are the two operations a typewriter would do in order to start writing at the beginning of the next line. 
- Some may remember "line endings" differences between DOS and Unix - this was because DOS represented "Newline" as (`\r\n`) just like the typewriter, whereas Unix would simply represent it as LF. (`\n`). 





