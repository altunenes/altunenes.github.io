+++
title = "Willcaster: four years of building a game about a frog who argues with you"
date = "2026-07-22"

[taxonomies]
tags=["rust","bevy","gamedev","willcaster"]
+++


<div align="center">

![Logo](/images/logo.png)

</div>

I put the Steam page up this week: https://store.steampowered.com/app/4953880/Willcaster/ , so it's finally time to write about the thing
I've been building in my free time for about four years: **Willcaster**, a grid
roguelike made in Rust with [Bevy](https://bevy.org/).

## <span style="color:orange;"> The idea </span>

You're a wizard, and you don't control the character.

You draw a path across the grid with your mouse, and a frog knight walks it.
That's it, that's the input. You draw, he walks, he fights whatever he runs into
on the way.

The catch is that he has opinions about your path. Before he walks, he tells you
what he wants from the room: the gold you routed around, the food tile, or just
to stay away from that elite over there. Whatever path you actually draw is your
answer to him. Ignore him enough times and he starts adding his own detours,
hesitating in fights, and eventually he just refuses to walk the path at all.

Pretty much everything else in the game grew out of that one idea.

<div align="center">

| ![Willcaster current build](/images/willcaster.png)|
|:-:|
| *The board today.*|

</div>

## <span style="color:orange;"> Four years of placeholders </span>

It didn't look like that for most of its life. Here's the first playable
version:

<div align="center">

| ![Early prototype](/images/4.png)|
|:-:|
| *Colored quads, `X` for walls, two letter enemy codes. Everything here is a placeholder except the rules, and it was fully playable.*|

</div>

Two years later it was 3D and still almost entirely placeholders, capsules for
enemies and a purple sky I never meant to keep:

<div align="center">

| ![Mid prototype](/images/2.png)|
|:-:|
| *Better, sort of.*|

</div>

This is a nights and weekends project, so there were long stretches where
nothing happened at all. The game isn't huge, it's just complicated, and there's
one of me.

## <span style="color:orange;"> On Bevy </span>

I've been on Bevy for years now and I'd pick it again.

The ECS is the main reason this design survived. The frog has traits, desires,
stress, a trust level, a combat AI, and an opinion about your path, so there are
a lot of systems that need to read each other's state. In a more classical
engine I'm fairly sure this would have turned into one giant `Character` class
that nobody wants to touch. Here it's a pile of small components and small
systems instead, and adding something new usually means writing a new system
rather than editing an old one. The whole disobedience layer came very late in
development and it plugged in without me touching the walking code at all.

The other thing I like is how little I depend on. This is the entire dependency
list for the game:

```toml
[dependencies]
bevy = { version = "0.19.0", features = ["jpeg", "ktx2", "basis-universal", "wav"] }
dirs = "6"
log = "0.4.29"
rand = "0.9"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

Bevy, a random number generator, JSON, and a crate that finds the save
directory. That's on purpose. Almost everything you see on screen I wrote
myself, and a surprising amount of it isn't art assets at all, it's shaders.
The special tile runes, the path ribbon, the poison gas, the portals, the god
rays, the grass, most of the spell effects: those are WGSL files, not models. I
kept trying to solve those with props from asset stores and kept ending up
writing a shader instead. I also created a specular anti aliasing pipeline specifically for this game, 
and I even opened a PR on Bevy in case it might be useful to others.
see: https://github.com/bevyengine/bevy/pull/24427

The tax you pay is that Bevy moves fast. I've migrated this project across a lot
of versions and each one costs an afternoon or three of renaming things. It's
never hard, it's just tedious, and honestly it's a fair price for an engine
that's improving this quickly.

## <span style="color:orange;"> I don't balance this game by feel </span>

This is the part I get asked about most, so it's worth explaining.

A frog who disobeys you 3% of the time and a frog who disobeys 12% of the time
are completely different characters. One is a loyal companion who surprises you
now and then, the other is an unreliable jerk. And I cannot tell those two apart
by playing the game. Twenty runs is nowhere near enough samples. I'd just be
reacting to whatever happened to me last Tuesday.

So playtesting isn't how the numbers get decided here. Every system gets a
design document in JSON, then a Monte Carlo simulator in Python, then thousands
of simulated runs across several different player archetypes (the obedient one,
the selfish one, the tyrant, the careless one), and only then do real numbers go
into the Rust code. There are 25 of those design documents and 11 simulator
scripts sitting in the repo right now.

It's saved me more than once. When I recently merged two of the game's meters
into one, the first simulation run passed all six of its checks. Then I looked
closer and realised three of the checks were measuring the wrong thing. One of
them was supposed to prove a player could dig themselves out of a hole, except
the archetype it tested never actually reached the bottom of the hole to begin
with. After I fixed the checks, a passive income mechanic I'd been quite pleased
with turned out to make the endgame unlock basically free: the selfish playstyle
was reaching it in 98% of runs.

There's no version of me that finds that by playing. A character whose whole
appeal is that he might say no is a character defined by a frequency, and
frequencies have to be measured.

## <span style="color:orange;"> Where it is now </span>

The core systems are done. What's left is balance, polish, sound, and all the
things that only show up once other people get their hands on it.

**[Willcaster on Steam](https://store.steampowered.com/app/4953880/Willcaster)**
— wishlists genuinely matter a lot for a small game like this.

I'll write more focused posts as I go: the compute pipeline, the grass, and some shader tricks that I implemented on the game... 

