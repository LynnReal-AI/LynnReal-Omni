# LynnReal-Omni · ComfyUI 🧩

ComfyUI workflows for the **Standard four-step** and **Flash three-step** checkpoints, plus the
small node pack they need and every file they load. The folder mirrors a ComfyUI install, so
copying three directories into place is the whole installation.

> [!NOTE]
> This port is **experimental and still being built out**. It runs the same checkpoints and the
> same schedules as `script/sample/`, but the pipeline around them is ComfyUI's: numbers and
> timings should be taken from the **original scripts**, and the same seed does **not** produce
> the same sample in both engines (different noise source and decoder path). We keep improving
> it — issues and pull requests are very welcome.

## Recommended models for limited VRAM

**For limited VRAM, strongly prefer Standard Lite for Standard quality, or Flash Lite for
a much smaller memory footprint and very fast generation.** Choose the matching Lite workflow
and checkpoint below when setting up a memory-constrained GPU.

| Priority | Recommended workflow | DiT checkpoint |
| --- | --- | --- |
| Preserve Standard quality | `*_4step_lite.json`, **4 steps** | `lynnreal_omni_standard_bf16_lite.safetensors` (**37.6 GiB**) or `lynnreal_omni_standard_int8_lite.safetensors` (**20.4 GiB**) via the INT8 switch |
| Minimize memory and generate quickly | `*_flash_3_step_lite.json`, **3 steps** | `lynnreal_omni_flash_int8_lite.safetensors` (**16.7 GiB**), with the Light VAE |

At the shipped step counts, Lite preserves its corresponding original checkpoint's tested
outputs. Flash Lite retains Flash speed; Lite itself primarily saves memory. The sizes above
are DiT files, not total runtime VRAM: the encoder, VAEs and intermediate tensors also need
memory. Install the current `ComfyUI-LynnReal` node pack alongside the Lite workflow.

## Layout

```
comfyui/
├── workflows/                       -> ComfyUI/user/default/workflows/
│   ├── t2v_lynnreal_4step.json          text → video + audio
│   ├── i2v_lynnreal_4step.json          first frame → video
│   ├── r2v_lynnreal_4step.json          reference images/videos → video
│   ├── pose2v_lynnreal_4step.json       pose control clip → video
│   ├── v2v_lynnreal_4step.json          video continuation
│   ├── t2v_lynnreal_flash_3_step.json   Flash: text → video + audio
│   ├── ti2v_lynnreal_flash_3_step.json  Flash: first frame → video + audio
│   ├── ref2v_lynnreal_flash_3_step.json Flash: reference pictures → video + audio
│   └── *_lite.json                      matching Standard / Flash Lite checkpoints
├── custom_nodes/ComfyUI-LynnReal/   -> ComfyUI/custom_nodes/
├── models/                          -> ComfyUI/models/            (see the tables below)
├── input/                           -> ComfyUI/input/             (demo assets the workflows load)
├── tools/quantize_h3_standard_int8.py   how the INT8 checkpoint was built
└── VERIFICATION.md                      Flash / Light-VAE end-to-end record
```

