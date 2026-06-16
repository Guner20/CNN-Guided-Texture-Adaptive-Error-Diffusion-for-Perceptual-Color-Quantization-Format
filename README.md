# CNN-TAD: CNN-Guided Texture-Adaptive Error Diffusion for Perceptual Color Quantization

> **COMP430 — Digital Image Processing**  
> Abdullah Gül University • Spring 2026  
> Author: Güner Bilgin  
> Instructor: Rifat Kurban

---

## Overview

CNN-TAD is a CNN-guided, texture-adaptive error diffusion pipeline for perceptual color quantization. A Median Cut algorithm reduces an input image to a K-color palette (K ∈ {64, 128, 256}). A lightweight CNN (~5,200 parameters) predicts a continuous per-patch diffusion coefficient α ∈ [0, 1], trained via cluster-amortized SSIM oracle labels. The predicted α map modulates Floyd–Steinberg error diffusion at each pixel: α ≈ 1 applies full diffusion in smooth regions (suppressing banding), while α ≈ 0 suppresses diffusion in textured regions (preserving structure).

**Key Finding:** The SSIM oracle consistently assigns α\* ≈ 0 across all patch types and palette sizes, demonstrating that — under SSIM-optimal evaluation — error diffusion *decreases* perceptual fidelity. CNN-TAD converges to match the no-diffusion Median Cut baseline while substantially outperforming fixed and variance-adaptive Floyd–Steinberg diffusion.

---

## Pipeline Architecture

```
┌────────────┐     ┌──────────────┐     ┌────────────────┐     ┌──────────────────────┐
│  Input RGB  │────>│  Median Cut  │────>│  Patch Extract │────>│  SSIM Oracle + KMeans│
│   Image     │     │  K-palette   │     │  24×24, s=12   │     │  C=512 centroids     │
└────────────┘     └──────────────┘     └────────────────┘     └──────────┬───────────┘
                                                                          │ α* labels
                                                               ┌──────────▼───────────┐
                                                               │    CNN Training       │
                                                               │  Conv(3→16→32) + GAP  │
                                                               │  MSE regression       │
                                                               └──────────┬───────────┘
                                                                          │ α map
┌────────────┐     ┌──────────────────────────────┐           ┌──────────▼───────────┐
│   Output   │<────│  Adaptive Floyd-Steinberg     │<──────────│  Per-pixel α map     │
│   Image    │     │  error diffusion              │           │  from CNN inference   │
└────────────┘     └──────────────────────────────┘           └──────────────────────┘
```

---

## Project Structure

```
.
├── final_project.py         # Complete pipeline (Stages 1–6), designed for Google Colab
├── README.md
└── cnntad/                  # Generated at runtime
    ├── kodak/               # Kodak24 dataset (auto-downloaded)
    ├── bsd500/              # BSD500 test images (auto-downloaded)
    ├── patches/             # Extracted 24×24 training patches
    ├── labels/              # SSIM oracle α* labels per K
    ├── models/              # Trained CNN weights (.pt)
    └── results/             # Metrics CSV, plots, ablation figures
```

---

## Requirements

- Python 3.10+
- PyTorch 2.x (GPU recommended)
- NumPy, scikit-image, scikit-learn, Pillow, matplotlib, tqdm, requests, pandas, scipy
- brisque (`pip install brisque`)

Install all dependencies:
```bash
pip install torch torchvision numpy scikit-image scikit-learn Pillow matplotlib tqdm requests pandas scipy brisque
```

---

## Usage (Google Colab)

This project is designed to run as a single script in **Google Colab** with a T4 GPU.

1. Upload `final_project.py` to Colab
2. Run the entire notebook — each stage executes sequentially:

| Stage | Description | Approx. Time |
|-------|-------------|---------------|
| 1 | Dataset download + patch extraction | ~2 min |
| 2 | SSIM oracle + K-means amortization | ~15 min |
| 3 | CNN training (3 models, 30 epochs each) | ~10 min |
| 4 | Adaptive diffusion on test images | ~20 min |
| 5 | PSNR / SSIM / BRISQUE evaluation (Kodak) | ~15 min |
| 5b | BSD500 generalization evaluation | ~60 min |
| 6 | Ablation studies (A–D) | ~45 min |

**Total runtime:** ~2.5 hours on Colab T4

---

## Methods Compared

| ID | Method | Description |
|----|--------|-------------|
| M1 | Median Cut | Nearest-neighbor quantization, no diffusion |
| M2 | Fixed Floyd-Steinberg | Standard full-strength error diffusion (α=1.0) |
| M3 | Variance-Adaptive | Per-pixel α from local 5×5 variance |
| M4 | **CNN-TAD (proposed)** | Per-patch α from CNN regression |

---

## Key Results

### Kodak24 (test images kodim21–24, mean metrics)

| K | Method | PSNR (dB) | SSIM | BRISQUE ↓ |
|---|--------|-----------|------|-----------|
| 64 | M1 MedCut | 32.05 | 0.918 | 27.29 |
| 64 | M4 CNN-TAD | 31.99 | 0.917 | 27.51 |
| 128 | M1 MedCut | 34.34 | 0.944 | 15.62 |
| 128 | M4 CNN-TAD | 34.29 | 0.943 | 15.65 |
| 256 | M1 MedCut | 36.68 | 0.961 | 9.08 |
| 256 | M4 CNN-TAD | 36.63 | 0.961 | 9.02 |

CNN-TAD matches Median Cut while outperforming both fixed FS (+5.79 dB at K=64) and variance-adaptive FS.

---

## Ablation Studies

- **Ablation A:** Regression vs. 3-class classification — both converge to α≈0
- **Ablation B:** Cluster count C — C=512 optimal, 200× cost reduction
- **Ablation C:** K-α interaction — α\* ≈ 0 across all K values
- **Ablation D:** Patch size sensitivity — 16/24/32 yield similar SSIM

---

## Reproducibility

- Fixed random seed: `SEED = 42` across all stages
- All results are deterministic given the same hardware
- Compute environment: Google Colab T4 GPU

---

## Citation

If you use this code, please cite:

```
Bilgin, G. (2026). CNN-TAD: CNN-Guided Texture-Adaptive Error Diffusion
for Perceptual Color Quantization. COMP430 Term Project, Abdullah Gül University.
```

---

## License

This project is for academic purposes (COMP430 course project). 

---

## Acknowledgements

This project was developed with assistance from Claude (Anthropic), ChatGPT (OpenAI), and Gemini (Google). These tools were used for technical pipeline design, code generation, and document editing. All experimental results were generated by running the code on Google Colab.
