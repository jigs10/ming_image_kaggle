# Ming Image 0.1 Design on Kaggle (Wan2GP)

Kaggle notebook that runs **Ming-Image-0.1-Design** in [Wan2GP](https://github.com/deepbeepmeep/Wan2GP) on a single free **T4** GPU.

Ming Image Design is a text-to-image and image-editing model built for design work: UI screens, posters, infographics and other layouts with readable text. This notebook uses the INT8 ConvRot weights repackaged for Wan2GP by DeepBeepMeep, so it fits on a T4.

- Weights: [`DeepBeepMeep/MingImage`](https://huggingface.co/DeepBeepMeep/MingImage)
- Original model: [`inclusionAI/Ming-Image-0.1-Design`](https://huggingface.co/inclusionAI/Ming-Image-0.1-Design)

## What you need

- A Kaggle account (phone verified, so GPU and internet are available)
- In the notebook settings: **Accelerator = GPU T4**, **Internet = On**

## How to use

1. Open `ming_image_kaggle.ipynb` in Kaggle and set the accelerator and internet as above.
2. Run the cells top to bottom:
   1. **Install Wan2GP**: clones (or updates) the repo.
   2. **Install requirements**.
   3. **Download heavy models**: transformer (~7 GB) and text encoder (~17 GB) go to `/tmp`, so your 20 GB working folder stays free.
   4. **Patch FlashAttention default**: required on T4 (see below).
   5. **Launch Wan2GP**: starts the UI and prints a public Gradio link.
3. Open the link. In the model dropdown pick **Ming Image 0.1 → Design**, choose the **INT8** variants, then click **Create Finetune** and point it at the files below.
4. Generate.

### File paths

Main checkpoint:
```
/tmp/ming_image/Ming-Image-0.1-Design_int8_convrot.safetensors
```

Text encoder checkpoint:
```
/tmp/ming_image/BailingMM2-Ming-Image/BailingMM2-Ming-Image-Core_int8_convrot.safetensors
```

The VAE, vision tower and tokenizer files are small, so Wan2GP downloads them itself.

## Settings

| Setting | Value |
|---|---|
| Resolution | `1024x1024` |
| Steps | `12` |
| Guidance (CFG) | `1` |
| Number of images | `1` |
| Reference image | optional (one, for editing) |
| Acceleration LoRA | none |

The upstream default resolution is 2048x2048, which needs far more memory. Stay at 1024x1024 on a T4.

## Test prompt

```
A clean mobile app UI screen for a coffee shop called "Bean There", header with the app name, three menu cards with prices, bottom navigation bar, warm brown and cream colors, flat modern design, crisp readable text.
```

## Why the FlashAttention patch?

Ming's vision tower defaults to FlashAttention 2, and a T4 can't run it. Without the patch you get:

```
FlashAttention2 has been toggled on, but it cannot be used ... the package flash_attn seems to be not installed.
```

The patch cell switches the defaults to `sdpa` (vision tower) and `eager` (language model). **Re-run it after every `git pull`**, because an update can overwrite the edit.

## Troubleshooting

- **FlashAttention error on launch**: run the patch cell, then relaunch.
- **`Unable to locate file 'ming_image/transformer_config.json'`**: Wan2GP is missing its small config files in `Wan2GP/ckpts`. Select the model again, or let Wan2GP download them.
- **Out of memory or the kernel dies while loading**: the text encoder is ~17 GB even in INT8, and Kaggle has limited system RAM. Restart and try a lower-memory profile in Wan2GP's configuration tab.
- **No space left on device**: keep the big files in `/tmp` as the notebook does. `/kaggle/working` is limited to 20 GB.
- **Red `TypeError: 'NoneType' object is not subscriptable` tracebacks in the console**: Gradio gallery refresh glitches. They are harmless and don't stop generation.

## Not covered yet

**Ming-Image-0.1-Design-Layer**, which splits a flat image into transparent layers, is not part of this notebook yet.

## License

Ming-Image-0.1 is MIT licensed. Wan2GP and the repackaged weights have their own licenses; check them before redistributing.
