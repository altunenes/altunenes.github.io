+++
title = "How LLMs See Illusory Faces"
date = "2025-06-22"
[taxonomies]
tags=["llm","pareidolia","AI","vision","perception"]
+++

#### <span style="color:orange;"> Background </span>

During my master's studies, I became fascinated with how the human brain processes illusory faces, a phenomenon known as face pareidolia. My research specifically focused on the role of different **spatial frequencies**—the building blocks of an image—in triggering this fascinating illusion. This led to an [ERP paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4341900) exploring how our brain uses these frequencies to see real versus illusory faces. The paper is still under review, but the core ideas about perception inspired this new line of questioning.

With the incredible rise of powerful Vision-Language Models (VLMs), a new question emerged: how do these "artificial minds" perceive the same illusions? I think probing their "vision" isn't just an academic curiosity. It might helps us understand their inner workings, question our implicit assumptions when building LLM apps, and appreciate the subtle but significant gap that still exists between artificial and biological sight.

#### <span style="color:orange;"> What is Spatial Frequency and Coarse-to-Fine Theory? </span>

At its core, any image can be broken down into its fundamental visual components. **Low Spatial Frequencies (LSF)** represent the broad, blurry, large-scale shapes—the "gist" of an image. **High Spatial Frequencies (HSF)** represent the sharp edges, fine lines, and intricate textures.

**Coarse-to-Fine** theory is a cornerstone of human vision. It posits that our brain processes visual information with incredible efficiency by first grasping the LSF "gist" of a scene in a fraction of a second. This rapid, coarse understanding then guides the slower, more deliberate analysis of the HSF details. It's a predictive, top-down strategy that allows us to make sense of a complex world with remarkable speed.

| ![SF](/images/sf.png)|
|:-:|
| *Visualizing the Coarse-to-Fine theory. The left shows an image broken into coarse (Low Frequency) and fine (High Frequency) information. The right shows how the brain processes the coarse 'gist' first to guide perception.*|

