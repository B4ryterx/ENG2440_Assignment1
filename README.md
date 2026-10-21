# Pneumonia detection on chest X-rays (RSNA 2018)

Binary classification of frontal chest X-rays (lung opacity consistent with possible pneumonia, yes/no)
on the RSNA 2018 Pneumonia Detection Challenge data, with patient-level splits, an ImageNet ResNet-18,
threshold selection on validation, calibration and an error analysis that looks for shortcut learning.
Coursework for ENG2440 (Fall 2026). **Learning exercise, not a clinical tool.**

## Results (held-out test set, 3,851 exams, 23.9% positive)

| Model | Accuracy | Sensitivity | Specificity | ROC-AUC |
|---|---|---|---|---|
| Trivial (always negative) | 0.761 | 0.000 | 1.000 | 0.500 |
| Logistic regression | 0.653 | 0.591 | 0.672 | 0.650 |
| ResNet-18 (seed 42, threshold 0.5) | 0.764 | 0.797 | 0.754 | 0.864 |

- **Seeds:** ROC-AUC 0.864 / 0.870 / 0.861 for seeds 42 / 1 / 2026 (0.865 ± 0.004). PR-AUC 0.691 against a prevalence of 0.239.
- **Operating points, chosen on validation and frozen before test:**
  - screening (validation sensitivity ≥ 0.90): test sensitivity 0.875, specificity 0.646;
  - high specificity (validation specificity ≥ 0.90): test sensitivity 0.598, specificity 0.903.
- **Subgroup finding:** sensitivity is 0.892 on AP (portable) films but only 0.422 on PA films (about 2,000 exams each). Grad-CAM often highlights image borders and markers rather than lung tissue. Together, these suggest the model partly uses acquisition signal, not only anatomy. Grad-CAM alone cannot prove this, so a border-masking control would be the next test.

## Method
- **Data:** 25,684 DICOM exams (1024 × 1024, MONOCHROME2), 22% positive.
  - Label = 1 if any annotation row of an exam has `Target = 1`.
- **Leakage-safe split:** grouped by `nih_patient_id` (via the RSNA → NIH mapping), not by exam, so no patient appears in two splits.
  - 17,947 train / 3,886 validation / 3,851 test exams.
  - Assertions check that the patient sets are disjoint.
  - The notebook also discusses the leak that patient grouping cannot stop (site and scanner fingerprints), and how a leave-site-out test would detect it.
- **Preprocessing:**
  - Per-image min–max scaling to [0, 1], resized to 224 × 224.
  - Augmentation: rotation ±10° and translation ±5% only. No flips: a horizontal flip would move the heart to the wrong side.
- **Model:**
  - ResNet-18 with ImageNet weights, adapted to one input channel and one output.
  - Trained as a feature extractor for 6 epochs, then fine-tuned for 4 epochs.
  - Class-weighted BCE loss (`pos_weight`), batch 64.
  - Trained on an NVIDIA RTX 5000 Ada (HPC).
- **Evaluation:** confusion matrix, ROC and PR curves, validation-chosen thresholds, reliability diagram, Grad-CAM on TP/TN/FP/FN cases, and an AP vs PA subgroup analysis.

## Repository contents
| Path | What |
|---|---|
| `ENG2440_Assignment1_Supporting_Files/Assignment1_Brandon-Jason.ipynb` | The executed notebook, Tasks A–G |
| `ENG2440_Assignment1_Supporting_Files/RSNA-2018-Pneumonia-Detection-Challenge-Dataset-Description.pdf` | Dataset description |
| `assignment1_labels.csv` | Labels supplied with the assignment |
| `rsna_to_nih_mapping.csv` | RSNA exam → NIH patient mapping, used for the patient-level split |
| `ai_use_declaration.txt` | How AI tools were used |

The DICOM images (~3.2 GB) are not in the repository (see `.gitignore`). Download them from the RSNA / Kaggle challenge page.

## Setup
```bash
python -m venv .venv
source .venv/bin/activate
pip install jupyter pandas numpy pydicom matplotlib scikit-learn torch torchvision
jupyter lab
```
Set `DATA_ROOT` (an environment variable or the first code cell) to the folder with the DICOM files, then run the notebook top to bottom.

## Data and references
- RSNA Pneumonia Detection Challenge (2018), Radiological Society of North America.
- X. Wang et al., ChestX-ray8, CVPR 2017 (the NIH source images and patient IDs).
- K. He et al., Deep residual learning for image recognition, CVPR 2016 (ResNet).
