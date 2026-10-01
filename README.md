# Image Restoration Using Diffusion Models

A benchmark-based study and implementation of real-world image denoising using
**DDRM (Denoising Diffusion Restoration Models)** on the **SIDD** dataset.


---

## Problem Statement

Given a degraded image `y = Hx + z`, the goal is to recover the original clean image
`x`. This project focuses on the denoising case (`H = identity`): removing noise from
images without any dataset-specific training, using a pretrained diffusion model as a
generative prior.

## Novelty / Approach

- Uses **DDRM**, which solves linear inverse problems (including denoising) with an
  **off-the-shelf pretrained diffusion model** (OpenAI's ImageNet 256x256 unconditional
  DDPM) — no training from scratch, no SIDD-specific fine-tuning.
- Evaluated on **real-world smartphone camera noise** (SIDD), not just synthetic noise,
  and the two are directly compared to test how well a synthetic-noise-based diffusion
  restoration model generalizes to real sensor noise.
- Correct evaluation required recovering the true image order from DDRM's internal data
  shuffling via nearest-neighbor pixel matching — documented in `results/` with full
  per-image metrics for transparency.

## Methodology

```text
Dataset (SIDD) → Preprocessing (resize/crop to 256x256)
 → Degradation (real sensor noise, or controlled Gaussian noise)
 → DDRM Restoration (pretrained diffusion model, reverse diffusion sampling)
 → Evaluation (PSNR / SSIM / LPIPS) + qualitative comparison
```

- **Model**: DDRM (Kawar et al., NeurIPS 2022), pretrained ImageNet 256x256
  unconditional diffusion backbone
- **Degradation type**: denoising (`--deg deno`)
- **Sampling**: 20 timesteps, eta = 0.85, etaB = 1

## Dataset

**SIDD-Medium sRGB** — 160 real-world scene instances, 320 noisy/clean image pairs,
captured with 5 smartphone cameras under varying ISO, shutter speed, and lighting.

## Experiments

### Experiment A — Controlled Gaussian Noise

Clean SIDD ground-truth images (50 scenes) with synthetic Gaussian noise added at three
standard noise levels (σ = 15, 25, 50 on a 0–255 scale), restored with DDRM and
evaluated against the true clean image. Tests DDRM under its intended, well-defined
noise model.

### Experiment B — Real-World SIDD Noise

All 320 real noisy SIDD images, restored with DDRM and evaluated two ways: (1) against
DDRM's own reconstruction target, and (2) against the true clean SIDD ground truth.
Tests how well a synthetic-noise-based diffusion restoration model generalizes to real
camera sensor noise.

## Results

### Experiment A (mean over 50 images per noise level)

| Sigma | Noisy PSNR | Restored PSNR | Noisy SSIM | Restored SSIM | Noisy LPIPS | Restored LPIPS |
|-------|-----------|---------------|-----------|---------------|------------|----------------|
| 15    | 24.78     | 37.27         | 0.456     | 0.938         | 0.409      | 0.064          |
| 25    | 20.56     | 34.83         | 0.286     | 0.902         | 0.676      | 0.119          |
| 50    | 15.19     | 32.35         | 0.132     | 0.854         | 1.051      | 0.182          |

Restoration quality degrades smoothly as noise increases, and DDRM substantially
improves on the noisy input at every level. See `results/experiment_a_summary.md` and
`results/figures/experiment_a_qualitative.png`.

### Experiment B (320 real-world images)

| Metric | vs. DDRM's reconstruction target | vs. TRUE clean ground truth |
|--------|-----------------------------------|------------------------------|
| PSNR   | 33.96                             | 33.79                        |
| SSIM   | 0.868                             | 0.897                        |
| LPIPS  | 0.136                             | 0.125                        |

The two evaluations are nearly identical, showing that DDRM generalizes well to real
camera sensor noise rather than simply reconstructing its noisy input. See
`results/experiment_b_summary.md` and `results/figures/experiment_b_qualitative.png`.

## Limitations

- Images are center-cropped/resized to 256x256 to match the pretrained model's input
  size, so results are not directly comparable to published benchmarks that use
  full-resolution images.
- No SIDD-specific pretrained diffusion checkpoint exists; the ImageNet-pretrained
  backbone is used as a general natural-image prior.
- DDRM's internal data loader shuffles image order, which required an extra
  pixel-matching step to recover the correct ground-truth correspondence for Experiment
  B (documented in the code and results).
- Experiment A uses a 50-scene sample (not the full 160) for computational feasibility.

## Future Work

- Compare additional degradation types (blur, low-light) using the same pipeline.
- Evaluate with a SIDD-trained or fine-tuned diffusion backbone, if one becomes
  available, to isolate the effect of the generative prior from domain mismatch.
- Analyze computational efficiency (sampling steps vs. quality trade-off).

## Project Structure

```text
dataset/        SIDD dataset documentation (raw data kept out of git, see .gitignore)
preprocessing/  Resize/normalize/crop utilities
models/         DDRM / pretrained model references
restoration/    Restoration pipeline scripts
evaluation/     PSNR / SSIM / LPIPS evaluation scripts
results/        Summaries, metrics (CSV), and qualitative comparison figures
notebooks/      Kaggle notebook configs / reference files
```

## How to Reproduce

1. Clone the official [DDRM repository](https://github.com/bahjat-kawar/ddrm) and
   download the pretrained ImageNet 256x256 unconditional diffusion checkpoint.
2. Set `out_of_dist: True` and `subset_1k: False` in `configs/imagenet_256.yml` to feed
   in custom images (SIDD) instead of DDRM's built-in dataset.
3. Place noisy images in `exp/datasets/ood/sidd/` (or `sidd_expA/` for the controlled
   experiment), then run:

```bash
python main.py --ni --config imagenet_256.yml --doc imagenet \
  --timesteps 20 --eta 0.85 --etaB 1 --deg deno --sigma_0 <value> \
  -i <output_folder>
```

4. Evaluate outputs against ground truth using the scripts referenced in
   `evaluation/` (PSNR/SSIM via `scikit-image`, LPIPS via the `lpips` package).

This project was developed and run on [Kaggle Notebooks](https://www.kaggle.com/code/snahatrahman/image-restoration-ddrm) (2x Tesla T4 GPUs).

## References

- Kawar, B., Elad, M., Ermon, S., & Song, J. (2022). *Denoising Diffusion Restoration
  Models*. NeurIPS 2022. [arXiv:2201.11793](https://arxiv.org/abs/2201.11793)
- Abdelhamed, A., Lin, S., & Brown, M. S. (2018). *A High-Quality Denoising Dataset for
  Smartphone Cameras (SIDD)*. CVPR 2018.
