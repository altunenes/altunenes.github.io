+++
title = "Reading brushstrokes, reading code"
date = "2026-11-13"

[taxonomies]
tags=["programming","AI","perception","creativity","thoughts"]
+++

People often treat AI assisted coding as welcome relief from routine labor, while treating AI generated images or text as a threat to creativity. Technically, both rely on the same kind of probabilistic models. Modern AI debuggers and code assistants do not run deterministic rules; they predict tokens from learned distributions, much like image models predict pixels. The difference in how we judge them comes down to perception, not mechanics.

Writing code is closer to painting than most non programmers assume. You start with a rough idea, hit constraints, rework your abstractions, and slowly shape the result, much like the revised limbs visible under "X rays" of old paintings *([Graham, 2003](https://paulgraham.com/hp.html))*. The finished works, however, hit the brain differently. Visual perception groups thousands of brushstrokes into a single image almost effortlessly. Code offers no such shortcut. Reading it barely engages the brain's language network and instead taxes the fronto parietal Multiple demand system used for working memory and formal reasoning *([Ivanova et al., 2020](https://doi.org/10.7554/eLife.58906))*.

because reading code is tiring and requires fluency, people outside the craft rarely see the decisions inside it. End users only interact with the surface interface, never the source text. Art history has seen similar blind spots, from early critics calling photography a mechanical copy of nature to galleries rejecting Duchamp's *Fountain* as plain plumbing. When you cannot read a medium, the work behind it looks purely mechanical.

Peter Naur wrote that a program is not its text, but the mental theory of how a rea world problem maps onto formal rules *([Naur, 1985](https://doi.org/10.1016/0165-6074(85)90032-8))*. That is why offloading the entire process to an LLM feels hollow, something I wrote about in an [earlier post](/posts/aianddopamine/). In both painting and programming, the hard part is building that mental model and deciding what belongs. Code only looks mundane because the craft is hidden from the people looking at the screen.