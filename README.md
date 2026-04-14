# Madrid Rental Market — Clustering & Regression

Segment 2,089 Madrid rental properties into meaningful market groups (K-Means, Phase 1) and fit rent regression models per segment (Phase 2).

**Course:** Machine Learning I, MBDS Program
**Deliverable:** 4-page report + technical annex (see `reports/`)

## Layout

```
.
├── data/raw/                  # Source dataset (madrid_rentals.xlsx)
├── notebooks/                 # Main analysis notebook
├── outputs/                   # Generated CSVs, pkl models, HTML figures (gitignored)
│   └── figures/
├── docs/                      # Methodology: workflow, plan, guidelines PDF
├── reports/                   # LaTeX source, compiled PDF, PNG figures
├── CLAUDE.md                  # Project authority doc (read first)
├── requirements.txt
└── .gitignore
```

## Running

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/clustering_regression.ipynb
```

Running all cells regenerates everything under `outputs/`.

## Key decisions

- Clustering variables: **Bedrooms + Floor** (k=4, silhouette 0.431)
- Outlier handling: IQR capping with 99th percentile safeguard
- Missing values: contextual median (District+Area → District → Global)
- Random state: 42

Full rationale in `CLAUDE.md` and `docs/clustering_workflow.md`.
