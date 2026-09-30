# Muse Character Sheet — Qwen Image 2.1

A Qwen Image 2.1 port of **Muse Character Sheet (Klein)** (`muse-character-sheet-klein`). It's one ComfyUI node, `MuseCharacterSheetQwen`, that generates a 5-pose character turnaround (portrait, front, left profile, right profile, back) from **one character reference photo**. The UI is the same as Klein's: **Confirm** / **New seed** / **Apply edit** / **Reset** per pose, **Edit all poses**, and **Build final sheet now** (4096x2304 by default).

Local pack, not published. The workflow lives in the Muse H3 Director repo at `workflows/Muse Character Sheet Qwen.json`.

## What differs from Klein

| | Klein | Qwen Image 2.1 |
|---|---|---|
| models | FLUX.2 klein 9B-KV + `qwen_3_8b` (clip type `flux2`) + `flux2-vae` | `qwen_image_2.1` + `qwen3vl_8b` (clip type `qwen_image`) + `qwen_image_2.1_vae` |
| conditioning | `CLIPTextEncode` → `ReferenceLatent`; negative = `ConditioningZeroOut` | `TextEncodeQwenImage21`: the VL encoder sees each reference, which is also spliced in as a latent. Real negative prompt. |
| latent | `EmptyFlux2LatentImage` (128 ch) | zero 64-ch latent at /16; edits use the encoder's own latent (sized to the source) |
| sampling | 4 steps, cfg 1 | 40 steps, cfg 4, euler/simple |
| cache | `FluxKVCache` (9B-KV only, `kv_cache` widget) | optional `QwenImage21Cache` (`cache_device`, `cache_dtype`); the model's own default is auto |
| pose guide | silent extra ReferenceLatent, never named in the prompt | passed as image 2, with `POSE_REF_SUFFIX` naming it a pose guide only |
| portrait canvas | 544x976 | 544x960 (multiple of 32) |

These are unchanged from Klein: session state and actions, face detail (Impact SEGS, largest face only), RMBG-2.0 white background, `MuseSheetAlignFigure` (bundled; registering it twice alongside Klein is harmless), output presets and seed modes.

## Requires

- ComfyUI with Qwen Image 2.1 support in core (`TextEncodeQwenImage21`, `QwenImage21Cache`).
- [ComfyUI-RMBG](https://github.com/1038lab/ComfyUI-RMBG).
- [ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack) + Subpack, only if Face Detail is on (off by default).

## Models — [Comfy-Org/Qwen-Image-2.1](https://huggingface.co/Comfy-Org/Qwen-Image-2.1)

- `diffusion_models/qwen_image_2.1_bf16.safetensors` (or `qwen_image_2.1_int8_convrot`)
- `text_encoders/qwen3vl_8b_bf16.safetensors` (or `qwen3vl_8b_int8_convrot`)
- `vae/qwen_image_2.1_vae_bf16.safetensors`

`model_override` / `clip_override` / `vae_override` must be Qwen Image 2.1 models, with the CLIP loaded as type `qwen_image`.
