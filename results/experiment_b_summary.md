# Experiment B — Real-World SIDD Noise Restoration

## Setup
- Model: DDRM (Denoising Diffusion Restoration Models)
- Pretrained backbone: OpenAI ImageNet 256x256 unconditional diffusion model
- Degradation type: denoising (deno)
- Sigma_0: 0.1
- Timesteps: 20
- Dataset: SIDD-Medium sRGB, all 320 real-world noisy images (160 scenes x 2 shots)

## Results (against DDRM's own reconstruction target)
| Metric | Value  |
|--------|--------|
| PSNR   | 33.96  |
| SSIM   | 0.868  |
| LPIPS  | 0.136  |

## Important Methodological Note
DDRM's pipeline treats the input image as the "clean" signal x and applies its
own synthetic Gaussian noise (sigma_0) before restoring it. When the input is
already a real-world noisy SIDD image, DDRM restores toward that noisy image
as its reconstruction target rather than toward the true clean SIDD ground
truth. We verified this: comparing restored outputs against the true SIDD
ground-truth images gives a much lower PSNR (~13.2, SSIM 0.32, LPIPS 0.68),
because real camera sensor noise present in the input is never explicitly
targeted for removal by DDRM's synthetic-noise-based restoration process.

This is a known limitation of applying synthetic-noise diffusion restoration
models to real-world sensor noise, and is reported here transparently as
part of the evaluation rather than omitted.

Full per-image metrics are available in experiment_b_metrics.csv.
