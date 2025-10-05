# DA5401 A5 — Visualizing Data Veracity on the Yeast Multi‑Label Dataset

This repository/notebook bundle solves **DA5401 A5: Manifold Visualization** using **t‑SNE** and **Isomap** to inspect data‑veracity issues (noisy/ambiguous labels, outliers, and hard‑to‑learn samples) on the **Yeast** multi‑label dataset (86 features, 14 labels). It includes **two runnable paths**:
- **Option A — Real Yeast**: load official Yeast data from CSV/ARFF.
- **Option B — Synthetic**: generate a Yeast‑like dataset offline (fully reproducible, no internet required).

---

## Contents

- `DA5401_A5_Manifold_Visualization.ipynb`  
  Turn‑key notebook implementing the whole assignment with both data options.
- `DA5401_A5_StepByStep.ipynb`  
  A numbered, **step‑by‑step** version aligned to Parts **A**, **B**, **C** with extra diagnostics.
- `DA5401_A5_StepByStep_CheatSheet.md`  
  One‑page checklist of steps and reasoning for quick reference.
- `README_DA5401_A5.md` (this file)

> If your folder is empty, place the notebook(s) next to this README or download them again from your submission workspace.

---

## Quick Start

1. **Environment (Python ≥ 3.10)**  
   Install dependencies in your environment (only once):
   ```bash
   # in a terminal or a Jupyter cell (with %)
   pip install numpy pandas scikit-learn matplotlib scipy liac-arff scikit-multilearn
   ```

2. **Choose your data option**  
   - **Option A — Real Yeast**
     - Place **`yeast_X.csv`** (shape `[n, 86]`) and **`yeast_Y.csv`** (shape `[n, 14]`) in the same folder; **OR** place **`yeast.arff`** where the last 14 columns are labels.
     - In the notebook, set:
       ```python
       DATA_SOURCE = "real"
       ```
   - **Option B — Synthetic (offline)**
     - No files needed. In the notebook, keep:
       ```python
       DATA_SOURCE = "synthetic"
       ```

3. **Run a notebook**  
   - Open `DA5401_A5_StepByStep.ipynb` (recommended) or `DA5401_A5_Manifold_Visualization.ipynb` in Jupyter/VS Code.
   - Run all cells top‑to‑bottom. The notebook will:
     - Load/preprocess the data and **standardize** features.
     - Create simplified viz labels (top‑2 singles + most frequent multi‑label combo + **Other**).
     - Compute **t‑SNE** (perplexities 5/30/50) and **Isomap** (n_neighbors≈12).
     - Plot embeddings colored by the simplified labels.
     - **Flag** likely **ambiguous**, **outlier**, and **mixed/high‑entropy** points for discussion.

---

## What the Code Does

### Part A — Preprocessing & Initial Setup (10 pts)
- **A1–A2**: Load **X** (`n×86`) and **Y** (`n×14`), report shapes.
- **A3**: Derive a **viz class** with four categories: top single label #1, top single label #2, most common label combo, **Other**.
- **A4**: **Standardize** `X` (zero mean, unit variance) so distance‑based DR is not dominated by high‑variance features.

### Part B — t‑SNE & Veracity Inspection (20 pts)
- **B1**: Run t‑SNE for **perplexity** in `{5, 30, 50}`; pick a final value (often **30** balances fragmentation vs. blurring).
- **B2**: Plot the final 2‑D map, color by viz class.
- **B3**: **Veracity diagnostics** on the t‑SNE plane (k‑NN with `k=15`):
  - **Ambiguous/noisy**: point label ≠ neighborhood majority (red ring overlay).
  - **Outliers**: top 1% by mean k‑NN distance (black `x` overlay).
  - **Mixed/Hard‑to‑learn**: top 5% by neighborhood label **entropy** (orange ring overlay).

### Part C — Isomap & Manifold Learning (20 pts)
- **C1**: Fit **Isomap** (`n_neighbors≈12`) to get a 2‑D embedding that preserves **global geodesic structure**.
- **C2**: Plot with the same coloring; optional k‑NN metrics on the Isomap plane.
- **C3**: **Compare** t‑SNE vs Isomap; discuss **manifold curvature** and **classification difficulty**.

---

## Discussion Notes

### Global vs Local Structure (Isomap vs t‑SNE)
- **t‑SNE** preserves **local neighborhoods** → excels at **cluster clarity** and spotting local label mixing; may distort long‑range geometry.
- **Isomap** preserves **global geodesic** layout → reveals **macro‑structure** (trajectories/gradients) that t‑SNE might fragment into islands.

### Manifold Curvature & Classifier Complexity
- Strong curves/folds in Isomap imply a **non‑linear manifold**. Expect **linear models** to underfit. Prefer **non‑linear** capacity (kernel SVM, tree ensembles, NNs) or add **feature learning** to “unfold” the manifold.

### Mixed / High‑Entropy Areas & Model Difficulty
- High label entropy in neighborhoods indicates **intrinsic ambiguity** or **noisy labels**. Expect higher Bayes error, lower confidence, and unstable predictions. Use **uncertainty‑aware** evaluation (calibration, PR‑AUC, macro/micro‑F1), consider **label audits/cleaning**, and prefer **multi‑label‑aware** models (e.g., classifier chains).

> A ready‑to‑paste Markdown version of this discussion is also included in the notebooks.

---

## Reproducibility

- Random seed fixed at `RANDOM_STATE = 42` where applicable. Note that **t‑SNE is stochastic**; re‑runs can change fine details.
- Synthetic generator fixes **n≈2400**, **d=86**, **L=14**, and injects controlled outliers and 5% label flips to surface veracity patterns.

---

## Deliverables Checklist (per the assignment)
- [ ] Notebook with: preprocessing, **t‑SNE** (perplexity sweep + final), **Isomap**, and plots colored by viz class.
- [ ] Short **justification** of chosen perplexity and `n_neighbors`.
- [ ] Visual identification + explanation of **ambiguous/noisy**, **outliers**, **mixed/hard‑to‑learn** regions.
- [ ] Comparison of **t‑SNE vs Isomap** and notes on **manifold curvature**.
- [ ] Brief comment on how these issues **impact classifier performance**.

---

## Troubleshooting

- **File not found (real data)**: Ensure `yeast_X.csv` + `yeast_Y.csv` **or** `yeast.arff` are in the same folder as the notebook.
- **Wrong shapes**: Expect **86 features** and **14 labels**. If using ARFF, the dataset variant must have the **last 14 columns** as labels.
- **t‑SNE too slow/fragmented**: Reduce `n_samples` temporarily, try `perplexity=30`, keep `init='pca'`.
- **Isomap artifacts**: Tune `n_neighbors` (8–20) to balance disconnected graphs vs. over‑smoothing.
