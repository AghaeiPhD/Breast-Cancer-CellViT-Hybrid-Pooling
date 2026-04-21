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

**This implementation is built upon the official [CellViT](https://github.com/TIO-IKIM/CellViT) codebase and utilizes the `cellvit_256.pth` checkpoint, a fine-tuned variant of CellViT-Small (embed_dim=384). The base model was pre-trained on the PanNuke dataset—a comprehensive collection of over 200,000 labeled nuclei across 19 tissue types, including breast and colon. For this project, the pre-trained encoder is used as a powerful feature extractor and fine-tuned on 200 breast cancer histopathology slides (137 train / 63 validation) sourced from the GDC (Genomic Data Commons) TCGA-BRCA cohort. The model is then evaluated on a completely independent test set of 86 external TCGA slides, with no overlap between fine-tuning and test data. Notably, this fine-tuning process does not require nucleus masks, as only the encoder is adapted for slide-level classification.**

| Component | Specification |
|:---|:---|
| **Base Model** | CellViT (Cell Vision Transformer) |
| **Model Variant** | CellViT-Small (embed_dim=384) |
| **Weight File** | `cellvit_256.pth` |
| **Pre-training** | PanNuke dataset (200,000+ labeled nuclei) |
| **Fine-tuning Data** | 200 breast histopathology slides (137 train / 63 val) from GDC TCGA-BRCA |
| **Fine-tuning Strategy** | Frozen Encoder + Trainable Decoder (Transfer Learning) |
| **Trainable Parameters** | 1,474,434 (~1.5M, decoder heads only) |
| **Embedding Dimension** | 384 |
| **Transformer Layers** | 12 |
| **Attention Heads** | 6 |
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

### Cross-Project Consistency Analysis
- **False Negatives:** 3 out of the 5 missed cases in this study were **also missed** by our previous [ABMIL + H-optimus-0](https://github.com/AghaeiPhD/Breast-Cancer-Detection-System-WSI-Analysis) pipeline.
- **False Positives:** 4 out of the 5 false alarms in this study were **also misclassified** by the ABMIL pipeline.
- **Interpretation:** The high overlap in errors across two completely different architectures (CellViT with Hybrid Pooling vs. ABMIL with Attention) strongly suggests that these specific slides represent **inherently challenging cases**:
  - The 3 consistently missed malignant slides likely contain **low-grade or diffuse tumors** that are fundamentally difficult for patch-based methods.
  - The 4 consistently misclassified benign slides likely contain **dense inflammation or atypical benign tissue** that mimics malignancy.
- **Conclusion:** These errors are **data-centric** rather than model-centric. Addressing them would require **higher magnification scans**, **multi-resolution analysis**, or **incorporation of clinical context**, not merely architectural changes.

---

## Limitations & Data-Centric Considerations

Although the pipeline operates on fixed-size patches extracted from whole slide images, variations in scanner resolution (MPP: microns-per-pixel) were not explicitly normalized during preprocessing.

As a result, identical patch sizes may correspond to different physical tissue areas across slides. This introduces potential scale inconsistency, which may affect the model’s sensitivity to:

- Subtle or low-grade tumors requiring fine-grained spatial resolution  
- Benign inflammatory regions that may resemble malignant patterns at different scales  

Importantly, similar failure patterns are observed across both CellViT and ABMIL-based pipelines, suggesting that these limitations are primarily **data-centric rather than model-specific**.

Future improvements could include:
- MPP normalization or resolution standardization
- Multi-scale or hierarchical patch sampling
- Incorporation of contextual slide-level information
- Future work may include incorporating datasets with segmentation annotations to provide spatial guidance and improve robustness.


Despite this limitation, the model demonstrates consistent performance on the external TCGA test set, suggesting robustness to moderate resolution variation within the current dataset.

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
Affiliation: Independent Researcher  
- Project Date: March - April 2026  
- License: MIT

---

## Citation

If you use this work, please cite:
Aghaei, F.P. (2026). Breast Cancer Detection with CellViT + Hybrid Pooling.
Fine-tuned CellViT model with Top-K + Mean aggregation.
GitHub: https://github.com/AghaeiPhD/Breast-Cancer-CellViT-Hybrid-Pooling
