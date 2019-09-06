---
title: "JS Extreme Minification: Binary Char Encoding - Part 2"
date: 2019-09-06T12:20:14+01:00
draft: false
tags: ["Extreme Minification", "Development", "Code Calisthenics"]
categories: ["Development"]
---

This is part 2 of binary character string encoding. In this part I discuss in detail how the encoding process was optimised even further to reduce the shape information to just a few bytes.

<!--more-->

In my [previous post]({{< ref "/posts/em-binary-encoding-decoding-part1.md" >}}) I discussed various optimisation steps that were taken to encode the character shape, colour and orientation information into just **48 bytes**.

Here I will take it one step further and explain how I managed to reduce the size even more.

### Optimisation 3: Decimal Representation

Looking at the binary char code for our shape matrix, one thing that jumped out to me immediately was that each row is an 8-bit word.

```
00111100
00111101
01101101
10111110
00011000
00001110
```

Also that each character in 8-bit word can be only in 2 different states i.e. black box or a white box.

Combining the binary nature of the colour information and how the shape can be arranged really easily in 8-bit words.

The *logical* (:sunglasses:) conclusion therefore was to try and represent each word in decimal so instead of taking 8 bytes per word, we will use no more than 3 bytes per word.

JS fortunately has functions available to convert between binary and decimal fairly easily.

For example, an 8-bit word represented by `00111100` can be converted to it's decimal representation using:

{{< highlight js >}}
String.fromCharCode(parseInt('00111100'), 2), 2);
{{< /highlight>}}

It follows that `00111100` in binary is equal to `60` in decimal. So by converting binary map to decimal we can save between 3 and 5 bytes per word. (8-bit binary can go up to 255 so it depends if the binary word is represented by a single digit decimal or 3 digit decimal).

Encoding our `space invaders` alien example from binary to decimal give us:

```
60
61
109
190
24
14
```

So here we've gone from **48 bytes** to just **14 bytes** for our alien!

| Optmisation        | Alien size | % Diff. form original |
|--------------------|------------|-----------------------|
| Original           | 77 bytes   | -                     |
| Mirroring          | 48 bytes   | -37.66%                |
| Decimal encoding   | 14 bytes   | -81.83%               |

### Optimisation 4: ASCII Char Codes

While I was working on my js1k project, I debugged an issue at work where our client had implemented some JavaScript code on their site which we had provided but it was failing on a string comparison.

Debugging it in console showed no difference in the strings but the comparator kept failing for whatever reason. After much headache I decided to compare `String.length` instead and it turned out that the string were different lengths although they were being printed out exactly the same.

***Invisible character***

Our client had copy pasted the code from an email client *cough* outlook *cough* which had added invisible characters to the code thus introducing the bug.

As painful as it was to discover that actual issue, it did gave me an idea for js1k: To use ASCII representation for my decimal encoded shape map. Because ASCII can represent entire 8-bit word spectrum and I don't really care if I end up in the invisible character range for this project.

So from previous step I computed ASCII representation for each decimal word by running:

{{< highlight js >}}
String.fromCharCode(60,2)
{{< /highlight>}}

That gave me:

{{< highlight js >}}
"<=m¾<0x18><0x0e>"
{{< /highlight>}}
*Note: I have typed out the hex representations for invisible characters so you can see them in the string, copy pasting these into a program to run will not yield the same result. For that, please generate the ASCII using fromCharCode.*

The entire shape information represented in just **6 bytes** :tada:

| Optmisation        | Alien size | % Diff. form original |
|--------------------|------------|-----------------------|
| Original           | 77 bytes   | -                     |
| Mirroring          | 48 bytes   | -37.66%               |
| Decimal encoding   | 14 bytes   | -81.83%               |
| ASCII              | 6 bytes    | -92.20%               |

ASCII can be unserialised and rendered to canvas as follows:

{{< highlight js >}}
const asciiChars = '<=m¾<0x18><0x0e>';
const sf = 6; //scaling factor;

for (let col = 0; col <= 10; col++) {
  for (let row = 0; row <= 7; row++) {
    const charCode = asciiChars.charCodeAt(Math.abs(5 - col));

    if (charCode & 1 << row) { // using bitmask
        c.fillRect(col * sf, row * sf, sf, sf);
    }
  }
}
{{< /highlight>}}
*Note: The code above will render the alien shape upside down but that's because of some modification I've made to make these example easy to understand and to remain consistent between every step. With minor modifications it can be made to render the right way up. Also, I've typed out the hex representation for the invisible characters, please replace them with actual ASCII chars if you wish to run this in the browser.*

I can now encode both my alien variations in **12 bytes** which is awesome but I have yet another trick up my sleeve which I will leave for another day :)

***to be continued ...***
