# Visually Appealing README Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce a portfolio-grade `README.md` showcasing the existing leakage-free solution, plus one reproducible model-comparison figure generated from the notebook.

**Architecture:** No ML changes (model already at the honest ~0.70 ceiling). Task 1 makes the notebook's comparison cell save its chart to `data/model_comparison.png` and re-executes the notebook so the figure reflects the real computed numbers. Task 2 authors the Polished README that embeds both figures and reports the exact notebook numbers.

**Tech Stack:** Jupyter (python3 kernel, Python 3.11), matplotlib/seaborn, GitHub-flavored markdown, shields.io badges.

---

### Task 1: Generate `data/model_comparison.png` from the notebook

**Files:**
- Modify: `beer_consumption_solution.ipynb` (cell id `166b6c6e`, the "Model comparison & winner" code cell)
- Produces: `data/model_comparison.png`

**Context:** The comparison cell currently ends with `plt.tight_layout(); plt.show()`. We add a `savefig` so the bar chart is written to disk, then re-execute the whole notebook so every figure and number is regenerated consistently.

- [ ] **Step 1: Edit the comparison cell to save the figure**

In cell `166b6c6e`, replace this exact line:

```python
plt.tight_layout(); plt.show()
```

with:

```python
plt.tight_layout()
plt.savefig("data/model_comparison.png", dpi=120, bbox_inches="tight")
plt.show()
```

Use the Edit tool on the `.ipynb` JSON, or `jq`/python to patch the cell source. Confirm only that one cell's `source` changed.

- [ ] **Step 2: Re-execute the notebook end-to-end**

Run:

```bash
jupyter nbconvert --to notebook --execute --inplace beer_consumption_solution.ipynb --ExecutePreprocessor.timeout=300
```

Expected: `Writing ... bytes to beer_consumption_solution.ipynb`, no traceback.

- [ ] **Step 3: Verify zero errors and the figure exists**

Run:

```bash
python3 -c "import json; nb=json.load(open('beer_consumption_solution.ipynb')); print('errors:', sum(1 for c in nb['cells'] for o in c.get('outputs',[]) if o.get('output_type')=='error'))"
ls -l data/model_comparison.png
```

Expected: `errors: 0` and `data/model_comparison.png` present with size > 10 KB.

- [ ] **Step 4: Commit**

```bash
git add beer_consumption_solution.ipynb data/model_comparison.png data/heatmap.png
git commit -m "Save model comparison chart from notebook for README"
```

---

### Task 2: Author the Polished GitHub-portfolio README

**Files:**
- Modify (overwrite): `README.md`

**Context:** Use the exact numbers from the executed notebook (do not invent any):
- Leaderboard (CV R² / CV RMSE in L): Lasso 0.704 / 2.342 · ElasticNet 0.704 / 2.345 · Ridge 0.703 / 2.349 · LinearRegression 0.699 / 2.361 · GradientBoosting 0.653 / 2.521 · RandomForest 0.646 / 2.556 · HistGradientBoosting 0.625 / 2.630.
- Winner: **Lasso**, CV R² = 0.704 ± 0.052, RMSE 2.34 L.
- Held-out (20% test, seed 42): R² 0.744, RMSE 2.380 L, MAE 1.990 L.
- Correlations: MaxTemp r≈0.64, Weekend r≈0.51, Rainfall r≈-0.19.
- Mean consumption ≈ 25.4 L; 365 rows; year 2015.
- Figures to embed: `data/model_comparison.png` (Results), `data/heatmap.png` (Methodology).
- No LICENSE file exists → use a "Learning / Educational" badge, NOT a specific OSS license.

- [ ] **Step 1: Write `README.md`**

Overwrite `README.md` with the following content exactly:

````markdown
<div align="center">

# 🍺 Beer Consumption — São Paulo

