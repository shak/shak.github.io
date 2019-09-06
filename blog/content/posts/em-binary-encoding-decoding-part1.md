---
title: "JS Extreme Minification: Binary Char Encoding - Part 1"
date: 2019-09-05T22:13:30+01:00
draft: false
tags: ["Extreme Minification", "Development", "Code Calisthenics"]
categories: ["Development"]
---

This post documents the process of binary char encoding that was used for the js1k space invaders project.

<!--more-->

In my [previous post]({{< ref "/posts/em-encoding-shapes.md" >}}) I discussed why binary char encoding was chosen to draw character shapes in the game and how it benefits from `canvas.fillRect` which we already have available to us in the `js1k` shim.

Using `space invaders` alien as an example for this post, it can be seen that the alien shape can be drawn as a series of black and white sqaures in a 11x8 grid.

{{% figure src="/img/alien_flat.png#center" width="50%"%}}

When using a monochrome colour pallete, the grid can be then represented using a series of 1's and 0's. 1 being black square and 0 being a white.

```
00100000100 // pink
00010001000 // blue
00111111100 // green
01101110110 // etc.
11111111111
10100000101
00011011000
```

Information from the grid can be serialised into a single binary char string that contains the information about colour and position for each pixel in the character shape.

```
00100000100000100010000011111110001101110110111111111111010000010100011011000
```

To decode and draw the shape, we will simply need to run a nested loop. The outer loop will draw rows while the inner loop will draw the columns while the value of each char in the binary string will either paint a black square or a white square.


{{< highlight js >}}
  const bitChars = '00100000100000100010000011111110001101110110111111111111010000010100011011000';
  const sf = 6; //scaling factor;

  for (let row = 0; row <= 6; row++) {
    for (let col = 0; col <= 10; col++) {
      const bit = bitChars.charAt(row * 11 + col);

      if (bit === '1') {
          context.fillRect(col * sf, row * sf, sf, sf);
      }
    }
  }
{{< /highlight >}}

In the example above, the alien encoded in binary chars is about **77 bytes**. We need 2 different alien shapes for the game so that's about **~150 bytes** plus maybe another **60 bytes** for the ship and add about another **10 bytes** for the bullet and the starts. So we are looking at **~220 bytes** in total.

This is already looking better than the solutions discussed in the [previous post]({{< ref "/posts/em-encoding-shapes.md" >}}) so we are probably on the right track here.

However, we must optimise it still to achieve the **1,024 byte** goal.

### Optimisation 1: Mirroring

One benefit of the `space invader` esque alien shapes is that they are symmetrical along the y-axis. I assume the space invader devs back in the day had similar challenges to my js1k project where they were constained by available memory, processing power and storage space.

Mirroring would allow us to define only half the shape of the alien in the binary char string and we can draw the other half using the same information but in reverse.

{{% figure src="/img/alien_half.png#center" width="50%"%}}

However this approach will not work with how we are parsing and rendering our binary char map because the algorithm above renders row by row starting from the top whereas our shape is mirrored about the y-axis.

To make this work, we will need to encode and parse the shape column wise.

### Optimisation 2: Going Sideways

To help keep the algorithm simple and easy to understand, I have rotated the shape counter-clockwise for this step but essentially it is the same shape.

{{% figure src="/img/alien_sideways.png#center" width="50%"%}}

```
00111100
00111101
01101101 // etc.
10111110 // green
00011000 // blue
00001110 // pink
```

The binary char map above serialises to binary char string as follows:

```
001111000011110101101101101111100001100000001110
```

Minor changes have been made to parse and render the map to make it read and render column first.

{{< highlight js >}}
const bitChars = '001111000011110101101101101111100001100000001110';
const sf = 6; //scaling factor;

for (var col = 0; col <= 10; col++) {
  for (var row = 0; row <= 7; row++) {
    const bit = bitChars.charAt(Math.abs(5-col) * 8 + row);

    if (bit === '1') {
      c.fillRect(col * sf, row * sf, sf, sf);
    }
  }
}
{{< /highlight >}}

One thing to note in the algorithm above is how simple it was to mirror the columns in code. The instruction `5-col` - where 5 is the column about which we mirror - will take the column count into negative space e.g. `5,4,3,2,1,0,-1,-2,-3,-4,-5` but that is corrected easily using `Math.abs`.

The simplicity of mirroring in terms of code is also a big positive for this approach.

The binary sequence is now only **48 bytes** down from **77 bytes** which is a reduction of about **37%**. We should expect all shapes to take about **~139 bytes** altogether. :+1:

At 139 bytes it is probably not too bad already but there's a few more ways we can optimise it further. I will be discussing those in another post in detail.

***To be continued ...***

