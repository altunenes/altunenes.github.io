+++
title = "How to render fire as real geometry on the GPU"
date = "2026-07-28"

[taxonomies]
tags=["rust","bevy","gamedev","willcaster","shaders"]
+++

There are a lot of ways to put fire in a game, and almost all of them are a trick
in a fragment shader. You take a flat quad, point it at the camera, and colour the
pixels so they look like flames. The geometry is just a billboard, the fire only
lives in the pixels. That's what ships in
[Willcaster](https://store.steampowered.com/app/4953880/Willcaster/) and it works
fine.

This post is about a different one I don't really see people do: growing the fire
out of **real mesh**, on the GPU, at runtime. It looks different, it happens to be
cheaper to run, and the pipeline behind it is the actual reason I built any of it.

<div align="center">

<video width="100%" controls loop playsinline aria-label="Side by side comparison of the same brazier in Willcaster. On the left, the shipped raymarch flame: soft, red, slightly transparent. On the right, the compute-generated mesh flame: brighter, sharper, made of extruded ribbons of geometry.">
  <source src="/videos/fire.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

*Same brazier, both techniques. Left: the raymarch flame that ships in the game. Right: the same fire grown as real mesh on the GPU.*

</div>

## <span style="color:orange;"> The normal way, and what it actually costs </span>

My brazier fire is a small raymarch. For every pixel the flame covers, the shader
walks 8 steps up through an imaginary column of fire, and at each step it samples
some noise to decide how thick the flame is there. That's a few hundred little
calculations per pixel, and it happens every frame, for every pixel of fire on
screen.

That last part is the important bit. Walk the camera closer so the fire fills more
of the screen and it gets more expensive, even though nothing about the fire
changed. You pay by the pixel, forever.

It's cheap enough here (the braziers are small on screen), so I never worried
about it. But it set up the contrast that made the other approach interesting.

## <span style="color:orange;"> Why I built this in the first place </span>

The grass in Willcaster grows on the GPU. A compute shader fills a buffer with one
entry per blade, a vertex shader reads that buffer and builds the actual bent
geometry for each one, and nothing touches the CPU after setup.

The bezier part isn't my idea. Back in the spring I wrote a little ShaderToy,
[*Cubic Bezier Grass*](https://www.shadertoy.com/view/7fSSRc), messing with the
math from Sucker Punch's [*Procedural Grass in Ghost of
Tsushima*](https://www.youtube.com/watch?v=Ibe1JBF5i5Y) talk: each blade is a
cubic bezier spine, you take the derivative to get the tangent, and use that to
extrude a tapered width so the blade has volume and catches light. ShaderToy has
no vertex stage, so to even see it I had to raymarch the geometry per pixel, which
is wildly expensive and was never the point. It was just where I got the math
right before moving it onto the GPU properly.

But grass is just decoration here, a bit of green on the border tiles. It isn't
why I wrote any of this. Willcaster is a game about casting spells, and I don't
want those spells to be flat sprites stuck to the camera. I want them to be real
geometry sitting in the world: ice that catches a highlight, gold that actually
looks like metal, fire that has a shape when you walk around it. That's the payoff
of doing it this way. Because the result is ordinary mesh, it drops onto the
engine's normal lighting, depth and shadows for free, the same PBR every other
object gets. A billboard fakes all of that and falls apart the moment the camera
moves. Grass was just the cheapest thing to prove the pipeline on. Fire is the
second, and the spells are where it's actually headed.

The shape of it is small. Bevy lets you add your own passes to its render graph,
so there's a compute pass that fills a plain GPU buffer, and then that same buffer
is handed to an ordinary material as a read-only input, so Bevy's normal mesh
drawing reads it in the vertex shader. No custom renderer, no custom draw calls, no
extra crates. I fill a buffer, Bevy draws a mesh, and the vertex shader reads the
buffer to decide where each vertex goes.

The whole trick is really just this one struct. The compute pass writes into the
buffer, and the material holds it as a normal read only binding the vertex shader
can read:

```rust
#[derive(Asset, TypePath, AsBindGroup, Clone)]
pub struct FlameMaterial {
    #[storage(0, read_only, visibility(vertex))]
    tongues: Handle<ShaderBuffer>,
}
```

That `visibility(vertex)` is the important part. It says the buffer is readable
from the vertex stage, which is what lets an ordinary material grow geometry from
compute output without me writing any draw code at all.

## <span style="color:orange;"> Flames as ribbons </span>

The compute pass does the same job as it does for grass, except each entry is now
a "tongue" of flame instead of a blade:

```wgsl
struct Tongue {
    pos_height: vec4<f32>,  // where it sits on the base + how tall
    params: vec4<f32>,      // facing, curl, a flicker offset, a random seed
};

@compute @workgroup_size(64, 1, 1)
fn main(@builtin(global_invocation_id) id: vec3<u32>) {
    let i = id.x;
    if (i >= COUNT) { return; }

    // scatter the tongues in a disc, packed toward the middle
    let ang = hash1(i * 3u + 1u) * TAU;
    let rad = pow(hash1(i * 3u + 2u), 0.55) * DISC_RADIUS;
    let x = cos(ang) * rad;
    let z = sin(ang) * rad;

    // tallest in the centre, short around the rim -> a rounded flame shape
    let center_bias = 1.0 - rad / DISC_RADIUS;
    let height = 0.34 + center_bias * center_bias * 0.72 + hash1(i + 7u) * 0.22;

    tongues[i].pos_height = vec4<f32>(x, 0.0, z, height);
    tongues[i].params = vec4<f32>(facing, curl, phase, seed);
}
```

That's the entire "simulation". A few dozen tongues, placed once.

Then the vertex shader turns each tongue into a ribbon: a curve that starts at the
base, rises, leans over, and lashes side to side at the tip, so it looks like it's
licking upward instead of standing still.

```wgsl
// a curve up from the root that leans forward and whips at the top
let p0 = root;
let p1 = root + up * H * 0.33 + fwd * curl * ...;
let p2 = root + up * H * 0.66 + fwd * curl * ... + perp * sway;
let p3 = root + up * H        + fwd * curl * ... + perp * sway;
let center = bezier(p0, p1, p2, p3, t);

// widen it into a ribbon, wobble the edge with noise so it isn't a clean blade
out.local_pos = center + perp * (side * width);
```

The flicker, the licking, the wobble are all just `time` fed into that curve. The
tongues themselves never change after they're placed. The colour is a warm
gradient, white-hot at the base, cooling to orange and then red as it climbs.

The shaders were the easy half, honestly. The hard half was the Rust holding them
up: getting a buffer the GPU writes into to line up with a material the GPU also
reads from, on Bevy's render graph, without the two disagreeing about when things
exist. That took a few rewrites and one bug that only turned up on some frames,
and I'll spare you the exact shape of it here. If you're building something
similar and hit the same wall, come find me, I'm happy to compare notes.

The result is a flame you can turn on wireframe and see is genuinely made of
triangles. It's the one on the right in the clip up top.

## <span style="color:orange;"> Back to the cost </span>

Now the comparison pays off. The compute pass that places the tongues runs for a
handful of frames when the room loads, and then switches itself off, because the
tongues never move after that. From then on the fire is just a few dozen ribbons
drawn like any other model, plus a very cheap pixel shader. It doesn't care how
close the camera gets. The heavy lifting happened once, at the start, and never
again.

So the raymarch, the "simpler looking" one, pays for itself every frame per pixel.
The mesh fire, the "fancier" one, pays once and coasts.

The one thing the mesh version loses is softness. Real geometry has hard edges,
and no amount of noise fully hides that a flame is made of triangles, so it reads
a bit more stylised. Willcaster isn't a stylised game, so the soft raymarch is
what's on the braziers. But the mesh flame stays in the project as a toggle,
because I think it's a neat idea more people should play with, and because the
same pipeline will drive things like a wall of fire later, where hard directional
geometry is exactly what you want.


<div align="center">

<video width="100%" controls loop playsinline aria-label="Flame wall in Willcaster made with the same compute gen mesh tech.">
  <source src="/videos/des.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
Flame wall in Willcaster made with the same compute gen mesh tech.
*THIS IS FINE.*

</div>

## <span style="color:orange;"> Why bother </span>

The game didn't need any of this. The first flame was fine. But the grass pipeline
turned out to be a general tool rather than a grass thing, and pointing it at fire
cost me an afternoon and taught me more about where GPU time actually goes than
any amount of reading would have.

That's most of what these posts are going to be: small detours where I tried the
weird version of something and wrote down what happened.

**[Willcaster is on Steam](https://store.steampowered.com/app/4953880/Willcaster/)**
if you want to see where all of this ends up. Wishlists genuinely help a small
game like this a lot.

More soon.