**Predicting daily beer consumption from weather & calendar signals**

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3-F7931E?logo=scikit-learn&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-data-150458?logo=pandas&logoColor=white)
![CV R²](https://img.shields.io/badge/CV%20R²-0.70-2a9d8f)
![Purpose](https://img.shields.io/badge/purpose-learning-blue)

<sub>Kaggle: <a href="https://www.kaggle.com/datasets/dongeorge/beer-consumption-sao-paulo">Beer Consumption – São Paulo</a></sub>

</div>

---

## 📑 Contents

- [Overview](#overview)
- [Dataset](#-dataset)
- [Results](#-results)
- [Key Findings](#-key-findings)
- [Methodology](#-methodology)
- [Run it](#️-run-it)
- [Repo structure](#-repo-structure)

---

## Overview

Predict **daily beer consumption (litres)** in a São Paulo university district from weather
and calendar signals. The dataset covers every day of **2015** (365 rows). The challenge:
train linear models (LinearRegression, Ridge, Lasso, ElasticNet), tune them, and find the best.

This project does that **honestly** — with proper cross-validation and **no target leakage** —
and benchmarks the linear models against tree ensembles to confirm the winner.

## 📦 Dataset

| Column | Meaning |
|---|---|
| `Avg / Min / Max Temp (°C)` | Daily temperatures |
| `Rainfall (mm)` | Daily precipitation |
| `Weekend` | 1 if Saturday/Sunday |
| **`Beer Consumption (L)`** | **Target** |

> 365 daily records · year 2015 · numbers stored with a decimal **comma** (`27,3`), fixed during cleaning.

## 📊 Results

**Lasso wins** with a 5-fold cross-validated **R² = 0.704 ± 0.052** (RMSE ≈ 2.34 L on a mean of ≈ 25 L).

| Model | CV R² | CV RMSE (L) |
|---|---|---|
| 🥇 **Lasso** | **0.704** | **2.34** |
| ElasticNet | 0.704 | 2.35 |
| Ridge | 0.703 | 2.35 |
| LinearRegression | 0.699 | 2.36 |
| GradientBoosting | 0.653 | 2.52 |
| RandomForest | 0.646 | 2.56 |
| HistGradientBoosting | 0.625 | 2.63 |

![Model comparison](data/model_comparison.png)

> On a held-out 20% test set the model scores **R² = 0.744, RMSE 2.38 L, MAE 1.99 L**.

## 🔑 Key Findings

1. **Temperature + weekend explain ~70% of the variance.** Max temperature (r ≈ 0.64) and the
   weekend flag (r ≈ 0.51) are the two dominant drivers; rain has a small negative effect
   (r ≈ -0.19). The remaining ~30% is day-specific event noise no weather/calendar model can recover.
2. **Linear regression is the *correct* model, not a fallback.** Tree ensembles score **lower**
   (~0.63–0.65) — with only 365 rows they overfit. Model complexity should match data size.
3. **Beware inflated scores.** A naïve notebook can report R² ≈ 0.77 on this data via
   **target leakage** (rolling means of the target leak the answer) or a **lucky split** (R²
   swings 0.58–0.79 across random seeds). This project uses 5-fold CV with no target-derived
   features — so the reported ~0.70 is trustworthy and reproducible.

## 🛠 Methodology

<details>
<summary><b>Click to expand the full pipeline</b></summary>

1. **Clean** — fix decimal-comma numbers, parse dates, drop trailing empty rows.
2. **EDA** — correlation analysis + scatter/box plots to find the real drivers.
3. **Leakage-free feature engineering** — temperature range, weekend×temperature interaction,
   rain flag, cyclical month encoding. **No target-derived features.**
4. **One fair protocol** — 5-fold cross-validation reporting R² and RMSE with fold-level
   standard deviation, applied identically to every model.
5. **Model comparison** — four linear models (tuned via `GridSearchCV`) vs. three tree
   ensembles; winner chosen on CV.
6. **Inspect the winner** — held-out evaluation, standardised-coefficient importance, residual plot.

### Feature correlations

![Correlation heatmap](data/heatmap.png)

</details>

## ▶️ Run it

```bash
pip install pandas numpy scikit-learn matplotlib seaborn jupyter
jupyter notebook beer_consumption_solution.ipynb
```

## 📁 Repo structure

| Path | Description |
|---|---|
| `beer_consumption_solution.ipynb` | **The solution** — full, reproducible analysis |
| `data/beer_consumption.csv` | Raw dataset |
| `data/model_comparison.png` | Model leaderboard chart (regenerated by the notebook) |
| `data/heatmap.png` | Feature correlation heatmap (regenerated by the notebook) |
| `optional_challenge1.ipynb` | Original first-pass notebook (kept for reference) |
| `docs/superpowers/` | Design spec & implementation plan |

---

<div align="center">
<sub>Built with scikit-learn · The honest answer beats the impressive-looking one.</sub>
</div>
````

- [ ] **Step 2: Verify image paths and rendered structure**

Run:

```bash
ls data/model_comparison.png data/heatmap.png
grep -c "](data/" README.md
```

Expected: both PNGs exist; `grep` reports `2` (both image embeds present).

- [ ] **Step 3: Verify numbers match the notebook**

Run:

```bash
grep -E "0.704|0.744|2.34|0.625" README.md | head
```

Expected: lines containing the winner CV R² (0.704), held-out R² (0.744), and the worst tree score (0.625) — confirming numbers are the real ones.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "Add polished portfolio README with results chart and findings"
```

---

## Self-Review

- **Spec coverage:** Task 1 produces `data/model_comparison.png` from the notebook (spec §In scope 1). Task 2 produces the Polished README with all 9 structural sections, both embedded figures, the leaderboard, key findings incl. the honesty caveat, collapsible methodology, run instructions, and repo structure (spec §README structure). Educational badge per the spec ambiguity fix. ✓
- **Placeholder scan:** No TBD/TODO; all README content is literal; all commands have expected output. ✓
- **Consistency:** Numbers in Task 2 match the executed notebook outputs read in this session (Lasso 0.704, held-out 0.744, HistGBM 0.625). Figure filename `data/model_comparison.png` identical in both tasks. ✓
