# SHG Cornea Analysis

**In2Research Placement — Summer 2026**
Nimo Abdinor · UCL Biochemical Engineering
Supervisor: Dr Abby Wilson · West Drayton

## Based On
Original code and U-Net architecture developed by Melissa (previous student, UCL).
This repo contains modifications made by Nimo Abdinor during the placement.

## Modifications
- Removed forward channel binarisation — kept as continuous normalised images
- Per-file patch extraction and saving to Google Drive
- Explored alternative data splitting strategy: replaced patch-level split with file-level split to reduce potential overlap between training     and test patches from the same z-slice
- Train: cornea files + lenticule + SMILE 25mmHg | Val: SMILE 14mmHg | Test: SMILE 10mmHg
- Changed loss from BCEWithLogitsLoss to MSELoss (image restoration)
- Added torch.sigmoid() to model output before loss computation

## Baseline Results (Run 1)
Train MSE: 0.0001 | Val MSE: 0.005
Overfitting observed. Next step: add SSIM to loss function.

## Architecture
U-Net with 5 encoder/decoder levels (64-128-256-512-1024 channels),
DoubleConv blocks, InstanceNorm2d, Dropout 0.3, skip connections.
Input/output: single-channel 256x256 patches.

## Run 2 — SSIM+MSE Loss (alpha=0.2)
Train loss: 0.0005 | Val loss: 0.12
Result: model collapsed to predicting bright blobs — no fibre structure recovered.
Val loss increasing after epoch 130 — overfitting.
Conclusion: SSIM weight too high even at alpha=0.2. MSE-only baseline produces better visual results.
Next: intensity drop-off analysis (signal attenuation with depth).