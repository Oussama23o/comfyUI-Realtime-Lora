# ComfyUI Realtime LoRA Trainer

Train LoRAs directly inside ComfyUI. Capture a face, a style, or a subject from your reference images and apply it to new generations - all within the same workflow. Useful for subject consistency across a project, style matching, or rapid iteration when you need a LoRA but don't want to context-switch into separate training tools. 


## What This Does

This node trains LoRAs on-the-fly from your images without leaving ComfyUI. SDXL training is particularly fast - a few minutes on a decent GPU. This makes it practical to train a quick LoRA from an image and immediately use it for img2img variations, style transfer, or subject consistency within the same workflow.

No config files to edit, no command line. Just connect images and go.

If this node saves you time or helps your workflow, consider [buying me a coffee](https://buymeacoffee.com/lorasandlenses) — supporters get early access to new features. Due to family circumstances I'm short on work right now, which is partly why I've had the time to build this. Donations genuinely help me keep developing this tool.

**Personal note:** I think SDXL is due for a revival. It trains fast, runs on reasonable hardware, and the results are solid. For quick iteration - testing a concept before committing to a longer train, locking down a subject for consistency, or even training on first and last frames for Wan video work - SDXL hits a sweet spot that newer models don't always match. Sometimes the "old" tool is still the right one.

## Supported Models

**Via Kohya sd-scripts:**
- SDXL (any checkpoint) - tested with Juggernaut XL Ragnarok, base SDXL will work too
- SD 1.5 (any checkpoint) - blazingly fast, ~2 mins for 500 steps on a 5090

**Via Musubi Tuner (Recommended for Z-Image):**
- Z-Image - faster training, smaller LoRA files, no diffusers dependency. Requires the de-distilled model for training, but trained LoRAs work with the regular distilled Z-Image Turbo model.

**Via AI-Toolkit:**
- Z-Image Turbo
- FLUX.1-dev
- Wan 2.2 (High/Low/Combo)

**Note on Wan 2.2 modes:** Wan uses a two-stage noise model - High handles early denoising steps, Low handles later steps. You can train separate LoRAs for each, or use Combo mode which trains a single LoRA across all noise steps that works with both High and Low models.

**Technical note:** When using High or Low mode, the example workflows still pass the LoRA to both models but at zero strength for the one you didn't train. This prevents ComfyUI from loading the base model into memory before training starts - a workaround to avoid unnecessary VRAM usage.

## Requirements

**Python version:** Both AI-Toolkit and sd-scripts work best with Python 3.10-3.12. Python 3.10 is the safest bet. Avoid 3.13 for now.

You need to install the training backend(s) separately:

**For SDXL / SD 1.5 training:**
1. Install sd-scripts: https://github.com/kohya-ss/sd-scripts
2. Follow their install instructions

**For Z-Image training (Musubi Tuner - Recommended):**
1. Install Musubi Tuner: https://github.com/kohya-ss/musubi-tuner
2. Follow their install instructions
3. Download the de-distilled Z-Image model for training:
   - Get it from: https://huggingface.co/ostris/Z-Image-De-Turbo/tree/main
   - Save to your ComfyUI `models/diffusion_models` folder
   - Note: This model is only needed for training. Your trained LoRAs will work with the regular distilled Z-Image Turbo model.

**For FLUX/Z-Image/Wan training (AI-Toolkit):**
1. Install AI-Toolkit: https://github.com/ostris/ai-toolkit
2. Follow their install instructions

**RTX 50-series GPUs (AI-Toolkit only):** Blackwell GPUs (RTX 5080/5090) require PyTorch 2.7+ with CUDA 12.8 support. The standard AI-Toolkit installation may not work out of the box. A community installer is available at https://github.com/omgitsgb/ostris-ai-toolkit-50gpu-installer that handles the correct PyTorch/CUDA versions. Note: sd-scripts for SDXL training is unaffected - this applies only to AI-Toolkit.

You don't need to open either environment after installation. The node just needs the path to where you installed them.

## Installation

Clone this repo into your ComfyUI custom_nodes folder:

```
cd ComfyUI/custom_nodes
git clone https://github.com/ShootTheSound/comfyUI-Realtime-Lora
```

Restart ComfyUI.

## Nodes

Search for these in ComfyUI:

- **Realtime LoRA Trainer** - Trains using AI-Toolkit (FLUX, Z-Image, Wan)
- **Realtime LoRA Trainer (Z-Image - Musubi Tuner)** - Trains Z-Image using Musubi Tuner (recommended)
- **Realtime LoRA Trainer (SDXL - sd-scripts)** - Trains using sd-scripts (SDXL)
- **Realtime LoRA Trainer (SD 1.5 - sd-scripts)** - Trains using sd-scripts (SD 1.5)
- **Apply Trained LoRA** - Applies the trained LoRA to your model

## Getting Started

There are example workflows included in the custom_nodes/comfyUI-Realtime-Lora folder. Open one in ComfyUI and:

1. Paste the path to your training backend installation (sd-scripts, Musubi Tuner, or AI-Toolkit)
2. For SDXL/SD1.5: select your checkpoint from the dropdown
3. For Musubi Tuner Z-Image: select your de-distilled model, VAE, and text encoder from the dropdowns
4. For AI-Toolkit models: the first run will download the model from HuggingFace automatically

**First run with AI-Toolkit:** The model will download to your HuggingFace cache folder. On Windows this is `C:\Users\%USERNAME%\.cache\huggingface\hub`. You can watch that folder to monitor download progress - these models are large (several GB).

## Basic Usage

1. Add the trainer node for your model type
2. Connect your training image(s)
3. Set the path to your training backend installation
4. Queue the workflow
5. Connect the lora_path output to the Apply Trained LoRA node

## Features

- Train from 1 to 100+ images
- Per-image captions (optional)
- Folder input for batch training with .txt caption files
- Automatic caching - identical inputs skip training and reuse the LoRA
- VRAM presets for different GPU sizes
- Settings are saved between sessions

## Defaults

- 500 training steps
- Learning rate 0.0005
- LoRA rank 16
- Low VRAM mode (768px)

These defaults are starting points for experimentation, not ideal values. Every subject and style is different.

**Learning rate advice:**
- 0.0005 trains fast but can overshoot, causing artifacts or burning in the subject too hard
- Try lowering to 0.0001 or 0.00005 for more stable, gradual training
- If your LoRA looks overcooked or the subject bleeds into everything, lower the learning rate
- If your LoRA is too weak after 500 steps, try more steps before raising the learning rate

## Credits

This project is a thin wrapper that calls these excellent training tools:

- **AI-Toolkit** by ostris: https://github.com/ostris/ai-toolkit
- **sd-scripts** by kohya-ss: https://github.com/kohya-ss/sd-scripts
- **Musubi Tuner** by kohya-ss: https://github.com/kohya-ss/musubi-tuner

All the heavy lifting is done by these projects. This node just makes them accessible from within ComfyUI.

## Author

Peter Neill - [ShootTheSound.com](https://shootthesound.com) / [UltrawideWallpapers.net](https://ultrawidewallpapers.net)

Background in music industry photography and video. Built this node to make LoRA training accessible to creators who just want to get things done without diving into command line tools.

Feedback is welcome - open an issue or reach out. There's a roadmap for future development including an export/import presets system and more.

## License

MIT