To digitally manipulate these frequencies, we can use tools like the Butterworth filter. For anyone interested in experimenting with this, I developed a Rust crate and a WebAssembly (WASM) demo that you can use to apply these filters to your own images. You can find it here: [butter2d](https://altunenes.github.io/butter2d/). I conducted all my experiments using this.


#### <span style="color:orange;"> How LLMs see illusions </span>

Research shows that modern VLMs are not objective, infallible observers; they can be "fooled" by classic visual illusions, and their susceptibility often increases with model scale [Shen et al., 2023](https://arxiv.org/abs/2311.00047) . This suggests they are learning statistical heuristics from their training data that mimic human perception, rather than developing a deep, structural understanding of the world. 

This makes pareidolia a particularly interesting test. It's not a geometric trick, but an illusion driven by a powerful, top-down, and likely evolutionary bias to find faces in our environment.

### <span style="color:orange;"> The Experiment </span>

To test how Gemini handles pareidolia, I took several images and created three versions of each (using butterworth filter): the original **Broadband (BB)**, a blurry **Low Spatial Frequency (LSF)** version, and a sharp-edged **High Spatial Frequency (HSF)** version. I then fed them to the model with a simple prompt. To avoid any "memory" or context-priming effects, each images was processed in a completely separate session.

Here was the prompt:
> "What are the three most prominent objects you see in this image? Respond in a JSON format where each object has a 'name' and a 'confidence_score'."

Here are the results.

#### <span style="color:orange;"> Test 1: The Wire Face </span>

This image of server cables, which I found on X (formerly Twitter), has an uncanny facial structure. As hypothesized, the LLM completely missed the face in the broadband and HSF versions, describing only the literal content. However, when presented with the LSF version, where only the coarse, global shape remains, it immediately and confidently identified a **"Face"**. So the fine HSF details of the wires and components seem to break the illusion for the model, while the blurry LSF version provides the ideal template for a face.

| ![W](/images/W.png)|
|:-:|
| *LSF (left), Broadband (middle), HSF (right) versions of the 'Wire Face'.*|

#### <span style="color:orange;"> Test 2: The Church Face </span>

Next, I used one of the most famous pareidolia images on the internet. My rationale was that if the model's perception is purely a function of its training data, it would have surely seen this image and would recognize the face (or illusory face). Once again, it failed to see the face in the BB and HSF versions, focusing only on the architecture. But in the LSF version, it correctly identified a **"Face (Pareidolia)"**. This suggests the model's failure isn't just about a lack of training data. The high-frequency details of the building's facade actively mask the illusion for the AI, even for a classic example.

| ![C](/images/C.png)|
|:-:|
| *The famous 'Church Face' pareidolia.*<br>*Note that the model only sees a "Face (Pareidolia)" in the LSF version, describing only architectural elements in the others.*|

#### <span style="color:orange;"> Test 3: The Oval Hypothesis </span>

The previous results sparked a new idea: perhaps the LLM's internal "face template" is strongly biased towards the oval, rounded shapes of human faces? The "Wire Face" is very angular. To test this, I selected two images with a more circular structure.

The first, an electrical component, followed the now-established pattern. A **"Illusory Face"** was detected in the LSF version, but the BB and HSF versions were seen only as literal machine parts.

| ![E](/images/E.png)|
|:-:|
| *An illusory face in an electrical component.*<br>*The LSF version triggered a "Pareidolia Face" detection, while the detailed versions only yielded descriptions of machine parts.*|

The second image, however, gave a breakthrough result. This time, the LLM saw a face in **all three versions!** It identified a "Face" in both LSF and BB, and even a **"Illusory Face"** in the sharp HSF image. This is a brilliant finding. It suggests that when an object's structure is a strong enough match for the AI's internal face template (a round shape, two distinct "eyes," a "mouth"), it can overcome the distracting HSF noise. This is also highly consistent with human vision, where HSF information is vital for analyzing the fine features *of a face* once it has been detected.

| ![M](/images/M.png)|
|:-:|
| *A illusory face on a can or clock mechanism that the LLM saw in all versions.*|

#### <span style="color:orange;"> Test 4: The Illusory Robot </span>

This next test uses a common object that happens to have face-like features: a set of viewpoint binoculars. The results show another interesting form of interpretation by the model. In the LSF version, the blurry shape with two prominent circles triggers an anthropomorphic classification: **"robot"**. The model defaults to a familiar humanoid template. However, once the HSF details are available in the broadband and sharp versions, the model corrects its initial "guess" and accurately identifies the object as **"viewpoint binoculars"**.

| ![R](/images/R.png)|
|:-:|
| *An LSF-induced "robot" is corrected into binoculars with more detail.*|

#### <span style="color:orange;"> Test 5: The Holistic Face Paintings </span>

Finally, I moved from pure pareidolia to a different kind of illusion: composite portraits, famously painted by artists like Giuseppe Arcimboldo. In these images, the face is **intentionally** constructed from other objects. These aren't really pareidolia in the same way; they are deliberate artistic constructions where recognizing the face requires **holistic processing**—seeing the overall arrangement rather than just the sum of the individual parts. How would the LLM fare?

The first painting is a face constructed from a landscape. Interestingly, the model identified a **"large face"** in both the broadband and high-frequency versions. This is a departure from the earlier pareidolia examples. Here, the individual components (trees, rocks) don't look like facial features on their own, but their careful arrangement creates a powerful holistic impression that the model was able to perceive, even with all the fine details present.

| ![P](/images/P.png)|
|:-:|
| *A composite face, testing holistic perception.*<br>*Notably, the model identified the face in all three versions, even describing it as an 'optical illusion' in the BB.*|

I tried a second, similar painting of a shepherd in a landscape forming a face. The results were just as intriguing. In the full-detail versions, the model successfully identified both the whole (**"Face"**) and the parts (**"Sheep"**, **"Shepherd"**). It seemed to parse the image on multiple levels simultaneously.

However, an interesting twist occurred in the LSF version. The blur, which helped reveal faces in the pareidolia examples, seemed to *weaken* the illusion here. The model's top guess for the LSF version was **"Tree"**, not "Face". This might suggest that for these complex, deliberately constructed images, the precise arrangement and HSF details are actually *critical* for the holistic face to emerge, and blurring them can break the carefully crafted composition. It's a fascinating case where the general rule (LSF reveals faces) is reversed, highlighting the complexity of both human and machine perception.

| ![P2](/images/P2.png)|
|:-:|
| *Another composite face, where blur seemed to hinder, rather than help, perception.*|

### <span style="color:orange;"> Conclusion and Final Thoughts </span>

These little experiments, while far from a rigorous scientific study, offer a fascinating glimpse into the perceptual world of a modern VLM. For anyone working with or building on top of these AI systems, I believe understanding these kinds of behaviors is incredibly important. An AI's failure to see a pattern that is obvious to us—or its tendency to see one only under specific conditions like blurring—highlights the inherent differences in how they process visual information.

This method of probing with spatial frequencies and illusions could serve as a simple, intuitive benchmark for tracking the progress of future vision models. As new architectures are developed, seeing how they handle these edge cases can tell us a lot about whether they are developing more robust, human-like perception or simply becoming better at pattern-matching their training data.

Of course, there are clear limitations here. I only used one model, Gemini 2.5 Pro, primarily because company I work provides free access to it. Other powerful models from OpenAI, Anthropic, or elsewhere might react to these images in completely different ways. The number of images was also small.

Also you might be realized that model's own distinction between a "Face" and a "Pareidolia Face". What is the difference? When the model uses the simple "Face" label, does it believe it's seeing a real person or animal? Is the "Pareidolia" tag an admission that it recognizes the illusion?
 

Perhaps the most important takeaway is the value of using novel stimuli. The real test for these models isn't showing them the famous "Church Face" again, but presenting them with the new, illusory faces we discover in our daily lives—a pattern in a coffee stain, the front of a new car, or a strangely-shaped vegetable. These "wild" pareidolia images, which the AI could not have been trained on, are the truest test of whether they are learning to *see* or just to *recognize*. And for me, that's an experiment that never gets old.