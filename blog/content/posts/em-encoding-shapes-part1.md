---
title: "JS Extreme Minification: Encoding Shapes - Part 1"
date: 2019-09-05T20:53:45+01:00
draft: false
tags: ["Extreme Minification", "Development", "Code Calisthenics"]
categories: ["Development"]
---

One of the biggest challenges while doing my js1k challenge was rendering shapes. In part 1 of this series of posts I will discuss in detail the various possibilities and why I chose to go with binary char encoding.

<!--more-->

## The Task

For my js1k project I decided to write a game roughly based on `space invaders` in under 1,024 bytes.

For those not familiar with [js1k](https://js1k.com/), it's a challenge where you are only given an HTML doc with a canvas element in it and a script tag and you are required to write an application in under 1,024 bytes without any external includes (no CSS, no images, no external JS).

Once I had figured out the game clock & key event listeners; it was time to start adding shapes for the game characters.

Primarily, I required:

- At least two different kind of alien characters
- A shape for the battle ship
- A shape for the bullet that the battle ship will shoot
- Shapes for the stars (and perhaps planets too?)

## Searching For The Solution

### :no_entry_sign: Solution 1: SVGs

The original space invader images looked like they would translate to SVG quite nicely. So that was the first potential solution I looked into.

{{% zoom-img src="/img/space_invaders_original.png" %}}

I drew half the alien and then mirror flipped it.

{{< highlight svg >}}
<svg viewBox="0 0 350 257">
  <g id="hsi">
    <path d="M0 127h176v65H0z"/>
    <path d="M64 63h32v161H64z"/>
    <path d="M96 224h64v33H96z"/>
    <path d="M64 0h32v33H64z"/>
    <path d="M31 97h38v33H31z"/>
    <path d="M0 190h32v33H0z"/>
    <path d="M95 32h33v65h-33z"/>
    <path d="M128 64h48v65h-48z"/>
    <path d="M63 63h112v33H63z"/>
  </g>
  <use xlink:href="#hsi" transform="translate(350, 0) scale(-1, 1)"></use>
</svg>
{{< / highlight >}}


At this point I was going to optimise and minify the SVG but ... just *half* of the first alien is **377 bytes** which is about a quarter of my entire byte allowance so this was not going to work.

Also, considering the code I would have required to create and insert SVG element into the DOM would have made this approach impractical.

### :no_entry_sign: Solution 2: Base 64 Encoded Image

OK. So we can base 64 encode images. I can potentially put all images into a single sprite and some how crop it and render it to create the aliens.

{{< highlight js >}}
'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAAEACAYAAAADRnAGAAACGUlEQVR42u3aSQ7CMBAEQIsn8P+/hiviAAK8zFIt5QbELiTHmfEYE3L9mZE9AAAAqAVwBQ8AAAD6THY5CgAAAKbfbPX3AQAAYBEEAADAuZrC6UUyfMEEAIBiAN8OePXnAQAAsLcmmKFPAQAAgHMbm+gbr3Sdo/LtcAAAANR6GywPAgBAM4D2JXAAABoBzBjA7AmlOx8AAEAzAOcDAADovTc4vQim6wUCABAYQG8QAADd4dPd2fRVYQAAANQG0B4HAABAawDnAwAA6AXgfAAAALpA2uMAAABwPgAAgPoAM9Ci/R4AAAD2dmqcEQIAIC/AiQGuAAYAAECcRS/a/cJXkUf2AAAAoBaA3iAAALrD+gIAAADY9baX/nwAAADNADwFAADo9YK0e5FMX/UFACA5QPSNEAAAAHKtCekmDAAAAADvBljtfgAAAGgMMGOrunvCy2uCAAAACFU6BwAAwF6AGQPa/XsAAADYB+B8AAAAtU+ItD4OAwAAAFVhAACaA0T7B44/BQAAANALwGMQAAAAADYO8If2+P31AgAAQN0SWbhFDwCAZlXgaO1xAAAA1FngnA8AACAeQPSNEAAAAM4CnC64AAAA4GzN4N9NSfgKEAAAAACszO26X8/X6BYAAAD0Anid8KcLAAAAAAAAAJBnwNEvAAAA9Jns1ygAAAAAAAAAAAAAAAAAAABAQ4COCENERERERERERBrnAa1sJuUVr3rsAAAAAElFTkSuQmCC'
{{< / highlight >}}

Again, the same problem. The sprite is about **814 bytes** and at this point I wasn't even sure how I was going to crop aliens out of the sprite so I had to abandon this approach as well.

### :no_entry_sign: Solution 3: Unicode Baby!

There are block unicode characters ( ▉ ). I can insert `<p>` tags with block characters in the right order and perhaps create the alien shapes.

I actually pursued this idea for a good few days and had a working solution but I still needed to encode the alien shape information in the code at which point the unicode characters were simply replacing a single call to `canvas.fillRect`. This, I decided was not worth the extra complication and size it added to the application.

### :white_check_mark: Solution 4: "canvas.fillRect"

I already had the `canvas` element provided to me by the `js1k` shim so it made sense to use that to draw the shapes. All I had to do was to encode the shape information in the application somehow.

The idea first seemed like another lost cause but it made enough sense to spend some time on it and see how far I could take it.

It turned out to be one the most enjoyable part of the entire project.

Stick around for part 2 of this series where I will go in depth to explain how I solved the shape encoding/decoding problem using JS and how I optimised the shape information to encode everything in just **13 bytes** of information.



