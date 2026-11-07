+++
title = "Coarse to fine in language models"
date = "2026-11-07"
[taxonomies]
tags=["coarse-to-fine","pareidolia","AI","vision","perception"]
+++

#### <span style="color:orange;"> Coarse to fine in language models </span>

Coarse to fine is an old theory of human vision. The brain first gets the blurry gist of a scene from low spatial frequencies, and uses it to guide the slower analysis of details from high spatial frequencies. The coarse version is cheap and fast, and once you have it, the details are easier. I explained it a bit more in my [earlier post](/posts/pareidolia/).

This week I read [Next-Concept Modeling](https://arxiv.org/abs/2610.14162) (Kumbong et al., 2026), which uses the same idea for text. The model does not write one word after another. First it compresses text into a few levels of codes. The first level is short and holds the rough, global structure, and each next level adds more detail. A transformer then generates one level at a time, and all codes inside a level in parallel. They report 10 to 31 times faster generation than the best diffusion language models at similar quality, and better results on Sudoku.

Image models already went this way. [VAR](https://arxiv.org/abs/2404.02905) does "next scale prediction", starting from a tiny image and going up in resolution. And Sander Dieleman showed that diffusion models are a kind of [spectral autoregression](https://sander.ai/2024/09/02/spectral-autoregression.html): they build the low spatial frequencies first and the high ones later. So now text is following too, for the same reason the theory gives for the brain: it is efficient.

What I find funny is that in my own work coarse to fine did not always hold. In the [earlier post](/posts/pareidolia/) Gemini saw illusory faces mostly in the blurry, coarse versions of the images. But in my [EEG study](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4341900) (now under review), the human brain did not show a face response to illusory faces from the coarse version at all, only a weak one from fine details. Blur a socket and you get two dark blobs and a line, almost the definition of a face, and still the brain did not take it. So here the model followed the textbook, not us.

There is another way to read my Gemini result, though. A lot of text on the internet is people saying "this looks like a face" about sockets, cars and clouds, and the blurry versions look more like those typical pictures. So the model may be giving the answer it knows humans give, more than seeing a face. In EEG terms, it could be all behavioural report and no N170. And asking the model how it decided doesn't help much, because what a model says about its own processing is also just generated text.

My participants also said "face" almost every time. The interesting part was only in the EEG: when the face response came and from which frequencies. Coarse to fine models have explicit levels, so for once we could ask the same question to a machine: at which level does it decide there is a face?
