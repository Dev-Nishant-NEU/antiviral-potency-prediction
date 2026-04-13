# Antiviral Potency Prediction — Final Submission
### Group XN 
### ASAP Discovery | SARS-CoV-2 Mpro pIC50

> Machine learning pipeline for predicting antiviral potency against the SARS-CoV-2 main protease (Mpro), built on the [ASAP Discovery](https://polarishub.io/datasets/asap-discovery/antiviral-potency-2025-unblinded) dataset from Polaris Hub.

---

## Overview

This project predicts the inhibitory potency (pIC50) of small molecules against **SARS-CoV-2 Mpro** using ECFP Morgan fingerprints and two tree-based models. An external test set of 76 Cresset compounds (provided in OpenEye `.oez` format) is parsed and evaluated without any OpenEye license dependency.

**Dataset:** `asap-discovery/antiviral-potency-2025-unblinded`  
**Target:** SARS-CoV-2 Mpro pIC50 — 1,105 labeled molecules, range 4.00–8.73  
**External test set:** 76 Cresset compounds (`Cresset_OE_Mpro_76cmpds.oez`)

---

## Results

### Cross-Validation Performance (5-fold, training set)

| Model | CV R² | CV RMSE |
|---|---|---|
| Random Forest | 0.652 ± 0.046 | 0.681 ± 0.036 |
| XGBoost | 0.652 ± 0.044 | 0.680 ± 0.032 |

### External Test Set (76 Cresset compounds)

| Model | R² | RMSE |
|---|---|---|
| Random Forest | -0.083 | 0.966 |
| XGBoost | -0.196 | 1.015 |

Negative external R² reflects structural dissimilarity between the Polaris training compounds and the Cresset test set — an expected outcome for fingerprint-based models on out-of-distribution scaffolds.

---

## Repository Structure

```
.
├── checkin1_nishant.ipynb         # Full pipeline: data → features → training → inference
├── test_predictions.csv           # Predictions on the 76-compound Cresset external test set
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
ECFP Morgan Fingerprints
(RDKit MorganGenerator, radius=5, 4096 bits — GraphSim default)
        │
        ▼
NaN mask SARS-CoV-2 labels → 5-fold CV (cross_validate, single pass)
        │
        ├── Random Forest (n_estimators=200)
        └── XGBoost (n_estimators=200, lr=0.05, max_depth=6)
                │
                ▼
        Refit on full labeled training data
                │
                ▼
        External inference on Cresset OEZ test set
        (pure-Python parser — no OpenEye license required)
                │
                ▼
        test_predictions.csv
```

---

## Setup

### Environment

```bash
conda activate openeye
pip install xgboost
```

### Dependencies

| Package | Purpose |
|---|---|
| `polaris` | Dataset loading via Polaris Hub |
| `rdkit` | Morgan fingerprint generation (`MorganGenerator` API) |
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

The external test set is in OpenEye's proprietary `.oez` binary format. Rather than using the OpenEye toolkit (which requires a license and can hang on initialization), we parse it directly:

- **libzstd** (via Python `ctypes`) — the `.oez` format is a 3-byte magic header followed by 76 concatenated zstd-compressed frames, one per molecule
- **RDKit** — atoms and bonds are extracted from each frame's Kekulé-form binary encoding and reconstructed into a sanitized RDKit mol to generate canonical SMILES

This approach is fully self-contained and requires no proprietary licenses.

---

## Key Findings

- SARS-CoV-2 cross-validation performance is moderate (R² ≈ 0.65), suggesting ECFP Morgan fingerprints capture meaningful SAR signal for this target
- External generalization is poor (R² < 0), reflecting structural dissimilarity between Polaris training compounds and Cresset test compounds — a known limitation of fingerprint-based models on out-of-distribution scaffolds
- RF and XGBoost perform essentially identically, suggesting the bottleneck is the feature representation rather than model architecture

---

## Next Steps

- Incorporate 2D descriptors (MW, XLogP, TPSA, rotatable bonds, etc.) and shape/color features as additional inputs
- Investigate multi-task learning to leverage MERS-CoV data alongside SARS-CoV-2
- Explore richer molecular representations (3D descriptors, GNNs) to improve external generalization
- SHAP analysis to validate learned structure–activity patterns vs. dataset-specific correlations

---

*Group XN | ASAP Antiviral Potency Dataset*
