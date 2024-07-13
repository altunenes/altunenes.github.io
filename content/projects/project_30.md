+++
title = "Asahi illusion"
description = "A live/interactive GLSL demo representing the Asahi illusion"
weight = 30
date = "2024-01-01"

[extra]
local_image = "images/asahi.png"
[taxonomies]
tags=["illusion","asahi","perception","demo"]
+++

Read my story on [here](https://altunenes.github.io/posts/asahi/) about this illusion.


<div align="center">
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MX23Wz?gui=true&t=10&paused=true&muted=false" allowfullscreen></iframe>
</div>

To change the colors, you can play with the following functions and observe :-)

```GLSL
    vec3 color = colorDirection > 0.0 
                 ? mix(vec3(1.0, 1.0, 0.0), vec3(0.0, 0.0, 0.0), r / (size * 1.0))
                 : mix(vec3(0.0, 0.0, 0.0), vec3(1.0, 1.0, 0.0), r / (size * 1.0));

    return vec4(color, petalMask);
}
```

