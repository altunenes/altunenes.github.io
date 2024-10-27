+++
title = "Adelson's Checker-Shadow Illusion"
description = "A live/interactive GLSL demo representing the 3D illusion"
weight = 31
date = "2024-01-01"

[extra]
local_image = "images/adelson.png"
[taxonomies]
tags=["illusion","adelson","perception","demo"]
+++

Took a break to code this classic visual illusion - looks can be deceiving! The two squares marked with red dots are exactly the same color (RGB: 95,95,95). Don't believe me? Screenshot on bellow illusion and try to use a simple color picker on this [**<font color="red">website</font>**](https://imagecolorpicker.com/).

<div align="center">
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MXjBW3?gui=true&t=10&paused=true&muted=false" allowfullscreen></iframe>
</div>

# <span style="color:orange;">Explanation</span>

Your visual system isn't just measuring light - it's solving a complex problem: determining true object colors despite shadows. Here's how it works:

1️⃣ Local Contrast: Your brain compares each square with its neighbors. The light square in shadow is surrounded by darker ones, so your brain says "must be white!" Even though it's physically dark (RGB: 95,95,95).

2️⃣ Shadow Detection: Notice how the shadow has soft edges while the checkers have sharp ones? Your brain uses this to separate shadow effects from actual surface colors.

🎯 The Key Point: This isn't a "failure" of vision - it's your brain successfully interpreting a 3D world! It cares more about true object properties than being a perfect light meter.


I also coded this in Rust/wgsl, you can find on here:

https://github.com/altunenes/rusty_art/blob/master/src/adelson.rs
