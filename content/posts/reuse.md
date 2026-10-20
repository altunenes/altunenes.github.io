+++
title = "I forced a speech model onto the GPU for months. The CPU was faster."
date = "2026-10-20"

[taxonomies]
tags=["Rust","audio","GStreamer","ML"]
+++

Back in March I wanted to clean noisy speech live, from a microphone, in Rust. NVIDIA has
[Real-time RE-USE](https://huggingface.co/nvidia/Real-time_RE-USE): a streaming Mamba
model that takes one 20 ms frame of audio at a time. Small, causal, made for exactly this.

I was sure it had to run on the GPU. Neural network, real time, GPU. Because that's just how it goes,
right? So I exported it to ONNX and ran it through [`ort`](https://github.com/pykeio/ort).

This post is how that went, and why the thing I ended up shipping,
[reuse-rs](https://github.com/altunenes/re-use-rs), is plain Rust on the CPU.

Before the story, here's what it does: the same recording, clean original, noisy, and cleaned by
reuse-rs (from [VoiceBank-DEMAND](https://huggingface.co/datasets/JacobLinCool/VoiceBank-DEMAND-16k)).
The spectrogram fills in as the audio plays.

<div align="center">

| Original (clean) | Noisy | reuse-rs, 8 layers |
|:-:|:-:|:-:|
| <video width="100%" controls playsinline><source src="/videos/reuse/vb_original.mp4" type="video/mp4"></video> | <video width="100%" controls playsinline><source src="/videos/reuse/vb_noisy.mp4" type="video/mp4"></video> | <video width="100%" controls playsinline><source src="/videos/reuse/vb_8layers.mp4" type="video/mp4"></video> |

</div>

## <span style="color:orange;"> The ONNX road </span>

Mamba's core is a selective scan: a small state per channel that gets decayed and updated every
step. ONNX has no op for that. The closest thing is `com.microsoft.LinearAttention`, a contrib
op in ONNX Runtime, so I mapped the scans onto it.

It worked. Sort of. I ran into problems inside ONNX Runtime on the way and reported what I found
([onnxruntime#27796](https://github.com/microsoft/onnxruntime/issues/27796),
[#27996](https://github.com/microsoft/onnxruntime/pull/27996)).

Then the numbers, per 20 ms frame on my M3:

- **WebGPU:** 16.9 ms with 4 layers, 20.6 ms with 8. That's at the edge: anything over 20 ms
  means you fall behind the microphone.
- **CPU (ONNX Runtime):** about 70 ms.
- **CoreML:** the graph got split, fell back to CPU, 168–198 ms.

I also tried NVIDIA's bigger offline model (30 bidirectional layers, their best quality). 0.19–0.33×
real time on WebGPU, 3.3–3.9 GB of memory. At one point I ran several backends in the same
process to compare them and watched it eat 26 GB of RAM. Lesson learned.

## <span style="color:orange;"> What if I just... don't </span>

I never considered writing the model myself. It felt like the "serious" path was the GPU and
the runtime. But the more I looked at it, the less sense it made: every step is **one** frame.
321 frequency bins. You can't batch the future, because a live mic hasn't produced it yet, and
the model needs the state from the previous frame anyway.

So for each tiny frame, the GPU has to launch kernels, run a bit of math, and sync back. My
suspicion was that the launching and syncing was most of the bill.

The WebGPU numbers already said so, I just hadn't read them that way: going from 4 to 8 layers
only added 3.7 ms. Double the work, almost the same time. Most of those ~17 ms were fixed
per-frame overhead, not the model.

So I wrote it in plain Rust. The convolutions are im2col plus one matrix multiply (Accelerate on
macOS, the [`matrixmultiply`](https://crates.io/crates/matrixmultiply) crate elsewhere). The scan
is the interesting part. RE-USE uses a state size of 16 everywhere, so the state is a fixed
`[f32; 16]` and the update is three tiny loops the compiler can vectorize:

```rust
fn scan16(s: &mut [f32; 16], a: &[f32; 16], delta: f32, dx: f32,
          b: &[f32; 16], c: &[f32; 16]) -> f32 {
    let mut decay = [0f32; 16];
    for n in 0..16 {
        decay[n] = fast_exp(delta * a[n]);
    }
    let mut out = [0f32; 16];
    for n in 0..16 {
        s[n] = decay[n] * s[n] + dx * b[n];
        out[n] = s[n] * c[n];
    }
    let mut acc = [0f32; 4];
    for n in 0..16 {
        acc[n % 4] += out[n];
    }
    (acc[0] + acc[1]) + (acc[2] + acc[3])
}
```

`fast_exp` is a polynomial `exp` (about 1 ulp) written so it vectorizes too, because `f32::exp`
in that loop stops the compiler. The scans for different channels are independent, so they run
in parallel with `rayon`.

## <span style="color:orange;"> Is it actually right? </span>

A denoiser that "sounds fine" can still be wrong in ways you don't hear. So I compared every
output against a **float64** PyTorch reference.

| | Difference from the float64 reference |
|---|---|
| reuse-rs | **1.5e-6** |
| ONNX export through `ort` | 2.4e-3 |

1.5e-6 is about as close as PyTorch's own float32 run gets.

## <span style="color:orange;"> The numbers </span>

Same M3 (8 cores, 16 GB), idle, time per 20 ms frame:

| Layers | reuse-rs (CPU) | ONNX Runtime (WebGPU) |
|---|---|---|
| 4 | **8.4 ms** | 16.9 ms |
| 6 | **11.5 ms** | 17.4 ms |
| 8 | **14.7 ms** | 20.6 ms |

Memory went from about 420 MB to about 150 MB. And the CPU numbers grow with the work (+6.3 ms
from 4 to 8 layers), which is what you want: the time goes into the model, not around it.

Without Accelerate (the `matrixmultiply` path) 8 layers takes 21.4 ms on the same machine, so
on other platforms 4–6 layers is the safer pick. And the machine has to be reasonably free: when
I ran it next to a game, frame times went up.

## <span style="color:orange;"> Fewer layers? </span>

The model was trained so you can stop after any of 3 to 12 layers, each with its own decoders.
I couldn't hear a difference between 4 and 8, so I measured it on the same VoiceBank-DEMAND clip,
against the clean original (SI-SDR, higher is better):

| | SI-SDR |
|---|---|
| Noisy | 4.5 dB |
| 4 layers | 16.4 dB |
| 6 layers | 16.6 dB |
| 8 layers | 16.7 dB |

0.3 dB between 4 and 8, for 1.75× the speed. That's one clip, but it matches my ears.

<div align="center">

| 4 layers | 6 layers |
|:-:|:-:|
| <video width="100%" controls playsinline><source src="/videos/reuse/vb_4layers.mp4" type="video/mp4"></video> | <video width="100%" controls playsinline><source src="/videos/reuse/vb_6layers.mp4" type="video/mp4"></video> |

</div>

## <span style="color:orange;"> Using it </span>

```rust
use reuse_rs::Denoiser;

let mut denoiser = Denoiser::load("realtime_reuse/model.safetensors", 4)?;
// 16 kHz mono samples, in whatever chunks they arrive.
let clean = denoiser.process(&noisy_chunk)?;
// At the end of the stream:
let rest = denoiser.flush()?;
```

Output is delayed by exactly one frame (20 ms). Other sample rates (8 to 48 kHz) and look-ahead
(the model waits for up to 2 future frames, 20 ms each) are set with `Config`:

```rust
use reuse_rs::{Config, Denoiser};

let config = Config { layers: 4, sample_rate: 48000, look_ahead: 1 };
let mut denoiser = Denoiser::with_config("realtime_reuse/model.safetensors", config)?;
```

On the VoiceBank-DEMAND clip above, look-ahead gained only about 0.1 dB SI-SDR, so for a live mic
I leave it at 0.

I'm also adding it to GStreamer as a
`reusedenoise` element, so it can sit in any pipeline.

## <span style="color:orange;"> Where it falls short </span>

- When the background noise is **other people talking**, it struggles. Which makes sense: from
  the model's point of view, that noise is speech too. More examples are in the
  [README](https://github.com/altunenes/re-use-rs). If you don't need real time, NVIDIA's offline
  model does better, and [SpeechLens](https://github.com/faraday/SpeechLens) runs it on Apple
  Silicon.
- All speed numbers here are 16 kHz. Every frame at 48 kHz is about 3 times the work, so my rough
  estimate for 4 layers on the M3 is about 25 ms per frame: too slow for a live stream. For live
  audio, resampling to 16 kHz is the fast option.
- The code is MIT/Apache-2.0, but the weights are NVIDIA's, under a **non-commercial** license.

The thing I keep thinking about: the GPU wasn't wrong, it was just the wrong shape for the job.
For one stream of tiny frames, the CPU was sitting right there the whole time.

- GitHub: [altunenes/re-use-rs](https://github.com/altunenes/re-use-rs)
- crates.io: [reuse-rs](https://crates.io/crates/reuse-rs)
