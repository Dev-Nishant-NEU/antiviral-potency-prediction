# Antiviral Potency Prediction — Group XN
### ASAP Discovery | SARS-CoV-2 & MERS-CoV Mpro pIC50

> Machine learning pipeline for predicting antiviral potency against coronavirus main proteases (Mpro), built on the [ASAP Discovery](https://polarishub.io/datasets/asap-discovery/antiviral-potency-2025-unblinded) dataset from Polaris.

---

## Overview

This project predicts the inhibitory potency (pIC50) of small molecules against two coronavirus main protease targets:

- **SARS-CoV-2 Mpro** — 1,105 labeled compounds
- **MERS-CoV Mpro** — 1,198 labeled compounds

Molecules are represented as **ECFP4 Morgan fingerprints** (radius=2, 2048 bits) and trained independently per target using **Random Forest** and **XGBoost** regressors. External test set inference is performed on 76 Cresset compounds provided in OpenEye `.oez` format, parsed without any OpenEye license dependency.

---

## Results

### Cross-Validation Performance (5-fold, training set)

| Target | Model | CV R² | CV RMSE |
|---|---|---|---|
| SARS-CoV-2 Mpro | Random Forest | 0.652 ± 0.046 | 0.681 ± 0.036 |
| SARS-CoV-2 Mpro | XGBoost | 0.652 ± 0.044 | 0.680 ± 0.032 |
| MERS-CoV Mpro | Random Forest | 0.263 ± 0.059 | 0.810 ± 0.045 |
| MERS-CoV Mpro | XGBoost | 0.233 ± 0.081 | 0.826 ± 0.051 |

### External Test Set (76 Cresset compounds)

| Model | Target | R² | RMSE |
|---|---|---|---|
| Random Forest | SARS-CoV-2 | -0.083 | 0.966 |
| Random Forest | MERS-CoV | -1.719 | 1.530 |
| XGBoost | SARS-CoV-2 | -0.196 | 1.015 |
| XGBoost | MERS-CoV | -1.904 | 1.581 |

Negative external R² reflects structural dissimilarity between the Polaris training compounds and the Cresset test set — a known limitation of ECFP4-based models on out-of-distribution scaffolds.

---

## Repository Structure

```
.
├── checkin1_nishant.ipynb     # Full pipeline: data loading → featurization → training → inference
├── test_predictions.csv       # Predictions on the 76-compound Cresset external test set
└── README.md
```

---

## Pipeline

```
Polaris Dataset (asap-discovery/antiviral-potency-2025-unblinded)
        │
        ▼
Drop rows with no label for either target
        │
        ▼
ECFP4 Morgan Fingerprints (RDKit MorganGenerator, radius=2, 2048 bits)
        │
        ▼
Per-target NaN masking → 5-fold CV (cross_validate, single pass)
        │
        ├── Random Forest (n_estimators=200)
        └── XGBoost (n_estimators=200, lr=0.05, max_depth=6)
                │
                ▼
        Refit on full labeled data
                │
                ▼
        External inference on Cresset OEZ test set
        (pure-Python OEZ reader — no OpenEye license required)
```

---

## Setup

### Environment

This project uses the `openeye` conda environment (Python 3.11).

```bash
conda activate openeye
pip install xgboost
```

### Dependencies

| Package | Purpose |
|---|---|
| `polaris` | Dataset loading via Polaris Hub |
| `rdkit` | Morgan fingerprint generation (MorganGenerator API) |
| `scikit-learn` | Random Forest, cross-validation |
| `xgboost` | XGBoost regressor |
| `numpy`, `pandas` | Data handling |
| `matplotlib` | EDA visualizations |
| `ctypes` (stdlib) | OEZ binary parsing via libzstd |

### Running the Notebook

1. Place `Cresset_OE_Mpro_76cmpds.oez` in the same directory as the notebook (or update `OEZ_PATH`)
2. Open `checkin1_nishant.ipynb` and run all cells top to bottom
3. Predictions are saved to `test_predictions.csv`

---

## OEZ File Parsing

The external test set is provided as an OpenEye `.oez` binary file. Rather than relying on the OpenEye toolkit (which requires a license and can hang on initialization), we parse it directly using:

- **libzstd** (via Python `ctypes`) — the `.oez` format is a 3-byte magic header followed by 76 concatenated zstd-compressed frames, one per molecule
- **RDKit** — atoms and bonds are extracted from each frame's binary encoding (Kekulé form) and reconstructed into a sanitized mol object to generate canonical SMILES

This approach is fully self-contained and requires no proprietary licenses.

---

## Key Findings

- **SARS-CoV-2** cross-validation performance is moderate (R² ≈ 0.65), suggesting ECFP4 captures meaningful SAR signal for this target
- **MERS-CoV** performance is consistently weaker (R² ≈ 0.26), likely driven by assay noise at the low end of the pIC50 range (floor values at 1.0) and broader label distribution
- **External generalization is poor** for both targets, motivating future work on richer molecular representations (3D, GNNs) and multi-task learning to leverage shared target biology

---

## Next Steps

- Analyze MERS-CoV label distribution for censored/noisy measurements (pIC50 = 1.0)
- Compare ECFP4 hyperparameters (radius 3, 4096 bits) vs. alternative representations (MACCS keys, physicochemical descriptors)
- Explore multi-task neural networks to share learned representations across both targets
- SHAP analysis to validate learned structure–activity patterns vs. dataset artifacts

---

## Author

**Nishant Chaudhari** — Group XN, Week 10
