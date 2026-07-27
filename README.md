# SHG Cornea Analysis — Backward-to-Forward Image Restoration

A deep-learning study of Second Harmonic Generation (SHG) microscopy of corneal collagen, restoring interpretable **forward-SHG** structure from clinically accessible **backward-SHG** images, with a supporting analysis of how SHG signal behaves with tissue depth.

> **Status: work in progress (summer 2026 placement).** This README documents the project, the work so far, what I've learned, and what's next. Results are preliminary and will be updated as the analysis matures.

---

## Why this matters

SHG microscopy images collagen in the cornea without any staining or damage — useful for detecting diseases like keratoconus and glaucoma, where collagen structure is disrupted.

There are two SHG signals:

- **Forward-SHG** — clear, easy to interpret, but only measurable *ex vivo* (the detector sits behind the sample), so it can't be used on a living patient.
- **Backward-SHG** — measurable *in vivo* and therefore clinically usable, but noisier and harder to interpret.

If a model can reconstruct the clean forward image from the clinical backward image, backward-SHG becomes far more useful in practice. That's the core question here.

---

## Repository contents

| File | What it does |
|------|--------------|
| `1_processing.ipynb` | Loads raw `.tif` stacks, extracts the backward/forward channels, cuts 256×256 patches, normalises. |
| `2_training.ipynb` | U-Net training with a file-level train/val/test split; loss experiments. |
| `3_intensity_analysis.ipynb` | Diagnostic: mean SHG intensity vs tissue depth (in progress). |
| `README.md` | This file. |

_(File names being tidied to a numbered order for clarity.)_

---

## Data

- **7 corneal stacks** (porcine), imaged at 10× magnification.
- Each frame **1024 × 1024 px**, acquired as 5-channel `.czi`, converted to `.tif` in Fiji.
- **Backward = channel 2 (index 1), Forward = channel 5 (index 4)** — confirmed against the group's channel-extraction macro.
- Stacks span cornea (anterior/posterior), lenticule, and SMILE surgical scans at varying intraocular pressures.
- Split into 256×256 patches → **310 slices → 4,960 paired patches** total.
- z-step per stack read from `.tif` metadata (e.g. cornea 3.74 µm, lenticule/SMILE 0.81 µm).

---

## Method

### Preprocessing
Whole 1024×1024 slices are cut into 16 non-overlapping 256×256 patches. Intensities are normalised to [0, 1] using global 1st/99th-percentile scaling per channel (clipping outliers).

### Model
A 5-level U-Net (encoder channels 64→128→256→512→1024) with double-convolution blocks, InstanceNorm, dropout, and skip connections. Input: a backward patch; output: the predicted forward patch. This model and the processing pipeline were originally developed by **Melissa Diana Martins** in her BEng project; my work modifies and builds on them (file-level split, intensity analysis, documentation).

### Data splitting (file-level)
The train/validation/test split is done at the level of **whole stacks**, so every patch from one stack stays in a single split. This prevents leakage between structurally similar patches from the same tissue — a more conservative and honest evaluation than splitting patches at random. 

---

## Experiments so far

| Run | Loss | Outcome |
|-----|------|---------|
| 1 | MSE | Train MSE ~0.0001, val MSE ~0.005 — overfits, but predictions show real texture; the model learns something genuine. |
| 2 | SSIM + MSE (α = 0.2) | Val loss ~0.12; predictions collapse to bright blobs, val loss rises after ~epoch 130. SSIM term dominates and fails at this distribution shift. **Abandoned.** |

**Current direction:** revert to MSE-only with the file-level split, fully commented, as the cleaner baseline.

---

## Intensity–depth analysis (in progress)

A separate diagnostic asking: how does SHG signal change with depth, and do the forward and backward channels behave the same way?

Preliminary findings (on normalised data):
- Cornea and lenticule stacks show **roughly flat** mean intensity with depth — no simple exponential attenuation.
- The SMILE "around" scan shows a sharp **intensity minimum at the surgical plane**, present in *both* channels at the same depth — consistent with collagen disruption at the cut.

These are being **re-run on raw (un-normalised) intensities** to confirm the effect isn't an artefact of percentile clipping. Numbers may change; the qualitative features are expected to hold.

---

## What I've learned

- Verifying data *before* analysing it (channel assignment, frame size, z-intervals, NaN/dead/saturated screening) catches silent errors that would otherwise corrupt results.
- Loss choice matters enormously: a perceptual loss (SSIM) that sounds better can fail badly under distribution shift.
- Splitting strategy quietly determines whether evaluation is honest.
- For scientific restoration, faithfulness matters as much as sharpness: a model that invents plausible-looking structure can be worse than a blurrier but honest one. This shapes the choice of loss and architecture.

---

## Next steps

1. Finish the raw-intensity analysis; confirm the SMILE-plane dip and quantify it properly.
2. Re-run training: MSE-only, file-level split, fully commented.
3. Add a lightweight verification script (including a hash-based duplicate-patch check).
4. Compare against the group's original baseline.
5. Extend the dataset if more stacks become available.

---

## Acknowledgements

The original U-Net image-restoration model and processing pipeline were developed by **Melissa Diana Martins** as part of her BEng project — this work builds directly on hers. Paired SHG imaging data collected by the research group. Supervised by Dr Abby Wilson (In2research placement, 2026).
