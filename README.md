# Breast Cancer Detection with CellViT + Hybrid Pooling

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Model: CellViT](https://img.shields.io/badge/Model-CellViT-green)]()
[![Data: TCGA](https://img.shields.io/badge/Data-TCGA-lightgrey)]()

Fine-tuned CellViT (Cell Vision Transformer) for breast cancer detection from whole slide images. Uses **Hybrid Pooling (Top-K + Mean)** to aggregate patch-level predictions into slide-level diagnosis. Tested on 86 external TCGA slides.

---

## Pipeline Overview
```

Step 1: Load Fine-tuned CellViT Model (pre-trained on PanNuke)
↓
Step 2: Extract Patches from 86 External TCGA Slides (~200 patches/slide)
↓
Step 3: Generate Patch-level Malignancy Probabilities
↓
Step 4: Hybrid Pooling (Top-K Mean + Global Mean, α=0.7, K=100)
↓
Step 5: Slide-level Diagnosis with Threshold = 0.5

```
---

## Key Results

| Metric | Value |
|:---|:---|
| **Test Slides** | 86 (55 Malignant, 31 Benign) |
| **Accuracy** | 88.37% |
| **Precision** | 90.91% |
| **Recall (Sensitivity)** | 90.91% |
| **F1-Score** | 0.909 |
| **AUC-ROC** | 0.9179 |

### Confusion Matrix
| | Predicted Benign | Predicted Malignant |
|:---|:---:|:---:|
| **Actual Benign** | TN = 26 | FP = 5 |
| **Actual Malignant** | FN = 5 | TP = 50 |

---

## Model Architecture

**This implementation uses the fine-tuned weights from `cellvit_256.pth`. The base CellViT model was pre-trained on the PanNuke dataset (200,000+ labeled nuclei across 19 tissue types, including breast and colon), enabling robust feature extraction for histopathology.**

| Component | Specification |
|:---|:---|
| **Base Model** | CellViT (Cell Vision Transformer) |
| **Model Variant** | CellViT-Small (embed_dim=384) |
| **Weight File** | `cellvit_256.pth` |
| **Pre-training** | PanNuke dataset (200,000+ labeled nuclei) |
| **Fine-tuning Data** | 200 breast histopathology slides (137 train / 63 val) |
| **Fine-tuning Strategy** | Frozen Encoder + Trainable Decoder (Transfer Learning) |
| **Trainable Parameters** | 1,474,434 (~1.5M, decoder heads only) |
| **Embedding Dimension** | 384 |
| **Transformer Layers** | 12 |
| **Attention Heads** | 6 |
| **Source Repository** | [TIO-IKIM/CellViT](https://github.com/TIO-IKIM/CellViT) |

---

## Hybrid Pooling Strategy

| Parameter | Value |
|:---|:---|
| **K (Top Patches)** | 100 |
| **α (Top-K Weight)** | 0.7 |
| **Threshold** | 0.5 |

**Formula:** `Slide_Score = α × Top-K_Mean + (1-α) × All_Mean`

**Rationale:** Top-K focuses on highly suspicious regions, Mean provides global context. Hybrid balances sensitivity and specificity.

---

## Pooling Methods Comparison

| Method | Recall | False Positives |
|:---|:---:|:---:|
| Mean Pooling | 81.82% | 5 |
| Top-100 Pooling | 92.73% | 7 |
| **Hybrid (α=0.7)** | **90.91%** | **5** |

Hybrid pooling achieves the **best balance** between high recall and low false alarms.

---

## Error Analysis

### False Negatives (Missed Cancers): 5 slides
- All missed cases had **low malignancy scores** (31-48%), indicating subtle/low-grade tumors.
- Model was uncertain but threshold (0.5) classified them as benign.

### False Positives (False Alarms): 5 slides
- All false alarms were benign slides with **inflammation or dense tissue** mistaken for tumor.
- Scores ranged from 54-66%, showing moderate model confidence.

---

## Repository Structure
```

Breast-Cancer-CellViT-Hybrid-Pooling/
├── README.md
├── LICENSE
├── .gitignore
├── docs/
│   └── cellvit_report.md
├── pdfs/
│   └── cellvit_output.pdf
└── results/
└── roc_curve_hybrid_pooling.png

```
---

## Author

**F.P. Aghaei**
- Project Date: April 2026
- License: MIT

---

## Citation

If you use this work, please cite:
Aghaei, F.P. (2026). Breast Cancer Detection with CellViT + Hybrid Pooling.
Fine-tuned CellViT model with Top-K + Mean aggregation.
GitHub: https://github.com/AghaeiPhD/Breast-Cancer-CellViT-Hybrid-Pooling
