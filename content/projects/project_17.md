+++
title = "The Enigma"
description = "Leviant's Optical Illusion"
weight = 17
date = "2024-01-01"

[extra]
local_image = "images/enigma.png"
tags=["illusion","perception","rust","vision","demo"]

+++


Stop the shader and focus your gaze on the center  Over time, you will experience a vibrant and shimmering effect(?) :octopusballoon: It also may cause the perception of rotatory motion in the circles. Does the effect increase when there is a real motion?


<div align="center">
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/mt3Xzs?gui=true&t=10&paused=true&muted=false" allowfullscreen></iframe>
</div>


Originated by Isia Léviant (1982)   

An interesting paper by Zeki et al. (1993) revealed that our brain's V5 area, responsible for perceiving motion, can actually be activated by this illusion, even when there's no real motion involved. It suggests that our perception of motion is closely linked to activity in V5, even without any actual movement. 
https://royalsocietypublishing.org/doi/10.1098/rspb.1993.0068

But this paper not explaining the actual cause of this motion  . The paper is only about the V5. 


Above, GLSL version could work slow, but try my Rust version instead for better performance with various real time adjustments:
Note Source [Code](https://github.com/altunenes/rusty_art)


 Software Version | Operating System | Download Link                                                                                     |
|------------------|------------------|----------------------------------------------------------------------------------------------------|
| **Enigma**        | macOS            | [Download](https://github.com/altunenes/rusty_art/releases/download/v1.0.4/leviant-macos-latest.zip) |
|                  | Ubuntu           | [Download](https://github.com/altunenes/rusty_art/releases/download/v1.0.4/leviant-ubuntu-latest.zip)|
|                  | Windows          | [Download](https://github.com/altunenes/rusty_art/releases/download/v1.0.4/leviant-windows-latest.zip)|
