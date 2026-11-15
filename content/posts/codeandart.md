+++
title = "Reading brushstrokes, reading code"
date = "2026-11-13"

[taxonomies]
tags=["programming","AI","perception","creativity","thoughts"]
+++

There is an odd split in how people talk about generative AI. Automating code is widely treated as welcome relief from routine, mechanical labor, while automating painting or prose is treated as crossing a creative line.

Technically, the boundary does not hold. Traditional compilers and static analyzers are deterministic, whereas modern AI code generators and debuggers are probabilistic models. They predict tokens from learned distributions much like image models predict pixels. So the divide people feel between AI code and AI art is not coming from how the tools work. My guess is that it comes from how humans perceive the two crafts.

## <span style="color:orange;"> Holistic eyes, sequential code </span>

Writing code is a creative process, much like painting. A painter builds an image out of thousands of small circles and brushstrokes, hits resistance on the canvas, and invents new techniques to work around it. As Paul Graham noted in [*Hackers and Painters*](https://paulgraham.com/hp.html), X-rays of master paintings routinely show moved limbs and reworked faces. Coding works the same way. You start writing, hit a logical wall, rethink your abstractions, and keep going.

The difference is how the finished work hits the human brain. Looking at a painting is effortless because visual perception groups individual marks into a coherent whole right away. Nobody stands in a museum trying to mentally trace every stroke, because that would be exhausting.

Source code does not give you that shortcut. Reading code barely activates the brain's natural language network; instead, it relies heavily on the Multiple Demand system, the fronto-parietal network responsible for working memory and formal reasoning *([Ivanova et al., 2020](https://doi.org/10.7554/eLife.58906))*. Reading code is slow, sequential, and tiring. If you do not know the language, it is completely opaque. When people cannot see the craft behind an output, they tend to assume there was not any.

## <span style="color:orange;"> Missing the medium </span>

Dismissing a work because you lack the literacy to read its process is nothing new. In 1859, Charles Baudelaire called photography "the refuge of all failed painters," seeing only a mechanical copy of nature. In 1874, critics mocked Monet's *Impression, Sunrise* as unfinished wallpaper because they misread his deliberate color constraints as sloppiness. And when Marcel Duchamp submitted *Fountain* in 1917, it was rejected as plain plumbing. As Duchamp put it, whether he made the object with his own hands did not matter; he chose it and created a new thought for it.

Code suffers from a double invisibility. Most people cannot read it, and even when they use the software every day, they never see the source text at all. They only see the final interface, so the architectural choices underneath look like mechanical plumbing.

## <span style="color:orange;"> Where the work actually lives </span>

Peter Naur argued that a program is not its source code, but the living mental theory of how a messy real-world problem maps onto formal rules *([Programming as Theory Building](https://doi.org/10.1016/0165-6074(85)90032-8))*. That is also why blindly offloading the process to an LLM feels hollow, something I touched on in an [earlier post](/posts/aianddopamine/).

Whether you are painting or writing software, the creative act was never in dragging bristles across a canvas or typing brackets into an editor. It lives in building a mental model, imposing constraints that force clarity, and choosing what belongs. Treating code generation as harmless automation while treating image generation as a moral crisis misses that both target the same human habit: making sense of things through structure.