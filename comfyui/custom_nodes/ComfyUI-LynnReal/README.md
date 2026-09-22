# ComfyUI-LynnReal

Everything the LynnReal release needs from ComfyUI that is not core: the Flash token
compression, the Light VAE loader, the INT8 backend helper and the aligned-reference node
for the pose/hand workflows. **No file under `comfy/` is patched.**

## Model choice for limited VRAM

**显存受限时，强烈推荐 Standard Lite（保质量）和 Flash Lite（极其省显存、极速）。**
For limited VRAM, strongly prefer **Standard Lite** for Standard quality or **Flash Lite**
for a much smaller memory footprint and very fast generation. Use the matching `*_lite.json`
workflow and checkpoint; keep **4 steps for Standard Lite** and **3 steps for Flash Lite**.
Lite preserves its corresponding original checkpoint's tested outputs at these step counts.
See the [ComfyUI model selection guide](../../README.md#recommended-models-for-limited-vram)
for checkpoint names and sizes.

## Install

```bash
cd ComfyUI/custom_nodes
git clone <this repo> ComfyUI-LynnReal     # or copy the folder in
```

Restart ComfyUI. No launcher flag is required: `LynnRealInt8Backend` enables the
comfy-kitchen Triton backend when the CUDA backend is unavailable (torch < cu130), which is
what makes INT8 checkpoints fast -- about 3x versus the eager fallback
(`--disable-triton-backend` or `LYNNREAL_NO_TRITON=1` opts out).

## Automatic sampling safety (`sampling_safety.py`, `runtime.py`)

* **Default: no extra reserve.** With ComfyUI's own setting the 61.7 GiB DiT and the 15 GiB text
  encoder stay resident and a warm 4-step 1344x768 t2v takes ~50 s. Reserving VRAM makes
  ComfyUI evict the text encoder between runs, which cost ~38 s per run in our measurements.
* **DynamicVRAM follows ComfyUI’s detected capability.** The pack keeps an already active
  DynamicVRAM configuration (including the tested cu128 and cu130 environments). On a cu13x
  torch >= 2.8, it also attempts activation when ComfyUI has not enabled it, and logs the
  outcome (`LYNNREAL_NO_DYNAMIC_VRAM=1` skips this additional activation attempt).
* **Encoder memory includes expanded vision tokens.** Before loading the H3 text/vision
  encoder, the pack counts the actual Qwen image patches and expanded sequence length.
  This lets ComfyUI evict resident weights before encoding large references or pose frames,
  including when switching from a fully loaded BF16 DiT. Plain short prompts keep the
  ordinary encoder budget.
* **Reference and pose memory is budgeted per request.** The core H3 estimate accounts for
  the generated latent but omits reference and text rows. The node pack counts those actual
  conditioning rows before model loading and supplies extra working memory to ComfyUI's loader.
  No global reserve is changed, so a subsequent ordinary text-only request uses its normal
  budget. Explicit `--reserve-vram` still adds the requested process-wide reserve.
* **Large INT8 calls are split before entering the kernel.** Input, output and accumulator
  sizes are checked against a conservative byte-offset limit. Oversized calls run as independent
  row slices on the selected backend; small calls are unchanged. This also covers Standard
  workflows, which do not pass through the Flash block wrapper. Triton can stay enabled.
* **Reference `max` matches official sizing.** Images are resized to a 2048-pixel short edge,
  including upscaling, with both dimensions aligned to 32. Larger references need more time
  and working memory; the sampler budgets that automatically.

Start all shipped workflows with the same command:

```bash
python main.py
```

Or use `comfyui/tools/launch_comfyui.sh` from the ComfyUI directory (or set `COMFYUI_DIR`).
On QiZhi it uses the existing shared `lynnreal-comfyui` environment; elsewhere it uses `python`,
overridable with `COMFYUI_PYTHON`. Neither a workflow-specific reserve nor a Triton-disable
flag is required. The budget is an estimate, not a guarantee for arbitrarily long videos,
unbounded reference inputs, or a GPU already occupied by another process.

Safety regression checks (pass the installed ComfyUI directory):

```bash
# GPU: compare chunked/un-chunked INT8 output, including torch.ops dispatch.
python custom_nodes/ComfyUI-LynnReal/tools/verify_sampling_safety.py "$PWD"
# On a compatible cu130 build, also check native CUDA kernels.
python custom_nodes/ComfyUI-LynnReal/tools/verify_sampling_safety.py "$PWD" --backend cuda
# CPU: compare estimated Qwen token expansion with the real image preprocessor.
python custom_nodes/ComfyUI-LynnReal/tools/verify_encoder_budget.py "$PWD"
```

## Nodes

| Node | What it is for |
|---|---|
| `LynnReal Flash token compression (MiniMax H3)` | The Flash DiT's trained token compression. Insert between the Flash model loader and the guider (`start_block=2`, `end_block=28`, `stride=2`). |
| `Load LynnReal H3 VAE (Light VAE aware)` | Loads any MiniMax-H3 video VAE with the decoder depth taken from the checkpoint. Use it for `lynnreal_omni_light_vae_fp16.safetensors`; the official VAE loads identically to the stock loader. |
| `LynnReal INT8 backend info` | Reports which comfy-kitchen backend serves the quantized ops. |
| `LynnReal Aligned Reference` | Frame-aligned reference setup for the pose/hand control workflows. |

### Why a VAE loader instead of the stock one

The release's Light VAE is a 26-block distilled decoder; the official H3 video VAE has 36.
ComfyUI builds the H3 VAE with the depth hardcoded to 36 and loads state dicts with
`strict=False`, so the stock `VAELoader` accepts the Light VAE with nothing but a
`Missing VAE keys [...]` warning and decodes with ten randomly initialized blocks -- nothing
but garbage frames. This loader reads the depth from the checkpoint, applies the Light VAE's own tile geometry
(`272/16`, from its `decode_config.json`) and compiles the decoder the way the release's
`--compile-vae` does. Set `LYNNREAL_NO_COMPILE_VAE=1` to decode eagerly.

## Requirements

* ComfyUI with MiniMax-H3 support (`comfy/ldm/minimax/`, `ResolutionSelector`,
  `ComfyMathExpression`, `ComfySwitchNode`) and comfy-kitchen INT8
  (`int8_tensorwise`). Tested against ComfyUI 0.35.0.
* An 80 GB-class GPU for the tested 1344x768 / 124-frame workflows.
* `flash-attn` (FA2) for the fast attention path, `triton` for the INT8 GEMM.
