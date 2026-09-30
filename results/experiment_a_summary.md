# Experiment A — Controlled Gaussian Noise Restoration

## Setup
- Model: DDRM with the OpenAI ImageNet 256x256 unconditional diffusion model (pretrained)
- Degradation: synthetic Gaussian noise added by DDRM to clean ground-truth images (deno)
- Noise levels: sigma = 15, 25, 50 on the 0-255 pixel scale (passed to DDRM as sigma/255)
- Timesteps: 20, eta = 0.85, etaB = 1
- Data: 50 SIDD-Medium scenes (random sample, seed 42, shot 010), center-cropped to 256x256
- Reference: the true clean ground-truth image for every metric

## Results (mean over 50 images per noise level)
| Sigma | Noisy PSNR | Restored PSNR | Noisy SSIM | Restored SSIM | Noisy LPIPS | Restored LPIPS |
|-------|-----------|---------------|-----------|---------------|------------|----------------|
| 15    | 24.78     | 37.27         | 0.456     | 0.938         | 0.409      | 0.064          |
| 25    | 20.56     | 34.83         | 0.286     | 0.902         | 0.676      | 0.119          |
| 50    | 15.19     | 32.35         | 0.132     | 0.854         | 1.051      | 0.182          |

Higher is better for PSNR and SSIM; lower is better for LPIPS.

## Notes
- Restored quality decreases smoothly as the noise level increases, and restoration
  improves on the noisy input at every level.
- The noisy-input PSNR is slightly above the theoretical value for unclipped Gaussian
  noise (24.61, 20.17, 14.15 dB), which is consistent with pixel clipping.
- Images are downscaled to 256x256, so absolute values are not directly comparable to
  published benchmarks on full-resolution images; the relative trend is the finding.

Per-image metrics: experiment_a_metrics.csv
