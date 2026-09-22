<p align="center">
  <img src="docs/assets/readme-banner.svg" alt="LynnReal-Omni — Standard four-step and Flash three-step multimodal generation" width="100%">
</p>

<p align="center">
  <strong>Video generation, references, motion controls and editing.</strong>
</p>

<p align="center">
  <a href="https://arxiv.org/abs/2609.15863">Paper<sup>🔥HOT</sup></a>  &nbsp;·&nbsp;
  <a href="#introduction">Introduction</a> &nbsp;·&nbsp;
  <a href="#overview">Overview</a> &nbsp;·&nbsp;
  <a href="#models">Models</a> &nbsp;·&nbsp;
  <a href="#quick-start">Quick start</a> &nbsp;·&nbsp;
  <a href="#comfyui">ComfyUI</a> &nbsp;·&nbsp;
  <a href="#tasks">Tasks</a> &nbsp;·&nbsp;
  <a href="#usage-guides">Usage guides</a> &nbsp;·&nbsp;
  <a href="#roadmap">Roadmap</a> &nbsp;·&nbsp;
  <a href="CHANGELOG.md">Changelog</a> &nbsp;·&nbsp;
  <a href="#organization">Organization</a> &nbsp;·&nbsp;
  <a href="#acknowledgments">Acknowledgments</a> &nbsp;·&nbsp;
  <a href="#acknowledgments">Citation</a>
</p>

