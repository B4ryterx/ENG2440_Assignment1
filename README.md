# ENG2440 — Assignment 1

RSNA 2018 Pneumonia Detection Challenge.

## Contents

- `ENG2440_Assignment1_Supporting_Files/Assignment1_Brandon-Jason.ipynb` — main notebook (deliverable)
- `ENG2440_Assignment1_Supporting_Files/` — supporting data and documentation
  - `assignment1_labels.csv` — assignment labels
  - `rsna_to_nih_mapping.csv` — RSNA→NIH ID mapping
  - `pneumonia-challenge-annotations-*.json` — challenge annotations (original / adjudicated)
  - `pneumonia-challenge-dataset-mappings_2018.json` — dataset mappings
  - `RSNA-2018-Pneumonia-Detection-Challenge-Dataset-Description.pdf` — dataset description

## Data not in this repo

The raw image dataset is large (~3.2 GB) and is **excluded** from version control (see `.gitignore`):

- `pneumonia-challenge-dataset-original_2018.zip` — download separately from the RSNA / Kaggle challenge page.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install jupyter pandas numpy pydicom matplotlib scikit-learn
jupyter lab
```
