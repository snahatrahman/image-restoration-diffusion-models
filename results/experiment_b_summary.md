# Experiment B — Real-World SIDD Noise Restoration

## Setup
- Model: DDRM (Denoising Diffusion Restoration Models)
- Pretrained backbone: OpenAI ImageNet 256x256 unconditional diffusion model
- Degradation type: denoising (deno)
- Sigma_0: 0.1
- Timesteps: 20
- Dataset: SIDD-Medium sRGB, all 320 real-world noisy images (160 scenes x 2 shots)

## Results
- Total Average PSNR: 33.97
- Number of samples: 320

## Notes
Full output set (960 images: original, noisy, restored) generated on Kaggle.
This folder contains a representative sample of 15 scenes (45 images total)
for quick visual reference; the full run is reproducible via the notebook.
