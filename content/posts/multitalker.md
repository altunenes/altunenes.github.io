+++
title = "Overlapping speakers with Parakeet TDT"
date = "2026-10-31"

[taxonomies]
tags=["Rust","audio","ASR","ML"]
+++

When two people talk at the same time, speech recognition models write one sentence out of both,
or one of them just disappears.

In [parakeet-rs](https://github.com/altunenes/parakeet-rs) the usual way to get speakers is
diarization. [Nemotron-3 Diarization](https://huggingface.co/nvidia/Nemotron-3-Diarization) says
who speaks when, and every word goes to whoever was speaking at its timestamp. This works fine
until people talk over each other. Then a sentence gets cut between two speakers and the quieter
person is lost.

NVIDIA has a model for exactly this, the
[multitalker Parakeet](https://huggingface.co/nvidia/multitalker-parakeet-streaming-0.6b-v1), but
it is English only. I wanted the same thing on
[parakeet-tdt-0.6b-v3](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3), which knows 25
European languages.

## <span style="color:orange;"> The idea </span>

I used NVIDIA's design. The diarization output for one speaker (is this person talking in this
80 ms frame or not) goes into the encoder through a few extra layers, called speaker kernels. Then
the model runs once per speaker, and each time it writes down only that person, even when someone
else is talking at the same time.

I froze the whole TDT model and trained only the speaker kernels plus small adapters in its 24
encoder layers. That is 7.4M parameters out of 634M. The original model never changes, so one
person talking alone should come out as good as before.

## <span style="color:orange;"> Training </span>

Everything trained on a free Colab T4, around 30k steps per round.

The first rounds used [FLEURS](https://huggingface.co/datasets/google/fleurs) clips from all 25
languages, mixed on top of each other with 1 to 4 speakers. The model learned to separate voices,
but on real meetings it was worse than plain TDT (31.4% vs 24.7% on AMI). It caught the first
sentence of a speaker and dropped the later ones. Which makes sense: in a mixed clip every person
says one sentence. In a real conversation people speak again and again.

So in round 3 I added real conversations: dialogues built from
[VoxPopuli](https://huggingface.co/datasets/facebook/voxpopuli) speeches, and the AMI training
meetings. As labels I used TDT's own transcripts wherever they agreed with the human ones, so the
model keeps TDT's writing style.

That fixed it. On the AMI test meetings round 3 got 15.5%, plain TDT 26.5%.

## <span style="color:orange;"> The numbers </span>

All scores are cpWER: word error rate where each real speaker is matched with the best output
speaker, lower is better. Both systems get the same Nemotron-3 Diarization output. "Plain TDT" is
what parakeet-rs users run today, words given to speakers by their timestamps.

English, [LibriSpeechMix](https://github.com/NaoyukiKanda/LibriSpeechMix), the standard test set
for overlapping speech. NVIDIA reports their multitalker Parakeet with the same diarization model
in the [Nemotron 3.5 Transcribe report](https://huggingface.co/blog/nvidia/nemotron-transcribe-granary):

| Speakers | Plain TDT | NVIDIA multitalker (reported) | Multitalker TDT |
|---|---|---|---|
| 1 | 1.9% | 2.77% | 2.0% |
| 2 | 36.7% | 7.20% | **4.4%** |
| 3 | 47.3% | 12.94% | **7.8%** |

note that this comparison is not perfectly fair. NVIDIA's model is streaming, "mine" is offline and sees 20 s
at a time. Also, I ran 300 of the 2,620 mixtures per set, because I'm "proudly" GPU poor :-P the full
set kept running into free Colab's limits, so this one ran on Colab's CPU instead.

Six other languages, 900 mixtures from the
[Multilingual LibriSpeech](https://huggingface.co/datasets/facebook/multilingual_librispeech) test
set (not used in training), 1 to 3 different speakers of the same language talking over each other:

| | Plain TDT | Multitalker TDT |
|---|---|---|
| 1 speaker | 6.8% | **5.7%** |
| 2 speakers | 39.0% | **10.7%** |
| 3 speakers | 53.7% | **16.7%** |

Per language, with 2 or 3 speakers:

| | Plain TDT | Multitalker TDT |
|---|---|---|
| German | 44.8% | **12.5%** |
| French | 59.5% | **12.9%** |
| Spanish | 48.0% | **9.0%** |
| Italian | 43.6% | **17.4%** |
| Polish | 47.1% | **14.3%** |
| Portuguese | 45.3% | **19.7%** |

Real meetings, [AMI](https://groups.inf.ed.ac.uk/ami/corpus/) test set, 32 windows of one minute:

| | Plain TDT | Multitalker TDT |
|---|---|---|
| all | 26.5% | **15.5%** |
| overlap < 10% | 15.3% | **9.6%** |
| overlap ≥ 25% | 42.9% | **25.1%** |

## <span style="color:orange;"> In Rust </span>

The encoder is exported to ONNX with two extra inputs: the speaker's activity and everyone else's.
parakeet-rs runs the diarization, cuts the audio into chunks of about 20 s at pauses, and runs TDT
once per speaker in each chunk. A 106 s recording of two people takes 13.6 s on my M3 Air, CPU only.

```rust
use parakeet_rs::MultitalkerTDT;

let mut model = MultitalkerTDT::from_pretrained(
    "multitalker-tdt-0.6b-v3",
    "nemotron3_diar_v3.onnx",
    Some(config),
)?;
for u in model.transcribe(&audio, 16000, 1)? {
    println!("[{:.2} - {:.2}] speaker_{}: {}", u.start, u.end, u.speaker_id, u.text);
}
```

## <span style="color:orange;"> Where it falls short </span>

Two or four people at once is fine, eight at once comes out as fragments. It is also only as good
as the diarization: if Nemotron misses a speaker, so does this. And since it runs once per speaker,
it gets slower than plain TDT when many people talk.

It is offline. It needs the whole chunk, so for live captions NVIDIA's streaming model is the
right one.

AMI left a small accent too. Some English output now writes numbers as words ("eighty two") and
uses British spelling like "diarisation".

I measured 7 of the 25 languages. The others should follow the base TDT quality, but I haven't
tested them. 

- GitHub: [altunenes/parakeet-rs](https://github.com/altunenes/parakeet-rs)
- Model: [multitalker-tdt-0.6b-v3](https://huggingface.co/altunenes/parakeet-rs/tree/main/multitalker-tdt-0.6b-v3)
