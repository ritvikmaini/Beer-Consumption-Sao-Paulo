# Design: Visually Appealing README for Beer Consumption São Paulo

**Date:** 2026-06-08
**Status:** Approved

## Context

The repo solves the Kaggle [Beer Consumption – São Paulo](https://www.kaggle.com/datasets/dongeorge/beer-consumption-sao-paulo)
challenge: predict daily beer consumption (litres) from weather + calendar signals (365 rows, 2015).

A rigorous, leakage-free notebook (`beer_consumption_solution.ipynb`) already exists and is
executed clean. A brainstorming pass confirmed the model is **already optimal**: the honest
5-fold CV ceiling is **R² ≈ 0.70**, verified across ~10 techniques (OLS, Ridge, Lasso,
ElasticNet, polynomial features, spline features, robust regression, log-target, day-of-week
dummies, holiday features, and tree ensembles). Trees score *lower* (~0.63–0.65) — they overfit
365 rows. **There is no legitimate accuracy improvement left.**

Therefore the work is purely **presentation**: a portfolio-grade README that showcases the
existing solution, plus one reproducible figure.

## Goal

Produce a visually appealing, GitHub-portfolio-style `README.md` and a reproducible
model-comparison figure. No changes to the ML model or its conclusions.

## Scope

### In scope
1. **`data/model_comparison.png`** — CV-R² horizontal bar chart, linear models (green) vs tree
   models (orange), with fold-std error bars and value labels. Generated *from the notebook*
   (add a `savefig` to the existing comparison cell, then re-execute the notebook) so the
   figure stays in sync with the real computed numbers — single source of truth.
2. **`README.md`** — rewrite in Polished GitHub-portfolio style (replaces the current plain one).

### Out of scope
- Any change to the model, features, or reported metrics.
- Changes to `optional_challenge1.ipynb` (kept as historical reference).
- External image hosting (all figures live in `data/` and are committed).

## README structure (Polished style)

1. **Centered banner** — `<div align="center">` with 🍺 title, one-line tagline, and shields.io
   badges: Python version, scikit-learn, a "Learning / Educational" badge (the repo has no
   LICENSE file, so do not claim a specific OSS license), and a static "CV R²: 0.70" badge.
2. **📑 Table of contents** — anchor links to the sections below.
3. **Overview** — the challenge in 2–3 sentences + Kaggle dataset link.
4. **📦 Dataset** — feature table (column → meaning), note on 365 rows / 2015 / decimal-comma.
5. **📊 Results** — model leaderboard table (Lasso winner, CV R² 0.704 ± 0.052, RMSE 2.34 L,
   linear cluster ~0.70, trees ~0.63–0.65) + embedded `data/model_comparison.png`.
6. **🔑 Key Findings** — three points:
   - Temperature + weekend explain ~70% of variance; the rest is irreducible event noise.
   - Linear regression *wins*; tree ensembles overfit on 365 rows (complexity must match data size).
   - Honesty caveat: naïve notebooks report R² ≈ 0.77 via target leakage (rolling means of the
     target) or a lucky single split (R² swings 0.58–0.79 across seeds). This project uses
     5-fold CV with no target-derived features.
7. **🛠 Methodology** — collapsible `<details>` block: the 6-step pipeline (clean → EDA →
   leakage-free features → unified CV protocol → linear + tree comparison → inspect winner),
   with embedded `data/heatmap.png`.
8. **▶️ Run it** — `pip install ...` + `jupyter notebook beer_consumption_solution.ipynb`.
9. **📁 Repo structure** — file table.

## Data flow / source of truth

- The notebook is the single source of truth for all numbers and figures.
- The comparison cell will `plt.savefig("data/model_comparison.png", dpi=120, bbox_inches="tight")`.
- Re-executing the notebook regenerates both `heatmap.png` and `model_comparison.png`.
- The README references these committed PNGs by relative path.

## Verification

- Notebook re-executes end-to-end with **zero errors** (nbconvert `--execute`).
- `data/model_comparison.png` exists and is non-trivial in size.
- README renders valid GitHub-flavored markdown: all image paths resolve, all TOC anchors are
  valid, no broken tables, numbers match the notebook outputs exactly.

## Execution approach

Subagent-driven (per user request):
- **Task A (figure):** edit the notebook comparison cell to add the `savefig`, re-execute the
  notebook, confirm `data/model_comparison.png` is produced.
- **Task B (README):** author the Polished README per the structure above, using the exact
  numbers from the executed notebook; verify image paths and anchors.
- Task B depends on Task A (needs the figure to exist). Final step: verify and surface.
