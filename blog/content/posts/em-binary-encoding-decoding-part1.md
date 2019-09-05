---
title: "JS Extreme Minification: Binary Char Encoding - Part 1"
date: 2019-09-05T22:13:30+01:00
draft: false
tags: ["Extreme Minification", "Development", "Code Calisthenics"]
categories: ["Development"]
---

In this post I discuss in detail how I implemented the binary char encoding in JS and the various optimisation steps I took to minimise the final encoding map.

<!--more-->

In my [previous post]({{< ref "/posts/em-encoding-shapes.md" >}}) I discussed how I decided to implement a binary char map to encode the game character shapes to be drawn later using `canvas.fillRect`.

Taking `space invaders` alien as an example, it is hopefully quite straightforward to see that it lends itself quite nicely to be drawn as a series of black and white squares arranged in a grid pattern.

{{% figure src="/img/alien_flat.png#center" width="50%"%}}

Using monochrome colour pallete, the grid can be then represented using a series of 1's and 0's. 1 being black square and 0 being a white. Using this algorithm, we end up with something like:

```
00100000100 // pink
00010001000 // blue
00111111100 // green
01101110110 // etc.
11111111111
10100000101
00011011000
```

which can then be put into a long string and encoded within the JS file.

```
00100000100000100010000011111110001101110110111111111111010000010100011011000
```

To decode and draw the shape, we will simply need to run a nested loop. The outer loop will draw rows while the inner loop will draw the columns.

In pseudo code the algorithm to draw the shape would be:

{{< highlight js >}}
for (row=0; row <= 6; row++)
  for (col=0; col <= 10; col++)
    if characterAt(row, col) === 1
      drawBlackSquare(x, y)
    else
      drawWhiteSquare(x, y)
{{< /highlight >}}

It can be seen above that the alien encoded in binary chars is about **77 bytes**. We need 2 different alien shapes so that's about **~150 bytes** plus maybe another **50 bytes** for the ship and about **10 bytes** for the ship and the starts. So we are looking at **~220 bytes** in total.

This is already better than the solutions discussed in the [previous post]({{< ref "/posts/em-encoding-shapes.md" >}}) so we are definitely on the right track here.

However, we must optimise it still to achieve the 1,024 byte goal.

### Optimisation 1: Mirroring

One benefit of the `space invader` esque alien shapes is that they are symmetrical along the y-axis. I assume the devs back in the day had similar challenges when it came to storage space and available memory as my js1k project does so they intentionally chose shapes that could be mirrored.

That's primarily because such shapes can be defined using only half the information and then mirrored about an axis to draw the other half.

{{% figure src="/img/alien_half.png#center" width="50%"%}}

But this approach will not work with how we are parsing and rendering our bitmap using the algorithm above. Because the algorithm renders row by row starting from the top whereas our shape is mirrored across the y-axis.

So we will need to encode and render columnwise.

### Optimisation 2: Going Sideways

instead of encoding row by row, this step requires us to encode column by column. So:

{{% figure src="/img/alien_sideways.png#center" width="50%"%}}

```
01110000 // pink
00011000 // blue
01111101 // green
10110110 // etc.
10111100
00111100
```

This can be represented as binary char string:

```
011100000001100001111101101101101011110000111100
```

The sequence is now **48 bytes** down from **77 bytes** that is a reduction of about **37%** so we should expect all shapes to take about **~139 bytes** altogether. :+1:

Now when we parse and draw, we can mirror the shape after column 5.

{{< highlight js >}}
for (col=0; col <= 5; col++)
  for (row=0; row <= 8; row++)
    if characterAt(row, col) === 1
      drawBlackSquare(x, y)
    else
      drawWhiteSquare(x, y)

// keeping x and y form previous step, run in reverse to mirror

for (col=4; col >= 0; col--)
  for (row=0; row <= 8; row++)
    if characterAt(row, col) === 1
      drawBlackSquare(x, y)
    else
      drawWhiteSquare(x, y)
{{< /highlight >}}

The algorithm to parse and render does become a bit more complex but we will tackle it a bit later.

***To be continued ...***

