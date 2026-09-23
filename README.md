# larynx-swinunetr
# An Uncertainty-Guided Multi-Scale SwinUNETR Framework for 3D Larynx Segmentation in CT

Olga Menegaki, Konstantinos Georgas, George K. Matsopoulos

School of Electrical and Computer Engineering, National Technical University of Athens, Greece

📄 **Paper:** accepted — link and DOI will be added here after the conference.
🖼️ **Poster:** [`poster/larynx_poster_A0.pdf`](poster/larynx_poster_A0.pdf)
✉️ **Contact:** omenegaki@biomed.ntua.gr · [ORCID 0009-0005-8987-7355](https://orcid.org/0009-0005-8987-7355)

---

## Overview

Accurate larynx delineation on CT is a mandatory pre-processing step for radiotherapy
planning in head-and-neck cancer, and one of the harder organs to contour automatically:
soft-tissue contrast is low, anatomy is dense and variable, and tumours deform the
structures around them. Most work in this area targets healthy organs-at-risk only —
joint segmentation of healthy *and* tumour-bearing larynges is largely unaddressed.

This repository accompanies a segmentation framework that combines a SwinUNETR backbone
with multi-scale refinement and an uncertainty-guided calibration cascade.

**Components**

| Module | What it does |
| --- | --- |
| **MSFF3D** | Memory-efficient three-branch (1×, 2×, 0.5×) multi-scale feature fusion with per-voxel softmax weights |
| **MLF3D** | Cumulative multiplicative fusion across encoder stages — a response survives only when consistent across levels |
| **LC** | Top-down Localization Calibration guided by a voxel-wise uncertainty map, no Monte-Carlo sampling or ensembling |

Uncertainty is computed directly from the foreground probability as
`U = 1 - 4 (p_fg - 0.5)^2`, is fully differentiable, and modulates features through
cross-attention so that ambiguous boundary voxels receive amplified updates.

## Results

**Private cohort** (64 laryngeal-cancer + 59 healthy CT scans, 5-fold CV)

| Method | Class | mDice ↑ | HD95 ↓ | Precision ↑ | IoU ↑ |
| --- | --- | --- | --- | --- | --- |
| **Proposed** | Cancer | **0.7800** | **17.31** | 0.7780 | 0.6471 |
| SwinUNETR | Cancer | 0.7710 | 18.16 | 0.7573 | 0.6362 |
| nnU-Net | Cancer | 0.7676 | 17.66 | 0.7728 | 0.6358 |
| U-Net | Cancer | 0.7646 | 18.34 | 0.7707 | 0.6230 |
| **Proposed** | Healthy | 0.8851 | **7.22** | 0.8890 | 0.7975 |
| SwinUNETR | Healthy | 0.8804 | 7.59 | 0.8750 | 0.7900 |
| nnU-Net | Healthy | **0.8913** | 7.31 | 0.8960 | 0.8081 |
| U-Net | Healthy | 0.8821 | 7.70 | 0.8958 | 0.7910 |

On the cancer class the method achieves the best Dice and lowest HD95 of all compared
approaches (paired *t*-test, *p* < 0.05 against every baseline). On healthy anatomy it is
on par with nnU-Net (difference not significant) with a slightly better HD95.

**SegRap2023** (larynx subset, healthy only): Dice **0.9254**, HD95 2.54 mm — competitive
with nnU-Net and with published results on this benchmark.

## Data

- **SegRap2023** — public, available from the [challenge site](https://segrap2023.grand-challenge.org/).
- **Private cohort** — anonymized clinical CT scans of laryngeal-cancer and healthy
  subjects. Not publicly redistributable; contact the authors regarding access.

## Setup

```bash
git clone https://github.com/Olgameneg/larynx-swinunetr.git
cd larynx-swinunetr
pip install -r requirements.txt
```

Built on PyTorch and [MONAI](https://monai.io/). Experiments were run on an NVIDIA A100 (80 GB).

## Preprocessing and training configuration

- Patch size 128 × 128 × 128
- Intensities clipped to [−200, 500] HU, per-volume z-score normalization
- Resampled to 0.70 × 0.56 × 0.56 mm
- SwinUNETR encoder pretrained on BTCV, feature size 48, heads (3, 6, 12, 24)
- AdamW, lr 1e-4, weight decay 1e-5, up to 1000 epochs with early stopping
- Loss: Dice + cross-entropy
- Augmentation: random rotation, flipping, intensity scaling/shifting
- Sliding-window inference

## Repository layout

```
├── src/            # model, modules (MSFF3D, MLF3D, LC), training and inference
├── configs/        # experiment configurations
├── poster/         # conference poster (A0, PDF + LaTeX source)
└── README.md
```

## Citation

```bibtex
@inproceedings{menegaki2026uncertainty,
  title     = {An Uncertainty-Guided Multi-Scale SwinUNETR Framework
               for 3D Larynx Segmentation in CT},
  author    = {Menegaki, Olga and Georgas, Konstantinos and Matsopoulos, George K.},
  booktitle = {TODO},
  year      = {2026}
}
```

## Acknowledgments

No external funding supported this work. The authors declare no competing interests.

## License

TODO — MIT is the usual choice for research code; check your institution's policy first.
