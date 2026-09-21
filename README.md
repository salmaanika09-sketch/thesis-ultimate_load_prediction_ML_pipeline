# Prediction of Ultimate Load for High-Strength Steel Tubular T-Joints Using Machine Learning

**B.Sc. Thesis — Department of Building Engineering & Construction Management**  
**Rajshahi University of Engineering & Technology (RUET), Bangladesh**  
**July 2026**

**Authors:** Salma Anika · Md. Nafis Ul Abid · Shayed Al Shahab Shakib  
**Supervisor:** Faruque Abdullah, Assistant Professor, Dept. of BECM, RUET

---

## Overview

Hollow steel sections (HSS) are widely used in bridges, towers, and offshore platforms because of their high strength-to-weight ratio. The joints connecting these members — not the members themselves — usually govern structural failure. Current design codes such as Eurocode 3 and CIDECT rely on empirical equations calibrated for normal-strength steel, and they treat chord length only as a validity boundary, never as a predictor of capacity.

This project builds a machine-learning pipeline to predict the **peak load capacity (P_u)** of cold-formed high-strength steel tubular T-joints from their geometry and material properties alone, covering both RHS–RHS and RHS–CHS brace configurations.

---

## Dataset

- **611 specimens** compiled from published experimental studies (Pandey & Young, Pandey et al.)
- Two brace types: **rectangular (RHS–RHS, 66%)** and **circular (RHS–CHS, 34%)**
- Material: high-strength steel, yield strength ≈ 900–1060 MPa
- Raw features include chord and brace dimensions, wall thicknesses, and chord yield strength

---

## Key Methodology

### Physics-informed target
Instead of predicting P_u directly (skewness 2.33), the model learns the **dimensionless capacity coefficient C = P_u / (f_y0 · t_0²)** via its log-transform ln C (skewness −0.61). This embeds the CIDECT-form f_y0 · t_0² dependence that the near-constant yield strength (CoV 3.5%) cannot otherwise teach the model.

### Feature engineering
Nine engineered features are used:

| Feature | Definition | Note |
|---|---|---|
| β | b₁/b₀ | width ratio |
| τ | t₁/t₀ | thickness ratio |
| γ | b₀/(2t₀) | chord slenderness |
| η | h₁/b₀ | brace aspect ratio (0 for CHS) |
| Joint_Type | 1 = RHS, 0 = CHS | — |
| b₀ | chord width (mm) | — |
| b₁/d₁ | brace width or diameter (mm) | — |
| **α** | **L₀/(2b₀)** | **novel: chord aspect ratio** |
| ρ | h₀/b₀ | chord section ratio |

f_y0 and t₀ are held in the normaliser, not given as free features.

### Models trained
Eleven algorithms were trained inside leakage-free `Pipeline([StandardScaler, estimator])` wrappers: MLR, SVR, RF, XGBoost, LightGBM, **CatBoost**, ANN, GBR, Extra Trees, GPR, and a stacking ensemble.

### Hyperparameter tuning
Three-stage strategy for each model: Grid Search → Randomised Search → Bayesian optimisation (Optuna), all evaluated on 5-fold CV RMSE of ln C on the training set only.

### Validation
- 10-fold cross-validation (primary, in-domain)
- Stratified 80/10/10 hold-out test
- Leave-One-Source-Out (extrapolation upper bound)
- Friedman–Nemenyi statistical comparison

---

## Results

| Model | CV R² | CV MAPE | Test R² | Test MAPE |
|---|---|---|---|---|
| **CatBoost** | **0.985** | **5.30%** | **0.987** | **5.44%** |
| GBR | 0.984 | 6.31% | 0.988 | 6.64% |
| Stacking | 0.984 | 5.48% | 0.989 | 5.65% |
| LightGBM | 0.982 | 5.88% | 0.982 | 6.54% |
| XGBoost | 0.979 | 6.13% | 0.985 | 5.79% |
| GPR | 0.974 | 6.81% | 0.983 | 6.46% |
| MLR | 0.926 | 15.93% | 0.955 | 14.73% |

CatBoost is the **champion model**: CV R² = 0.985, RMSE = 122 kN, MAPE = 5.30%, mean prediction ratio μ = 1.001, CoV = 8.56%.

---

## Novel Finding: Chord Length is a Real Predictor

A controlled ablation experiment with paired Wilcoxon testing showed that **removing α (chord length ratio) collapses CV R² from 0.985 to 0.865** (ΔR² = −0.12, p ≈ 4 × 10⁻⁵³). This is the **first statistical proof** that chord length is an active strength driver — not just a code boundary — and that it acts through an **α × β interaction** visible in the 2-D partial dependence plot.

---

## Interpretability

- **SHAP analysis:** β dominates (mean |SHAP| = 0.495), followed by η, γ, and α
- **1-D PDP + ICE:** β shows a strong marginal effect; α is modest marginally but decisive under ablation
- **2-D PDP (β × α):** α's effect is large at high β and negligible at low β — a classic interaction signature

---

## Web Deployment

The trained CatBoost model is exported to **ONNX format** and runs directly in the browser via ONNX Runtime Web. The tool is deployed as a self-contained HTML page on Netlify — no server, no backend, no installation required.

🔗 [Live predictor →](https://your-netlify-link-here)

---

## Repository Structure

```
├── pipeline.ipynb          # Main training notebook (Cells 1–30)
├── polish.ipynb            # Figure and table polish (Cell 31+)
├── export_onnx.py          # Export CatBoost to ONNX and verify
├── TJoint_Predictor.html   # Browser-based prediction tool
├── tjoint_catboost.onnx    # Exported CatBoost model (ONNX format)
├── data/
│   └── TT4.xlsx            # Dataset (611 specimens)
└── outputs/
    └── figures/            # All thesis figures (600 DPI)
```

---

## Requirements

```
catboost
xgboost
lightgbm
scikit-learn
optuna==3.6.1
shap
onnxruntime
numpy
pandas
matplotlib
seaborn
scipy
```

Install all dependencies:
```bash
pip install catboost xgboost lightgbm scikit-learn optuna==3.6.1 shap onnxruntime numpy pandas matplotlib seaborn scipy
```

---

## Citation

If you use this code or dataset in your work, please cite:

> Abid, M. N. U., Anika, S., & Shakib, M. S. A. S. (2026). *Prediction of Ultimate Load for High-Strength Steel Tubular T-Joints Using Machine Learning Techniques*. B.Sc. Thesis, Department of BECM, Rajshahi University of Engineering & Technology, Bangladesh.

---

## Supervisor

**Faruque Abdullah**  
Assistant Professor, Department of BECM, RUET
