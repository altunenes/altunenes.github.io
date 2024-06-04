+++
title = "Galton Board"
description = "A live/interactive JavaScript demo representing the normal distribution"
weight = 29
date = "2024-01-01"

[extra]
local_image = "images/galton.png"
[taxonomies]
tags=["statistics","galton","gaussian","demo"]
+++

There is more than enough about Galton Board, but the code/interactive version is almost like an oasis in the desert! So in my spare time I sat down and created something like this:

Note: The “Matter” physics engine is really cool!  :-)

<div align="center">

<iframe src="https://openprocessing.org/sketch/2291810/embed/" width="800" height="700"></iframe>
</div>

To change the initial randomness, you can play with the following functions and observe :-)

```JavaScript
function addPrt() {
    let spcY = height / (rws + 7);
    let rndX = width / 2 + random(-0.02 * width, 0.02 * width);
    prts.push(new Prt(rndX, spcY / 2, 0.01 * height));
    pc++;
}

function addPrts(spcY) {
    for (let i = 0; i < maxPc; i++) {
        let rndX = width / 2 + random(-0.02 * width, 0.02 * width);
        prts.push(new Prt(rndX, spcY / 2, 0.01 * height));
        pc++;
    }
}
```

