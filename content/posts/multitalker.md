+++
title = "Overlapping speakers with Parakeet TDT"
date = "2026-10-31"
updated = "2026-11-15"

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
it is English only and made for streaming: it hears small pieces of audio at a time, which is
great for live captions. Most of what I transcribe is recordings, and there an offline model that
hears the whole sentence is usually more accurate. So I wanted the same thing on
[parakeet-tdt-0.6b-v3](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3), my favorite offline
model, which also knows 25 European languages.

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

## <span style="color:orange;"> Update: on Parakeet Ultra </span>

Moondream released [Parakeet Ultra](https://huggingface.co/moondream/parakeet-ultra), a post
trained TDT v3 with lower error rates than the original. So I trained the speaker parts again on
top of it, with the same data recipe plus AMI's headset recordings for cleaner meeting labels.

While testing it on [FLEURS](https://huggingface.co/datasets/google/fleurs) in all 25 languages I
found something I had missed: the speaker parts are not completely harmless. Even with one person
talking alone, they change the encoder a little. In English you can't see it, but in languages
where the base model is already weaker (Latvian, Maltese, Greek, Slovenian, Lithuanian) the error
went up by 2 to 4 points. Perfect speaker masks gave the same drop, so it was not the diarization.
I tried one more training round with extra data in those languages. It helped a bit there and hurt
the overlap results, so I dropped it.

I added a simple fix.  exported model now has a switch. When only one person speaks in a chunk, the
speaker parts are turned off and the model is exactly plain Ultra, the same output word for word.
When two or more people speak, they are on. I'm thinking of using this in production along with a ifelse and sortformer. If there's a single speaker, then turn it off... That's it :-P

With that, FLEURS mixtures in all 25 languages went from 11.9% to 10.9%, the same as plain Ultra
(10.8%). The overlap results did not change:

| | Plain Ultra | Multitalker Ultra |
|---|---|---|
| AMI, 16 full test meetings | 23.2% | **15.6%** |
| MLS mixtures, 1 to 3 speakers | 34.5% | **12.5%** |
| LibriSpeechMix, 2 speakers | 27.0% | **4.4%** |
| LibriSpeechMix, 3 speakers | 38.0% | **8.6%** |

The AMI number is per whole meeting now, not one minute windows. For comparison, plain Ultra
without any speakers gets 19.8% there, so the multitalker is better even before you count who said
what.

I also exported the TDT v3 version again with the same switch, so both models behave the same way.

## <span style="color:orange;"> In Rust </span>

The encoder is exported to ONNX with three extra inputs: the speaker's activity, everyone else's,
and the switch. parakeet-rs runs the diarization, cuts the audio into chunks of about 20 s at
pauses, and runs TDT once per speaker in each chunk, or once as plain TDT when only one person
speaks there. A 106 s recording of two people takes 13.6 s on my M3 Air, CPU only.

```rust
use parakeet_rs::MultitalkerTDT;

let mut model = MultitalkerTDT::from_pretrained(
    "multitalker-parakeet-ultra", // or "multitalker-tdt-0.6b-v3"
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

Single speaker parts now come out as plain TDT or Ultra, but with overlap the quality still
follows the base model: languages it is weaker in are weaker here too.

- GitHub: [altunenes/parakeet-rs](https://github.com/altunenes/parakeet-rs)
- Models: [multitalker-parakeet-ultra](https://huggingface.co/altunenes/parakeet-rs/tree/main/multitalker-parakeet-ultra),
  [multitalker-tdt-0.6b-v3](https://huggingface.co/altunenes/parakeet-rs/tree/main/multitalker-tdt-0.6b-v3)
