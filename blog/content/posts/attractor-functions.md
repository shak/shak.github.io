---
title: "Attractor Functions"
date: 2021-12-30T18:33:28+04:00
draft: false
tags: ["P5JS", "Development", "Attractor Functions"]
categories: ["Digital Art"]
---

I recently discovered [p5js](https://p5js.org/) that takes all the hardwork out of rendering on canvas and allows you to focus on the functional part of the application. As an experiment, I decided to investigate attractor functions.

<!--more-->

I do not understand the mathematics of attractors to great extent and therefore I am unable to dig into the details but roughly speaking an attractor function defines a set of states towards which the system will tend to evolve.

These functions are defined as a set of equations in n number of dimensions. The functions also define parameters which determine the - to put it in very non-technical terms - state the system will tend towards.

## Clifford Attractor

The first attractor I tried was Clifford attractor, mainly for it's simplicity.

{{< highlight js >}}
//a, b, c & d are the function parameters
// xn & yn are the value from previous iteration

x = sin(a * yn) + c * cos(a * xn)
y = sin(b * xn) + d * cos(b * yn)

{{< /highlight >}}

I ran it for 10 million points with each point iterated 5 times using the Clifford attractor.

{{% figure src="/img/clifford_coloured.png#center" alt="Image showing clifford attractor function rendered using points" %}}

## Jason Rampe Attractor

Jason Rampe was is an iteration on Cifford function using the same dimensions and similar number of parameters.

{{< highlight js >}}
//a, b, c & d are the function parameters
// xn & yn are the value from previous iteration

x = cos(b * yn) + c * sin(b * xn)
y = cos(a * xn) + d * sin(a * yn)
{{< /highlight >}}

I ran it for 10 million points with each point iterated 5 times using the Jason Rampe attractor.

{{% figure src="/img/rampe_coloured.png#center" alt="Image showing Jason Rampe attractor function rendered using points" %}}