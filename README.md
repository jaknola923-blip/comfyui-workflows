# comfyui-workflows

My ComfyUI workflows for local AI video. Everything runs at home on an RTX 4060 Ti 16GB.

## H3_Viggle.json — vid2vid character transfer (MiniMax H3 + Viggle)

Takes a source motion video plus one reference character image, and re-renders the performer as that character while keeping the original motion, camera and timing.

My test case: a stage dance video, the performer replaced with an orange cat in a suit and sunglasses.

Workflow file: [H3_Viggle.json](H3_Viggle.json) — download it and drag it into ComfyUI.

### Models you need

| File | Goes in |
| --- | --- |
| `minimax_h3_hybrid_fl2va_ref2va_b25-49-int8.safetensors` | `models/unet` |
| `minimax_h3_video_vae_int8_convrot.safetensors` | `models/vae` |
| `fixed_embed_fwd_anyframe.safetensors` | loaded by ViggleTextCondLoader |
| `viggle_animate_dmd_lora.safetensors` (strength 1.0) | `models/loras` |
| `minimax_h3_ref2v_turbo_8step_v1.0_768p_bf16.safetensors` (strength 0.8) | `models/loras` |
| `MysticXXX_MMH3-V4.safetensors` (strength 0.8) | `models/loras` |

The int8 builds are deliberate: the whole thing has to fit in 16GB of VRAM.

### Custom nodes

If nodes show up red/missing, install via ComfyUI Manager:

- VideoHelperSuite (`VHS_LoadVideo`, `VHS_VideoCombine`)
- KJNodes (`ModelPreviewOverrideKJ`)
- The Viggle Animate node pack (`ViggleAnimateConditioning`, `ViggleTextCondLoader`)
- MiniMax H3 support nodes (`MiniMaxH3SigmaShift`)

### Settings that matter

- 24 fps in and out, source resized to about 0.8 megapixels
- Sampler `er_sde`, `beta` scheduler, 8 steps (turbo LoRA setup), sigma shift 3
- Inputs: `VHS_LoadVideo` takes the source motion clip, `LoadImage` takes one reference image of the target character

### Known issues — help wanted

This workflow is public because of these two problems. For anything longer than one short clip, I cut the source into ~5 second chunks, run each chunk through the workflow, and stitch the results. That gives me:

1. **A stutter at every seam.** The motion freezes for a few frames at each join, then jumps. I checked at frame level: the stitched file has perfect timestamps and no dropped or duplicated frames. Each chunk simply starts with a few near-still frames, like the model has to "settle" before it moves.
2. **The character changes at every seam.** Same cat, different face, different rendering, every 5 seconds. There is no shared identity anchor across chunks, so each chunk re-invents the character.

Already tried: deleting the frozen frames (the freeze just becomes a jump), and motion-interpolating the seams (better, still not clean).

If you know the right approach — overlap the chunks and trim the warm-up frames? Longer chunks? One shared reference pass over the stitched video at the end? — I would genuinely love to hear it. Issues and suggestions welcome.
