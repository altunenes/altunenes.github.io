+++
title= "Randomness: CPU & GPU"
date= "2025-08-09"
[taxonomies]
tags=["GPU", "CPU", "Randomness", "Shadertoy", "Rust"]
+++

Traditional random number generation, common on CPUs in languages, often relies on a sequential process. A generator object holds an internal state, and each request for a number updates that state for the next call. This approach is fundamentally incompatible with the massively parallel nature of GPUs, where thousands of threads need to generate numbers independently and simultaneously. If all threads tried to access and update a single shared state, it would require complex synchronization that would effectively destroy the GPU's performance advantage.

GPUs "solve" this by adopting a "stateless" or functional approach. Instead of updating a state, each thread computes its random number directly by calling a function that scrambles its inputs. In GLSL shaders, a common technique is to use a hash function that takes a thread's unique coordinates (`gl_FragCoord.xy`) and often the current time (`iTime`: in shadertoy) as input. This ensures every pixel gets a different, independent number on every frame. A simple, widely-used hash function looks like this:

```glsl
// A simple hash function that turns a 2D vector into a pseudo-random float.
// Adding a time input makes it dynamic for animations.
float hash(vec2 p, float t) {
    vec2 p_with_time = p + vec2(t);
    return fract(sin(dot(p_with_time, vec2(12.9898, 78.233))) * 43758.5453);
}

// You would call it in your main shader code like this:
// (iTime is a standard uniform in environments like Shadertoy)
float randomValue = hash(gl_FragCoord.xy, iTime);
```
<span style="color:orange;">The Real-World Bottleneck</span>


This functional paradigm is a prime example of co-designing an algorithm for its hardware. However, the GPU is not always the faster choice. Performance studies show a clear "crossover point": for small tasks that require fewer than roughly 10,000 numbers, a modern CPU is often faster due to the overhead of launching a GPU kernel (Askar et al., 2021).
For larger tasks where the GPU's throughput dominates, a second, more surprising bottleneck emerges: data transfer. If the numbers are generated on the GPU but needed by the CPU, the transfer itself can cripple performance. One analysis found that copying the results back to the CPU took, on average, seven times longer than the computation itself (Gregg & Hazelwood, 2011). This highlights a fundamental rule: the highest performance is only achieved when random numbers are both generated and consumed on the same device.

While a simple hash is great for visual effects, this same "stateless" principle is the foundation of high-performance libraries. They use complex, cryptographically-inspired functions to generate statistically robust and uncorrelated random streams for demanding scientific simulations, an approach detailed in the landmark paper on parallel random numbers by Salmon et al..


Askar, T., Shukirgaliyev, B., Lukac, M., & Abdikamalov, E. (2021). Evaluation of Pseudo-Random Number Generation on GPU Cards. Computation, 9(12), 142.

Salmon, J.K., Moraes, M.A., Dror, R.O., & Shaw, D.E. (2011). Parallel random numbers: As easy as 1, 2, 3. 2011 International Conference for High Performance Computing, Networking, Storage and Analysis (SC), 1-12.

C. Gregg and K. Hazelwood, "Where is the data? Why you cannot debate CPU vs. GPU performance without the answer," (IEEE ISPASS) IEEE International Symposium on Performance Analysis of Systems and Software, Austin, TX, USA, 2011, pp. 134-144, doi: 10.1109/ISPASS.2011.5762730. keywords: {Graphics processing unit;Kernel;Benchmark testing;Bandwidth;Databases;Performance evaluation;Histograms},

