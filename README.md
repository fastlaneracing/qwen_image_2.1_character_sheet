# Muse Character Sheet — Qwen Image 2.1

[Qwen Image 2.1](https://huggingface.co/Comfy-Org/Qwen-Image-2.1) is a unified image generation and editing model. **Muse Character Sheet (Qwen Image 2.1)** is a single ComfyUI node that generates a full 5-pose character turnaround (portrait, front, left profile, right profile, back) from **one character reference photo**. No mannequin guide sheet is required: the poses are described directly in text.

The node runs all five generations internally and gives each pose its own **Confirm** / **New seed** / **Apply edit** buttons in its own panel, plus an **Edit all poses** box that applies one instruction to every unconfirmed pose at once.
- **Confirm** is a local lock only; it doesn't submit anything.
- **New seed** on a pose with an active edit re-rolls *that edit*, not the original.
- **Reset edit** and **Reset prompt to default** take a pose back to where it started.
- Once every pose is confirmed, **Build final sheet now** assembles the final sheet (4096x2304 by default, with smaller presets).

You can optionally connect a 5-panel mannequin **pose_reference_image**. It's cropped to the matching panel for each pose and passed as image 2, with a prompt line saying it's a pose guide only.

This is a port of [Muse Character Sheet — Klein](https://github.com/muse-collective-26/muse-character-sheet-klein) (FLUX.2 [klein]), itself a sibling of [Muse Character Sheet](https://github.com/muse-collective-26/muse-character-sheet) (Krea 2). The UI and session logic are the same; only the model backend differs.

## Install

```
cd ComfyUI/custom_nodes
git clone https://github.com/fastlaneracing/qwen_image_2.1_character_sheet
```

Restart ComfyUI and **hard-refresh the browser** (Ctrl+Shift+R). Until you do, the frontend doesn't know the new node and a queued workflow fails with `Node 'ID #9' has no class_type`. Then load `workflows/Muse Character Sheet Qwen.json`.

### Required

- **ComfyUI with Qwen Image 2.1 support in core** (`TextEncodeQwenImage21`, `QwenImage21Cache`). Update ComfyUI if those nodes are missing.
- **[ComfyUI-RMBG](https://github.com/1038lab/ComfyUI-RMBG)**, for the white-background cleanup on every pose. RMBG-2.0 downloads itself on first use.
- **[ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack)** + **[Impact-Subpack](https://github.com/ltdrdata/ComfyUI-Impact-Subpack)**, only if you turn on **Face Detail** (off by default).

The pack bundles **Muse Sheet: Align Figure Height** (`MuseSheetAlignFigure`), which aligns the panels of the final sheet. If the Klein or Krea 2 pack is also installed, ComfyUI logs a harmless duplicate-registration warning, because the file is identical.

## Models — [Comfy-Org/Qwen-Image-2.1](https://huggingface.co/Comfy-Org/Qwen-Image-2.1)

| widget | file | folder |
|---|---|---|
| `unet_name` | `qwen_image_2.1_bf16.safetensors` (or `qwen_image_2.1_int8_convrot`) | `models/diffusion_models/` |
| `clip_name` | `qwen3vl_8b_bf16.safetensors` (or `qwen3vl_8b_int8_convrot`) | `models/text_encoders/` |
| `vae_name` | `qwen_image_2.1_vae_bf16.safetensors` | `models/vae/` |

The `model_override` / `clip_override` / `vae_override` sockets take already-loaded models instead. They must be Qwen Image 2.1 models, with the CLIP loaded as type `qwen_image`.

## Settings

| widget | default | notes |
|---|---|---|
| `steps` / `cfg` | 40 / 4.0 | euler / simple. Qwen Image 2.1 isn't step-distilled; the official path uses about 40–50 steps. |
| `negative_prompt` | common anatomy/framing faults | Applied to every pose and every edit. Ignored at cfg 1. |
| `ref_resolution` | 1024 | Reference images are resized to about this² pixels before the text encoder and VAE see them. |
| `cache_device` / `cache_dtype` | auto / default | Drive `QwenImage21Cache`. Try `int8` if you're short on VRAM. |
| `seed_mode` | random | `random`: re-running after a finished sheet gives a fresh set. `fixed`: always starts from seeds 41001–41005. |
| `output_size` | 4096x2304 | Final sheet size. Panel proportions are identical across presets. |

## How it differs from the Klein version

| | Klein | Qwen Image 2.1 |
|---|---|---|
| conditioning | `CLIPTextEncode` → `ReferenceLatent`; negative = `ConditioningZeroOut` | `TextEncodeQwenImage21`: the Qwen3-VL encoder sees each reference, which is also spliced in as a latent. A real negative prompt. |
| latent | `EmptyFlux2LatentImage` (128 ch) | zero 64-ch latent at /16. Edits sample on the encoder's own latent (sized to the source), so the untouched parts stay in place. |
| sampling | 4 steps, cfg 1 | 40 steps, cfg 4 |
| pose guide | silent extra reference, never named in the prompt | image 2, named in the prompt as a pose guide only |
| VAE output | RGB | RGBA, composited over white before RMBG |

Speed: on a test machine, all 5 poses took about 165 s at the defaults (bf16 weights).