> [!IMPORTANT]
> 🔥 **ComfyUI supports Standard four-step and Flash three-step generation.** Workflows, required
> nodes and model layout are under [`comfyui/`](comfyui/). For limited VRAM, use the matching
> Standard Lite or Flash Lite workflow and checkpoint at its shipped step count; see
> [Recommended models for limited VRAM](#recommended-models-for-limited-vram).
>
> This repository is an early beta. Bugs, compatibility issues, unfinished features and
> inconsistent generation quality may remain.

## Recommended models for limited VRAM

**For limited VRAM, we strongly recommend Standard Lite to preserve Standard quality, or
Flash Lite for a much smaller memory footprint and very fast generation.** Start with the
matching Lite checkpoint and `*_lite.json` ComfyUI workflow when choosing a memory-saving setup.

- **Standard Lite · quality first:** BF16 Lite is **37.6 GiB**; INT8 Lite is **20.4 GiB**.
  At the shipped **4 steps**, each preserves its corresponding original checkpoint's tested
  outputs; the BF16/INT8 switch remains available.
- **Flash Lite · memory and speed first:** the INT8 DiT is **16.7 GiB**, paired with the Light
  VAE and shipped **3-step** workflows. It retains Flash's fast generation and tested quality;
  Lite reduces the checkpoint footprint rather than promising extra speed over full Flash.

These are DiT file sizes, not total runtime VRAM requirements. See the
[ComfyUI Lite setup and checkpoints](comfyui/README.md#recommended-models-for-limited-vram).

## Paper

> [LynnReal-Omni: Native multi-modal Video Generation for Agentic Visual Workflows](https://arxiv.org/abs/2609.15863)  
> Technical report, September 2026.

## Demo
https://github.com/user-attachments/assets/25b49015-cf5a-4eb3-b7f9-165c3176db3e

*Preview compressed to ~10 MB for this repository; for the full-quality demo, click the YouTube
or Bilibili link below.*

| YouTube | Bilibili |
| :---: | :---: |
| [Watch on YouTube ↗](https://www.youtube.com/watch?v=P5Bl2mriEmk) | [Watch on Bilibili ↗](https://www.bilibili.com/video/BV12vYB6BEjc/) |

## Introduction

**One model, unified across many video tasks.** Built on a 32B shared multimodal
diffusion transformer (following the MiniMax H3 architecture), LynnReal-Omni
brings text-to-video, image-to-video, human- and hand-pose guided generation,
structural control, omni-reference generation, style transfer, video editing,
degraded-video restoration (we find that it also repairs videos affected by
accumulated error) and streaming long-video generation into a single framework,
all at four-step fast generation. It also accepts heterogeneous inputs such as
appearance references, editable 3D renders and game recordings, so an Agent can
compose visual conditions inside one model.

**Flash, built for real-time rendering.** We train a 27B LynnReal-Omni-Flash
(three-step generation) that lowers inference cost through model and decoding
acceleration and a lightweight VAE decoder. On a single H100, warm generation
and decoding of a 22-frame 540p video takes 843 ms with the Standard model and
377 ms with Flash, laying the groundwork for real-time streaming video
generation.

**Data pipeline and MSAVP evaluation.** We build a systematic multi-shot and
omni-reference data pipeline covering video cleaning, subject association,
multimodal annotation and alignment control. From the large corpus of collected
videos we select a high-quality multi-shot audio-visual subset and extract a
variety of omni-reference condition controls. We also propose MSAVP for
evaluating multi-shot audio-visual generation.

## Overview

| Generate | Control | Edit & repair |
| :--- | :--- | :--- |
| Text-to-video and first-frame conditioning | Single- and multiple-subject references | Instruction-guided image editing |
| Video continuation | Body- and hand-pose control | Video appearance editing |
| Speech and video generation | Game- and mesh-video rendering | General frame-by-frame video repair |

**Standard · 4 steps** &nbsp; / &nbsp; **Flash · 3 steps** &nbsp; / &nbsp; **Optional lightweight VAE**

Standard uses four denoiser forwards. Flash uses three forwards with 42 blocks,
spatial token selection, INT8 projection weights and INT8 activations. Step counts
refer to each generation call: Standard first-pass generation uses **4 steps**;
independent frame repair uses **4 steps per source frame**. Long streaming can add
**2 refinement steps per later chunk**; see [streaming generation](#streaming-generation).

## Models

The repositories below are the planned Hugging Face upload locations. Weights
will become downloadable as their uploads are completed.

| Model | Sampling | Weights | Status |
| :--- | :--- | :--- | :--- |
| **Standard** · BF16 DiT | 4 steps | [Hugging Face ↗](https://huggingface.co/stdstu123/LynnReal-Onmi-beta-0.1) | Completed |
| **Flash** · W8A8 DiT | 3 steps | [Hugging Face ↗](https://huggingface.co/stdstu123/LynnReal-Onmi-flash-beta-0.1) | Completed |
| **Lightweight VAE** | Optional codec | [Hugging Face ↗](https://huggingface.co/stdstu123/LynnReal-Onmi-light-vae) | Completed |
| **Standard DiT INT8** | 4 steps | To be announced | **Coming soon** |

Place each downloaded bundle under `weight/`, preserving its configurations and
component subdirectories:

```text
weight/
├── standard/           # Standard bundle: DiT, text encoder, codecs and configs
│   └── transformer/    # The single physical Standard DiT checkpoint
├── flash/              # Flash bundle
├── vae/                # Official VAE
└── light-vae/          # Optional lightweight VAE
```

Pass the complete `weight/standard/` bundle to `--weights`. Its DiT lives in
`weight/standard/transformer/`; the other required components are loaded from the
bundle too. The lightweight VAE does not replace the DiT or text encoder.

> **Standard DiT INT8 is coming soon.** Experimental Standard INT8 launchers are
> already in the codebase; the downloadable checkpoint has not been released.
> Flash W8A8 is a separate model variant.

## Quick start

Run the following commands from the repository root, with the required local
weights in place.

### 1. Install

From Conda **base**, install this checkout and activate the new environment:

```bash
pip install .
conda activate lynnreal
```

The bootstrap creates or reuses a Python 3.12 environment named `lynnreal` and
checks attention on the visible GPU. Use `pip install . -v` for detailed progress.
See [Runtime notes](#runtime-notes) for other environments, offline installation
and hardware-specific behavior.

### 2. Generate a video

```bash
# Standard: five seconds, native 768p, four denoiser forwards
bash script/sample/standard/bf16/t2v.sh --frame 5s --resolution 768p

# Flash: three denoiser forwards
bash script/sample/flash/int8/t2v.sh
```

### 3. Try visual inputs

```bash
# First-frame conditioning
bash script/sample/standard/bf16/ti2v.sh --image test/assets/times_square.png

# Continue a video using the launcher's example input
bash script/sample/standard/bf16/v2v.sh

# Subject references: replace these paths with your own images
bash script/sample/standard/bf16/ref2v.sh --ref_image first.jpg,second.jpg
```

Each launcher saves the video, prompt, full log and measured DiT/decoder times
under `output/`. `--frame 120f` requests exactly 120 frames; `--frame 5s` is
five seconds at 24 fps. The default output is 1344 × 768. Standard BF16 and Flash
W8A8 have separate entry points. The experimental `script/sample/standard/int8/`
launchers remain available for development; the downloadable Standard DiT INT8
checkpoint is **coming soon**.

For a repeatable run, use `bash script/sample/standard/bf16/t2v.sh --seed 7 --frame 5s --name my_case`.
Choose a new output name for each run.
Keep prompt, seed, geometry, references and attention backend fixed for comparisons.
Changing attention precision can change a four-step result. Standard BF16 defaults
to the native attention backend and official VAE.

## ComfyUI 🧩

**We also ship a ComfyUI port — it is open, and you are very welcome to try it! 🎉**

[`comfyui/`](comfyui/) contains workflows for **t2v, i2v, r2v, pose2v and v2v** on the Standard
four-step checkpoints **and for t2v, ti2v and ref2v on the Flash three-step checkpoint**, the
small custom node pack they need (`ComfyUI-LynnReal`), the demo assets the workflows load, and
the exact model files each task needs. Those files are published in ComfyUI format alongside the
release weights on Hugging Face under
[🤗 stdstu123/LynnReal-Onmi-beta-0.1 · comfyui/models](https://huggingface.co/stdstu123/LynnReal-Onmi-beta-0.1/tree/main/comfyui/models).
Drop the workflows, the node pack and the models into your ComfyUI install and they run — no
launcher flags needed. Every task, its workflow and the weights it loads are listed in
[`comfyui/README.md`](comfyui/README.md).

### ⚡🔥 Flash three-step is live — and it is fast!

The Flash checkpoints run in ComfyUI at their trained **three** steps: `t2v`, `ti2v` (one first
frame) and `ref2v` (reference pictures), W8A8 DiT plus the Light VAE! On a **single H100 80 GB**
at 1344×768, warm — model already loaded, the way a session runs — three measured runs per cell:

| Task | 5 s · generate | 5 s · click-to-video | 10 s · generate | 10 s · click-to-video |
| :--- | ---: | ---: | ---: | ---: |
| Text → video | **8.4 s** | 12.2 s | **22.0 s** | 29.1 s |
| First frame → video | **8.9 s** | 13.0 s | **23.1 s** | 30.1 s |
| References → video | **9.5 s** | 13.2 s | **24.2 s** | 31.1 s |

A five-second 1344×768 clip **with native stereo audio** — three denoiser steps and both
decoders included — in about **eight and a half seconds**, on **one** GPU. *Generate* is the
release's own Generate-wall convention (first denoiser forward to decoded frames);
*click-to-video* is what you actually wait for, prompt encoding and muxing included, and repeat
runs agree to ±0.02 s!

> [!WARNING]
> Videos longer than 11 seconds are not usable in the ComfyUI Flash path yet — the accelerated
> path for long clips is still being fixed.

Nothing is assumed about your card: the node pack picks the fastest attention it can find and
verifies it numerically (FlashAttention-3 → FA2 → cuDNN SDPA → native), falls back to ComfyUI's
own block math whenever a fused kernel is unavailable, and tunes the INT8 GEMMs for the GPU it
actually runs on!

> [!WARNING]
> **The ComfyUI port is experimental and under active construction 🚧**
>
> - It runs the same checkpoints and the same schedules, but the pipeline around them is
>   ComfyUI's, so **performance numbers should be taken from the original scripts** in
>   [`script/sample/`](script/sample/) — those are the reference implementation and the ones
>   quoted in the paper. The port is measurably slower than the scripts today (fewer fused
>   kernels, a different attention backend and no Hopper-specific INT8 grouping yet).
> - Same-seed output is **not** comparable between the two engines: the noise source and the
>   decoder path differ. Compare quality, not pixel identity.
> - The port covers the Standard four-step checkpoints (with an optional INT8 switch on the
>   canvas, off by default except `pose2v`, backed by `lynnreal_omni_standard_int8.safetensors`)
>   and the **Flash three-step checkpoint**. Videos longer than 11 seconds are not usable in the
>   ComfyUI Flash path yet — the accelerated path for long clips is still being fixed.
> - We will keep improving it — speed, memory, more tasks and cleaner packaging are all on the
>   list. **Issues and pull requests are very welcome!** 🙌

## Tasks

`script/sample.py` accepts repeated `--reference` paths. Image and video prompt
labels count separately from one. `--aligned-reference` indexes the complete
input list from zero: an appearance image followed by a pose video uses index 1.

| Task | Mode | Inputs |
|---|---|---|
| Text to video | `t2v` | prompt |
| First-frame conditioning | `ti2v --native-keyframes` | first image |
| Single or multiple subjects | `reference` | reference images |
| Body/hand motion | `reference` | appearance image and aligned pose video |
| Video editing | `video-edit` | aligned source video, optional style image |
| Image editing | `image-edit` | source image; output filename ends in `.png` |

Image editing saves both a selected image and its complete companion clip.
See [the prompt-writing guide](skill/lynnreal-prompt/SKILL.md) for task-specific
instructions and reference conventions.

### Frame-by-frame video repair

`frame_repair.sh` is a **general repair entry point**. Supply your own video and
repair instruction for the scene and artifacts you want to address.

```bash
bash script/sample/standard/bf16/frame_repair.sh \
  --video /path/to/input.mp4 \
  --prompt-file /path/to/repair_prompt.txt \
  --output output/my_frame_repair
```

- **Input:** 24 fps; width and height must be divisible by 32.
- **Process:** edit each source frame independently, then assemble a silent video at 24 fps.
- **Budget:** four DiT forwards per source frame; 120 frames require **480 forwards**.
- **Preview:** add `--indices 0,24,48 --keep-clips` to inspect a few frames before a full run.
- **Limitation:** independent edits may introduce brightness or shape fluctuations across frames.

<details>
<summary><strong>Understand frame selection</strong></summary>

The default image-edit call generates an internal 22-frame clip and selects
index 11 (zero-based). Use `--image-frame` to choose another valid index.
One selected image is retained for each original source frame; the internal clip
does not extend the source timeline. No temporal interpolation or generated-frame
feedback is used.

A corrected still alone does not establish that the whole video is repaired.
Inspect the complete assembled sequence for remaining artifacts and flicker.

</details>

### Streaming generation

[`script/sample/standard/bf16/stream.sh`](script/sample/standard/bf16/stream.sh)
generates an image-conditioned first chunk and continues its latent history.
Run these commands from the repository root:

```bash
# Default: 5 seconds, 768p, 24 fps, seed 7; four DiT forwards per chunk.
bash script/sample/standard/bf16/stream.sh --frame 5s

# 30 seconds: preserve the initial four-step prefix, then use 4+2-step refinement.
bash script/sample/standard/bf16/stream.sh --frame 30s

# 30 seconds with four steps throughout; disable refinement and context refresh.
bash script/sample/standard/bf16/stream.sh --frame 30s --no-refresh-context

# Generate only the native 22-frame first chunk.
bash script/sample/standard/bf16/stream.sh --first-chunk-only
```

**The default five-second run uses 4 steps throughout.** Above 136 requested
frames, the default enables context refresh: the bootstrap and first seven
continuation chunks use **4 Standard BF16 DiT forwards each**; continuation
chunk eight onward uses **4 + 2 forwards**, including the second pass.
`--no-refresh-context` keeps the fixed-context, four-step-only policy at any duration.
These modes use the same DiT in `weight/standard/transformer/` and the official VAE.

| Configuration | Output frames | Total DiT forwards |
| :--- | ---: | ---: |
| First chunk only | 22 | 4 |
| Default 5 seconds | 120 | 32 |
| Default 30 seconds, with later refinement | 720 | 242 |
| 30 seconds with `--no-refresh-context` | 720 | 172 |

The totals include the bootstrap and every continuation chunk. They describe
sampling work, rather than elapsed time or text-encoder/decoder computation.
Long clips refresh image-aware text from the preceding chunk after the stable
prefix. Local texture and shape drift can still occur.

<details>
<summary><strong>Custom images, prompts and continuation captions</strong></summary>

```bash
bash script/sample/standard/bf16/stream.sh \
  --image inputs/first.png \
  --prompt-file inputs/initial.txt \
  --captions inputs/continuations.json \
  --frame 5s --seed 7 \
  --output output/my_stream
```

- `--image`: a **1344 × 768** first image; this entry currently supports only 768p.
- `--prompt-file`: the initial scene and action prompt. Use `--prompt "..."` for literal text instead.
- `--captions`: a JSON array of nonempty strings describing successive motion intervals. Provide at least `ceil((frames - 17) / 17)` captions: **7 for 5 seconds**, **42 for 30 seconds**.
- `--frame`: `5`, `5s` and `120f` all request 120 frames at 24 fps.
- `--output`: a new output directory; existing runs are preserved.

A custom image or initial prompt requires a matching continuation-caption file,
except with `--first-chunk-only`. Without custom inputs, the launcher selects the
bundled rainy-street image and five- or thirty-second caption plan.
Use `--dry-run` to validate inputs and inspect the planned commands and step counts
before loading the models. Set `LYNNREAL_PYTHON` if the runtime uses another interpreter.

</details>

Each run saves `video.mp4`, `sample.log` and `run.json` under
`output/standard/bf16/stream/<timestamp>/`, or the directory supplied with
`--output`. The `first_chunk/` and `continuation/` subdirectories retain their
prompts, latents and per-chunk timing records. `run.json` records the planned
step budget; per-chunk metadata records actual DiT forwards.

Long-stream context refresh stages the text encoder through CPU memory on a
single GPU, which increases end-to-end latency. An optional second-GPU
conditioning worker can be selected with `--conditioner-service`.
See [streaming instructions](test/streaming.md) for worker startup, native overlap,
appearance constraints and `--audit-codec` verification.
For continuation from an existing **video**, use
`script/sample/standard/bf16/v2v.sh --continuation`.


## Usage guides

| What to explore | Documentation |
| :--- | :--- |
| Sampling launchers and arguments | [Sampling guide](script/sample/README.md) |
| Timing and benchmark commands | [Speed-test guide](script/sample/speed_test/README.md) |
| Streaming generation and validation | [Streaming reproduction](test/streaming.md) |
| Prompt structure and reference conventions | [Prompt-writing guide](skill/lynnreal-prompt/SKILL.md) |

## Runtime notes

<details>
<summary><strong>Installation options · Conda, existing environments and offline wheels</strong></summary>

The local build hook creates/reuses the `lynnreal` Conda environment (Python 3.12),
installs the inference package and dependencies there, then probes attention on the
visible GPU. New environments use conda-forge without changing global channel settings.
Base receives only the small `lynnreal-bootstrap` receipt package.
Failures propagate to pip; installation does not activate the parent shell.
Use `pip install . -v` to see build/probe progress. The bootstrap uses PyPI for its
child installs; it does not change global pip settings. Weights remain in `weight/`.
For an existing complete local wheel cache, set `LYNNREAL_WHEELHOUSE=/path/to/wheels`
to install the same requirements offline inside the target environment.

Outside base, `pip install .` installs into the active Python 3.12/3.13 environment.
To explicitly install into the current environment, including base:

```bash
python script/setup_env.py --current-env
```

The standalone helper also creates the environment and shows progress directly:

```bash
python script/setup_env.py --pypi-only
conda activate lynnreal
```

It bootstraps checksum-verified Miniforge if Conda is absent. Existing incompatible
Python environments are not downgraded or deleted; choose a new `--env-name`.
`--no-shell-init` leaves shell startup files untouched. `--skip-attention` installs
only core dependencies. For ordinary wheel builds from base, set
`LYNNREAL_INSTALL_CURRENT=1` to disable the local Conda bootstrap.

</details>

<details>
<summary><strong>Attention and GPU compatibility · probes, memory and tested devices</strong></summary>

Attention is tested in a fresh process, in this order: FA3 (Hopper with CUDA >=12.3),
FA2 (Ampere or newer), cuDNN SDPA, PyTorch Flash SDPA, then native SDPA. FA3 is pinned to commit
`203b9b3dba39d5d08dffb49c09aa622984dff07d` and built from source when installation
is needed. RTX 4090 cannot run Hopper FA3 and normally selects FA2. Legacy FA1 is
not installed over FA2 because it does not satisfy this H3 BF16 interface.

```bash
python script/setup_env.py --attention-only
# Strict FA3 acceptance on a supported Hopper GPU:
python script/setup_env.py --attention-only --attention _flash_3
```

Auto mode prominently warns when FA3 is not active and records the selected backend
and numerical probes in `output/setup/attention.json`. An explicitly requested
backend must pass; otherwise installation exits nonzero. No working GPU means
attention remains unverified: rerun the probe on the sampling worker. Attention
backends can produce different four-step outputs; probe success alone does not
establish identical video quality. Sampling launchers retain explicit
`--attention-backend` selection for controlled comparisons. On GPUs below 64 GiB,
Flash launchers decode one tile at a time to bound temporary memory; spatial tile
geometry and blend order stay unchanged. This does not make the Standard model
fit into the same memory budget.

Blackwell GPUs select the same pinned PyTorch package versions with CUDA 12.8
wheels when the installed build is older. This follows [PyTorch's Blackwell
support requirements](https://pytorch.org/blog/pytorch-2-7/). H100 uses the pinned
FA3 commit; its source build retains the dense FP16/BF16 forward paths used by H3
(head dimensions up to 128), omitting unused training/KV-cache kernels. FA3 is
not selected on RTX 40/50-series, A100 or B200 merely because it is installed:
other devices try FA2, cuDNN SDPA, PyTorch Flash SDPA, then native SDPA, with a numerical
probe and a visible non-FA3 warning. An explicitly requested backend remains a
strict requirement.

Actual GPU validation for this revision covers H100 80GB and the available RTX
4090 with 48GB memory. A100, RTX 50-series and B200 use compatibility selection
but have not been measured here. Kernel compatibility does not remove model
memory requirements; a standard 24GB RTX 4090 is not equivalent to the tested
48GB node. See `output/setup/review/index.html` for generated samples and
`output/setup` for installation/probe records.

On an uncached prompt, the text encoder retains the 50 layers actually consumed
by H3 before moving to CUDA. When memory is limited, complete text layers are
staged through CPU without changing their precision or execution order. The
cached conditioning features remain compatible. This prevents the unused
14 text layers from causing a cold-prompt allocation failure on the tested
48GB RTX 4090; text encoding remains separate from DiT latency.

</details>

<details>
<summary><strong>Kernel precompilation and persistent caches</strong></summary>

Installation also precompiles representative inference kernels on the visible
GPU and writes `output/setup/kernels.json`. When local weights and GPU memory
permit, it also prepares both models at 540p/22 frames and 768p/5, 10, 15 seconds.
On the tested 48GB RTX 4090, full-model preparation covers Flash at 540p/22 frames.
Completed profiles are recorded under `output/setup/profiles-*.json`; repeated
installation reuses matching profiles. Profile signatures follow the local inference
imports, model/decoder configurations, and software versions; unrelated streaming
experiments do not invalidate them. Other shapes are prepared on demand.
Set `LYNNREAL_PRECOMPILE=0` to skip full-model preparation. The installer and INT8 launchers share
persistent caches under `output/.cache`; GPU architecture, kernel source and
software versions distinguish cached selections. Unsupported optional compilation
emits a warning and sampling uses the compatible path; a numerical mismatch
fails installation instead of being accepted as a successful fallback.

Hopper INT8 inference additionally uses persistent TMA GEMM for measured long
projection shapes and a single-warp Q/K kernel that retains the reference reduction
order. Short GEMMs and non-Hopper devices keep their existing dispatch. Set
`LYNNREAL_LONG_KERNELS=0` for the paired reference configuration. These changes
preserve exact latents, audio and RGB in the completed paired comparisons;
this statement does not cover changing attention, quantization or decoder settings.

Precompilation does not remove model loading or fresh-process tracing. The public
INT8 commands list preparation calls separately; their warm DiT+decoder times
are not shell-command elapsed times. Dense attention remains the main cost for
768p long clips, so short-clip speed does not extrapolate linearly with duration.
See `output/speed_test/long_acceleration_review/index.html` for measured pairs and videos.

</details>

<details>
<summary><strong>Codecs and timing · official VAE, lightweight VAE and measurement scope</strong></summary>

The core Python API and BF16 shell launchers retain the official VAE default.
INT8 shell launchers select the compiled lightweight decoder and adaptive tiles.
In the Python CLI, `--light-vae weight/light-vae` selects the 26-block distilled
decoder with the unchanged encoder and latent interface.
Both use indexed Safetensors and component configs. For reconstruction comparisons,
use the same encoded latent and report precision, tile geometry and hardware.

Timing records separate actual denoiser forwards and video decoder execution.
BF16 refers to the DiT; the official video decoder uses its native FP16 autocast.
Cold loading, conditioning, audio decoding and output encoding are outside this
sum; end-to-end generation is recorded separately. CPU offload can substantially
increase latency and must be disclosed in speed comparisons.

</details>

## Repository layout

```
model/   inference, conditioning, codecs and kernels
script/  sampling and video continuation
skill/   prompt-writing guidance
weight/  standard/, flash/, vae/, light-vae/
test/    default prompts, reference media and continuation plans
output/  generated media, complete logs and timing records
tool/    helpers required by streaming and structure editing
```

All model components are local to `weight/`, including the text encoder,
tokenizer, image processor, audio codec and schedulers. The DiT bundles contain
complete inference parameters. No additional adapter is required. Weight paths
outside this directory are rejected. Model licenses and upstream attribution
are retained in each bundle.

**Standard uses one DiT checkpoint in `weight/standard/transformer/`.**
Standard clip launchers validate this directory and run four denoiser forwards
per generated clip. Streaming uses four per chunk; when context refresh is enabled,
continuation chunk eight onward adds two refinement forwards. The internal reference-task
component name `transformer_ref` maps to the same `transformer/` directory through
`modular_model_index.json`; no `transformer_ref/` weight directory is needed.
The sampler loads only the DiT component selected by the task layout. See
the model card included with the downloaded Standard weight bundle for details.

## Roadmap

**This repository will continue to receive updates. The current beta is still
imperfect, and we appreciate your patience with its remaining defects.**

Our planned releases and improvements include:

- [ ] The Standard DiT INT8 checkpoint.
- [ ] Improved model checkpoints with better generation quality and consistency.
- [ ] Training code to support further experimentation and model development.
- [ ] Selected datasets or dataset subsets, subject to their redistribution permissions.
- [ ] Continued fixes to installation, hardware compatibility, inference, documentation,
  and reproducible examples.
- [ ] ComfyUI: keep the Light VAE decoder compiled on DynamicVRAM machines — it currently falls
  back to eager decoding there, which roughly doubles decode time.

These are development plans, not a fixed release schedule. Availability and
usage instructions will be updated here as each release is ready. Reproducible
bug reports and feedback on failure cases are welcome.

## Organization

This project is developed by **[Lynnreal Lab](https://github.com/LynnReal-AI)**.

**Leader:** Xiaofeng Mao, Shaohao Rui, Weijie Ma

**Core Contributors:** Xiaofeng Mao, Peijia Lin, Shaohao Rui, Yibo Zhang, Haibin Wan, Weijie Ma

### Join us

Welcome students with backgrounds in 3D reconstruction and interactive world
models to apply for internships and collaborate! Please send your resume to
[hr@lynnreal.com](mailto:hr@lynnreal.com).

## Acknowledgments

We especially thank the **[MiniMax H3 team](https://github.com/MiniMax-AI/MiniMax-H3)**
for the model that forms the foundation of LynnReal-Omni.

We also thank [Qwen3-VL](https://github.com/QwenLM/Qwen3-VL) for the multimodal
encoder, tokenizer and processor implementation used for conditioning.

### H3 community references

The following community projects provided useful implementation references and
technical discussions during our development and experiments:

| Project | Reference and contribution |
| :--- | :--- |
| [ComfyUI-JZL-MiniMax-H3](https://github.com/wjluoxiao/ComfyUI-JZL-MiniMax-H3) | Ref2VA reference encoding, multimodal input organization and reference-scale handling. |
| [ComfyUI-Minimax-H3-Prompt-Builder](https://github.com/Tasrovy/ComfyUI-Minimax-H3-Prompt-Builder) | Structured Ref2VA prompts, multi-segment continuity and second-pass sampling workflows. |
| [DiffSynth-Studio](https://github.com/modelscope/DiffSynth-Studio) | H3 reference/keyframe conditioning and training-loss implementations consulted during development. |
| [MiniMax-H3-FineTuning](https://github.com/IAmIronMan42/MiniMax-H3-FineTuning) | Community H3 fine-tuning guidance and numerical-correctness discussions. |
| [ComfyUI-MiniMax-H3-Turbo](https://github.com/Larryvrh/ComfyUI-MiniMax-H3-Turbo) | Community sampling implementations reviewed for audio/video time schedules and prediction conventions. |

### Second-pass sampling and latent upscaling

We thank the authors of these projects for sharing H3 refinement implementations
and workflows that informed our second-pass sampling experiments:

| Project | Reference and contribution |
| :--- | :--- |
| [h3-latent-upscaler](https://github.com/rockerBOO/h3-latent-upscaler) | Spatial latent upscaling between low-resolution and high-resolution sampling passes; see our [sampling guide](script/sample/README.md). |
| [Comfyui_Minimax_h3_latent_Upscaler](https://github.com/LBH-123-AI/Comfyui_Minimax_h3_latent_Upscaler) | Learned H3 latent upscaling and split-upscale workflows examined in our refinement experiments. |

These acknowledgments include development references and experimental comparisons;
the released sampling paths and their validated settings are documented separately
in this repository. We appreciate the authors' open-source contributions and the
community's reproducible bug reports and feedback.

Please see [LICENSE](LICENSE), [NOTICE](NOTICE), and the respective upstream
projects for their license terms and attribution notices.

### Citation
If you use Yume for your research, please cite our paper:

```bibtex
@misc{mao2026lynnrealomninativemultimodalvideo,
      title={LynnReal-Omni: Native multi-modal Video Generation for Agentic Visual Workflows}, 
      author={Xiaofeng Mao and Peijia Lin and Shaohao Rui and Yibo Zhang and Haibin Wan and Weijie Ma},
      year={2026},
      eprint={2609.15863},
      archivePrefix={arXiv},
      primaryClass={cs.CV},
      url={https://arxiv.org/abs/2609.15863}, 
}
```
---
<p align="center">
  <strong>Thank you for trying LynnReal-Omni.</strong><br>
  Better models, training code and selected datasets are planned.<br>
  <a href="#overview">Back to overview ↑</a>
</p>
