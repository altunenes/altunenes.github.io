+++
title= "Using Euclidian Distance for Emotion Similarities in VAD space"
date= "2023-09-30"
[taxonomies]
tags=["emotions","face","computer vision","method"]
+++

## <span style="color:orange;"> Is the VAD Model a Good Metric for Emotion Recognition in Vision Algorithms? </span>


Emotion recognition, particularly in the realm of vision, is a challenging task. Traditional methods often rely on labeling facial expressions with discrete emotion labels. However, human emotions are complex and multi-dimensional, and a simple label might not capture the full spectrum of an individual's emotional state. Enter the Valence-Arousal-Dominance (VAD) model, a promising approach that offers a more nuanced representation of emotions.

### <span style="color:orange;">The VAD Model in a Nutshell</span>

The VAD model is built on three independent dimensions:

- **Valence (Pleasure)**: Ranges from unhappiness to happiness.
- **Arousal (Affective Activation)**: Varies between sleep and excitement.
- **Dominance (Level of Control)**: Spans from submissive to dominant.

Each emotion, such as anger or happiness, is represented as a point in this three-dimensional space. For instance, anger might be represented by a negative valence (unhappiness), high arousal (excitement), and a moderate level of dominance.

<p align="center">
  <img src="/images/vad.png" alt="VAD Model" width="300" height="300">
</p>

### <span style="color:orange;">Why VAD Over Traditional Labeling?</span>

Traditional emotion recognition methods label an emotion based on the most dominant facial expression. However, emotions are rarely so black and white. A person might be feeling a mix of sadness and surprise, or anger with a hint of fear. The VAD model, by representing emotions in a continuous space, can capture these nuances.

Moreover, by using the VAD model, we can calculate the Euclidean distance between the predicted emotion and the ground truth. This provides a quantitative measure of how accurate the prediction is, rather than a simple right or wrong label. But of course, most of times we are just using these models so my blogppost not focusing on these "ground truth" scores. 


### <span style="color:orange;">Emotion Similarities and Dissimilarities</span>

An interesting application of the VAD model is the ability to measure the similarity between different emotions. Using the formula:

$$\text{distance} = \sqrt{(v_1 - v_2)^2 + (a_1 - a_2)^2 + (d_1 - d_2)^2}$$

where $$v$$, $$a$$, and $$d$$ represent valence, arousal, and dominance respectively, we can determine the distances between emotions in the VAD space. For instance, sadness and fear are found to be the most similar emotions, while happiness and fear are the least similar. This kind of analysis provides a deeper understanding of human emotions and their inter-relationships.

### <span style="color:orange;">Distances Between Emotions</span>

Using the VAD model, we can determine the "distance" between different emotions. For example:

- Distance between Angry and Happy is: 1.21
- Distance between Angry and Sad is: 0.81

These distances provide insights into how similar or dissimilar two emotions are in the VAD space. A smaller distance indicates that the two emotions are more similar in terms of valence, arousal, and dominance, while a larger distance suggests greater dissimilarity.

### <span style="color:orange;">Conclusion</span>

The VAD model offers a multi-dimensional perspective on emotions, allowing for a more nuanced understanding than traditional labeling methods. By evaluating emotions in the VAD space, we can quantitatively measure the similarities and differences between emotions, which is crucial for vision algorithms. In the realm of vision and emotion processing, understanding these subtle differences can lead to more accurate and holistic emotion recognition.

### <span style="color:orange;">References</span>

- Sampaio, E. V. B., Lévêque, L., da Silva, M. P., & Le Callet, P. (2022). *Are Facial Expression Recognition Algorithms Reliable in the Context of Interactive Media? A New Metric to Analyse Their Performance*. EmotionIMX: Considering Emotions in Multimedia Experience (ACM IMX 2022 Workshop), Aveiro, Portugal. [HAL-03789571f](https://hal.science/hal-03789571/document)