All model files are on Hugging Face:
[🤗 stdstu123/LynnReal-Onmi-beta-0.1 · comfyui/models](https://huggingface.co/stdstu123/LynnReal-Onmi-beta-0.1/tree/main/comfyui/models)

## What each task needs

Every task needs the current `ComfyUI-LynnReal` node pack for automatic memory accounting
and safe INT8 execution. The workflows differ in their checkpoints and extra inputs below.

| Task | Workflow | Diffusion model (`models/diffusion_models/`) | Text encoder (`models/text_encoders/`) | Video VAE (`models/vae/`) | Audio VAE (`models/vae/`) | Embedding (`models/embeddings/`) | Node pack | Extra input (`input/`) |
|---|---|---|---|---|---|---|---|---|
| Text → video | `t2v_lynnreal_4step.json` | `lynnreal_omni_standard_bf16.safetensors` (or `lynnreal_omni_standard_int8.safetensors` via the switch) | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | `minimax_h3_video_vae_fp16.safetensors` | `minimax_h3_audio_vae_fp32.safetensors` | `minimaxh3_art_is_explosion.safetensors` | `ComfyUI-LynnReal` | — |
| First frame → video | `i2v_lynnreal_4step.json` | same | same | same | same | same | `ComfyUI-LynnReal` | `transparent_rgb_gaming_mouse.png` |
| References → video | `r2v_lynnreal_4step.json` | same | same | same | same | same | `ComfyUI-LynnReal` | `red_superboy_on_city_roof.png`, `mecha_dragon_lightning.png` |
| Pose control | `pose2v_lynnreal_4step.json` | same (INT8 switch **on** by default) | same | same | same | same | `ComfyUI-LynnReal` | `pose_boxing_first.png`, `pose_boxing_control.mp4` |
| Video continuation | `v2v_lynnreal_4step.json` | same | same | same | same | same | `ComfyUI-LynnReal` | `snowboard.mp4` |
| **Flash** text → video | `t2v_lynnreal_flash_3_step.json` | `lynnreal_omni_flash_int8.safetensors` | same | `lynnreal_omni_light_vae_fp16.safetensors` | same | same | `ComfyUI-LynnReal` | — |
| **Flash** first frame → video | `ti2v_lynnreal_flash_3_step.json` | same | same | same | same | same | `ComfyUI-LynnReal` | `beauty_first_frame.png` |
| **Flash** references → video | `ref2v_lynnreal_flash_3_step.json` | same | same | same | same | same | `ComfyUI-LynnReal` | `beauty_reference_a.png`, `beauty_reference_b.png` |

The embedding is optional: the demo prompts reference it as
`embedding:minimaxh3_art_is_explosion`. Drop it and remove that token to run without it.

The five matching Standard Lite workflows are named `*_4step_lite.json`. They keep the same
inputs and the same `Use INT8 model?` switch, but select the BF16 Lite and INT8 Lite checkpoints.
They require the current `ComfyUI-LynnReal` node pack; the five original Standard workflows and
checkpoints remain unchanged.

### Files and sizes

| File | Size | Destination | Source |
|---|---|---|---|
| `lynnreal_omni_standard_bf16.safetensors` | 61.7 GiB | `models/diffusion_models/` | this release |
| `lynnreal_omni_standard_bf16_lite.safetensors` | **37.6 GiB** | `models/diffusion_models/` | this release (optional Standard Lite) |
| `lynnreal_omni_standard_int8.safetensors` | 44.5 GiB | `models/diffusion_models/` | this release (optional) |
| `lynnreal_omni_standard_int8_lite.safetensors` | **20.4 GiB** | `models/diffusion_models/` | this release (optional Standard Lite) |
| `lynnreal_omni_flash_int8.safetensors` | 37.0 GiB | `models/diffusion_models/` | this release (Flash workflows) |
| `lynnreal_omni_flash_int8_lite.safetensors` | **16.7 GiB** | `models/diffusion_models/` | this release (Flash workflows, optional) |
| `lynnreal_omni_light_vae_fp16.safetensors` | 3.6 GiB | `models/vae/` | this release (Flash workflows) |
| `minimax_h3_video_vae_fp16.safetensors` | 4.9 GiB | `models/vae/` | [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) |
| `minimax_h3_audio_vae_fp32.safetensors` | 577 MiB | `models/vae/` | [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) |
| `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | 14.6 GiB | `models/text_encoders/` | [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) |
| `minimaxh3_art_is_explosion.safetensors` | 500 KiB | `models/embeddings/` | [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) |

## ⚡ Standard Lite — 37.6 GiB BF16 / 20.4 GiB INT8

The Standard four-step model now has two optional Lite checkpoints:

- `lynnreal_omni_standard_bf16_lite.safetensors`: **37.6 GiB** instead of 61.7 GiB.
- `lynnreal_omni_standard_int8_lite.safetensors`: **20.4 GiB** instead of 44.5 GiB.

They replace the adaLN time-embedding MLP with an exact table for the timesteps visited by the
shipped Standard schedules. The table contains the original checkpoints' modulation vectors
verbatim, including the alternate one-row t2v call shape. Other timesteps use a compact curve
fallback.

**The shipped four-step results are bit-identical to the original checkpoints.** On one H100,
with the same seed (`970000`), all four sampler states and denoiser outputs matched exactly for
t2v, i2v, r2v, pose2v and v2v: `max |Δ| = 0`. This comparison used cold model loads and also
verified that `pose2v` selected the intended INT8 pair.

Open the matching `*_4step_lite.json` workflow after installing both the checkpoint you want and
the current node pack. The BF16/INT8 switch works exactly as in the original workflow.

> [!IMPORTANT]
> **Keep Standard Lite at the shipped 4 steps.** Other step counts can fall back to the curve
> columns, but they are not validated and are not covered by the bit-exact guarantee.

## ⚡🔥 Flash three-step is live!

The **Flash** checkpoints run in ComfyUI too: `t2v`, `ti2v` (one first frame) and `ref2v`
(reference pictures), each at its trained **three** steps with the W8A8 DiT and the Light VAE!

The earlier stage-level measurements below used a **single H100 80 GB**, 1344×768, warm,
with three measured runs per cell. Their reference preprocessing predates the current
2048-pixel `max` policy. For current full-workflow times across ordinary ComfyUI,
cu130 DynamicVRAM and FA2, see the [paired workflow validation](VERIFICATION_20260921.md).

| Task | 5 s · generate | 5 s · click-to-video | 10 s · generate | 10 s · click-to-video |
| :--- | ---: | ---: | ---: | ---: |
| Text → video | **8.4 s** | 12.2 s | **22.0 s** | 29.1 s |
| First frame → video | **8.9 s** | 13.0 s | **23.1 s** | 30.1 s |
| References → video | **9.5 s** | 13.2 s | **24.2 s** | 31.1 s |

A five-second 1344×768 clip **with native stereo audio** comes out in about **eight and a half
seconds**, three denoiser steps and both decoders included, on one card. *Generate* is the
release's own Generate-wall convention (first denoiser forward to decoded frames);
*click-to-video* is what you actually wait for, prompt encoding and muxing included. Repeat runs
agree to ±0.02 s!

> [!WARNING]
> Videos longer than 11 seconds are not usable yet — the accelerated path for long clips is
> still being fixed.

**It adapts to whatever card it lands on.** Nothing is assumed about your GPU: at startup the
pack picks the fastest attention it can find and verifies it numerically (FlashAttention-3 →
FlashAttention-2 → cuDNN SDPA → native SDPA), falls back to ComfyUI's own block math whenever a
fused kernel is unavailable, uses comfy-kitchen's CUDA backend when the torch build has it and
its Triton backend otherwise, and pins the INT8 GEMM config only on the Hopper part it was
measured on — everywhere else comfy-kitchen tunes for itself (`LYNNREAL_INT8_PIN=force`
overrides)! All changes live in the node pack; files under `comfy/` are not modified on disk.
The validation record identifies the ComfyUI and backend versions tested with this release.

### ⚡ Lite checkpoint — 16.7 GiB instead of 37.0 GiB

`lynnreal_omni_flash_int8_lite.safetensors` is the same three-step model with the adaLN step table
stored as the exact modulation vectors that its schedule visits — the original checkpoint's values,
verbatim — plus a curve fallback for anything else. Drop it in `models/diffusion_models/` and open
one of the `*_lite.json` workflows: no flags, no configuration, the node pack recognises the table
at load and prints one line.

**It produces the same frames as the original Flash.** Compared step by step at the same seed on an
H100 — t2v / ti2v / ref2v at 5 s and 10 s — the sampler state is identical (max |Δ| = 0), and the
speed is unchanged (warm 5 s t2v: DiT 6.14 s vs 6.15 s).

The exact table is pinned to the shipped schedule (`euler` + `simple`, three steps, stock shifts);
change the step count or the sampler and it falls back to the curve columns instead of failing.
**Keep Flash Lite at the shipped 3 steps; other step counts are not validated and may produce
different results.**

## The INT8 switch

All five Standard workflows carry a **`Use INT8 model?`** boolean (default **off** except
`pose2v`). In the original workflows it swaps `UNETLoader` to
`lynnreal_omni_standard_int8.safetensors`; in the matching Lite workflows it swaps to
`lynnreal_omni_standard_int8_lite.safetensors`. Both use the same W8A8 contract as the release's
`--precision int8` path (per-output-channel weight scales, per-token activation scales, INT32
accumulate; block 0, the last block, the token refiner, adaLN and the IO projections stay BF16).

The three Flash workflows do not need it: their checkpoint is already the trained W8A8 export.

## Install

1. ComfyUI recent enough to have MiniMax-H3 (`comfy/ldm/minimax/`), `ResolutionSelector`,
   `ComfyMathExpression`, `ComfySwitchNode` and comfy-kitchen INT8 (`int8_tensorwise`).
   Tested with 0.35.0.
2. Copy `custom_nodes/ComfyUI-LynnReal` into `ComfyUI/custom_nodes/`.
3. Copy `models/*` and `input/*` into the matching ComfyUI folders.
4. Start ComfyUI with `python main.py` and open a workflow. An 80 GB-class GPU is
   required for the tested 1344×768 / 124-frame workflows. The optional
   [launcher](tools/launch_comfyui.sh) uses the same defaults for every workflow.

   **VRAM:** the node pack keeps ComfyUI's own reserve. With the original checkpoint this keeps
   the 61.7 GiB DiT and the 15 GiB text encoder resident — a warm 4-step 1344×768 five-second
   t2v runs in ~50 s. Standard Lite reduces the loaded DiT footprint from 63.2 GB to 38.6 GB
   (BF16), or from 45.5 GB to 20.8 GB (INT8).
   Reference and pose requests automatically budget working memory before encoding and
   sampling, using their actual vision patches, reference and text token counts. The budget is
   request-local: switching back to an ordinary text prompt restores the usual budget. Large INT8 projections are row-chunked before they
   exceed safe kernel offsets, while smaller projections retain the fast path. Neither changing
   the reserve nor disabling Triton is required when switching workflows.

   `ref_image_size=max` follows the official 2048-pixel short edge, including upscaling,
   with both dimensions aligned to 32. These extra reference tokens can increase runtime;
   automatic memory accounting preserves the selected reference geometry.

   When ComfyUI enables DynamicVRAM, it manages the resident/offloaded split itself; the
   pack reports the actual active path at startup. The pack also attempts automatic
   activation on cu13x torch (≥ 2.8) if ComfyUI has not enabled it.

## Notes

* `minimax_h3_video_vae_fp16.safetensors` is loaded by ComfyUI's stock `VAELoader`.
* Same-seed output is not comparable across engines: the official scripts and ComfyUI draw
  their noise differently, and ComfyUI's decoder adds run-to-run spread at the same magnitude
  as ~44 dB PSNR (measured, see `VERIFICATION.md`).
