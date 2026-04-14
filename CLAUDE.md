# CLAUDE.md - Madrid Rental Properties Clustering Analysis

## Project Summary

**Objective:** Segment 2,089 Madrid rental properties into meaningful market groups using K-Means clustering (Phase 1), preparing clean segments for Phase 2 linear regression pricing models.

**Course:** Machine Learning I, MBDS Program
**Deadline:** February 28, 2026
**Report Constraint:** 4 pages + technical annex

**Key Principle:** Cluster on physical characteristics (what properties ARE), not economic variables (how much they COST). Reserve Rent for Phase 2 regression target to avoid circular logic.

---

## Authority Documents

| Document | Role | Priority |
|----------|------|----------|
| `clustering_workflow.md` | **Primary authority** -- defines 7-stage structure, variable selections, all decisions | Highest (use this when in doubt) |
| `brief.md` | Formatting specs, visualization standards, detailed methodology reference | Secondary |

**Critical discrepancy resolved:** `brief.md` originally recommended Sq.Mt + Floor as clustering variables. `clustering_workflow.md` specifies **Bedrooms + Floor**. User chose `clustering_workflow.md` as the authority. The notebook uses **Bedrooms + Floor**.

---

## Project Layout

```
.
├── data/raw/madrid_rentals.xlsx   # Source dataset (2,089 rows x 15 columns)
├── notebooks/
│   └── clustering_regression.ipynb   # Main analysis (123 cells, 7 stages)
├── outputs/                       # Generated — gitignored
│   ├── figures/                   # HTML plots
│   └── *.csv, *.pkl               # Data outputs + models
├── docs/
│   ├── clustering_workflow.md     # Authority doc for workflow structure
│   ├── clustering_plan.md         # Implementation plan
│   └── project_guidelines.pdf     # Course assignment spec
├── reports/
│   ├── main.tex                   # LaTeX source for deliverable
│   ├── clustering_regression.pdf  # Compiled report
│   └── figures/*.png              # Static figures for LaTeX
└── CLAUDE.md                      # This file
```

Notebook paths are relative to `notebooks/`: it reads `../data/raw/madrid_rentals.xlsx` and writes to `../outputs/`.

### Output Files

**Data files** (`outputs/`):
- `madrid_rentals_clustered.csv` -- full dataset with cluster assignments
- `cluster_profiles.csv` -- comprehensive profiles per cluster
- `cluster_evaluation_metrics.csv` -- k=2 through k=7 metrics
- `scaler.pkl` -- fitted StandardScaler for Phase 2
- `kmeans_model.pkl` -- trained KMeans model

**Figures** (`outputs/figures/`):
- `elbow_plot.html` -- inertia vs k
- `silhouette_scores.html` -- silhouette vs k
- `pca_clusters.html` -- Rent vs Sq.Mt scatter (not PCA; filename kept for compatibility)
- `centroid_heatmap.html` -- standardized centroids heatmap
- `rent_boxplot.html` -- rent distribution by cluster
- `price_per_sqm_boxplot.html` -- price per m2 by cluster

---

## Notebook Structure (7 Stages, 62 Cells)

### Stage 1: Data Exploration (Cells 0-12)
- Load data, descriptive stats, missing values
- Histograms for continuous variables (Rent, Sq.Mt, Bedrooms, Floor)
- Bar charts for categorical variables (District, Area)
- Donut charts for binary amenities (Elevator, Outer, Penthouse, Cottage, Duplex, Semidetached)
- Variable classification table

### Stage 2: Data Manipulation (Cells 13-22)
- Fix Floor = 43,039 error (set to NaN)
- Drop rows with missing Rent
- IQR outlier capping with 99th percentile safeguard
- Contextual median imputation for Bedrooms and Floor (3-tier: District+Area -> District -> Global)
- Mode imputation for Elevator and Outer
- Verification of zero missing values

### Stage 3: Statistical Screening (Cells 23-28)
- Pairwise correlation matrix
- VIF analysis (threshold: VIF < 10)
- Binary variable standardization distance analysis
- Conclusion: Bedrooms + Floor viable (VIF ~2.3); Sq.Mt + Bedrooms redundant (VIF >10, r=0.787)

### Stage 3.5: K Selection (Cells 29-34)
- K-Means for k=2 to k=7 on Bedrooms + Floor
- Inertia elbow plot + silhouette scores plot
- Selected k=4 (elbow + silhouette plateau + business interpretability)

### Stage 4: Exploratory Clustering (Cells 35-41)
- 12 scenario testing (2 base combos x 6 amenity configurations)
- Dominated cluster detection (>99% or <1% for any binary variable)
- Final selection: Scenario 1 (Bedrooms + Floor) -- best non-dominated result
- Silhouette = 0.431 for k=4

### Stage 5: Quality Evaluation (Cells 42-49)
- Cluster size balance check (all within 10-50%: 36.2%, 23.0%, 11.0%, 29.9%)
- Within-cluster Rent CV analysis
- Between-cluster rent separation
- Rent vs Sq.Mt scatter plot (continuous axes show cluster differentiation clearly)
- Centroid heatmap, rent box plots, price per sqm box plots

### Stage 6: Business Interpretability (Cells 50-54)
- Programmatic cluster naming based on centroid positions (not hardcoded IDs)
- Cluster profiles with geographic concentration and demographics
- Target demographics mapping

### Stage 7: Final Selection & Documentation (Cells 55-61)
- Decision matrix (technical + business + Phase 2 impact)
- Full justification
- Save all outputs
- Phase 2 preparation notes

---

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Clustering variables | **Bedrooms + Floor** | Authority doc; 9.7% better silhouette than Sq.Mt+Floor; aligns with customer search behavior |
| Number of clusters | **k = 4** | Elbow + silhouette plateau + natural 2x2 market structure |
| Include amenities in clustering? | **No** | Create dominated clusters; better as profiling variables |
| Bedrooms imputation | **Contextual median** (3-tier) | Bedroom counts vary by neighborhood (same approach as Floor) |
| Floor imputation | **Contextual median** (3-tier: District+Area -> District -> Global) | Floor levels correlate with building type by area |
| Missing Rent | **Drop rows** | Unusable for Phase 2 regression target |
| Outlier method | **IQR with 99th percentile safeguard** | Conservative; preserves high-end market while removing errors |
| Floor = 43,039 | **Treat as NaN** before IQR | Obvious data entry error |
| Standardization | **Z-score (StandardScaler)** | Equal variable weighting in distance calculations |
| Random state | **42** | Reproducibility |
| Output paths | **Relative to notebooks/** (`../outputs/`) | Structure: `outputs/` for CSVs/pkls, `outputs/figures/` for HTML |
| Scatter plot axes | **Rent vs Sq.Mt** | Both continuous; shows cluster differentiation clearly. Bedrooms/Floor are discrete and cause overplotting |
| VIF role | **Intuition in Stage 3** | Final variable selection deferred to Stage 4 empirical testing |
| 12 scenarios | **Keep all 12** | Comprehensive evaluation of all variable combinations |

---

## Style and Formatting

### Plotly Configuration
```python
pio.templates.default = 'plotly_white'
FONT_FAMILY = 'Arial'
FONT_SIZE_TITLE = 16
FONT_SIZE_AXIS = 12
FONT_SIZE_TEXT = 11
COLOR_PALETTE = ['#636EFA', '#EF553B', '#00CC96', '#AB63FA']
```

### Chart-Specific Styles

**Histograms (Cell 7):**
- 2x2 grid for Rent, Sq.Mt, Bedrooms, Floor
- Floor data filtered to exclude 43,039 error (`data[data < 1000]`) before plotting
- One color per variable from COLOR_PALETTE

**Donut charts (Cell 9):**
- 2x3 grid for Elevator, Outer, Penthouse, Cottage, Duplex, Semidetached
- Colors: Blue (#636EFA) = Yes, Red (#EF553B) = No, Grey (#CCCCCC) = Missing
- Unified horizontal legend at top via invisible scatter traces (standard Plotly pattern for custom legends with subplotted pies)
- Single `go.Pie` trace per variable with all labels/values together, `sort=False` to preserve consistent label ordering
- `hole=0.4`, `textinfo='percent+label'`

**Scatter plot (Cell 46):**
- Rent (y) vs Sq.Mt (x), colored by cluster
- `opacity=0.5` for overplotting
- Cluster medians shown as 'x' markers
- Hover shows Bedrooms, Floor, District
- Saved as `pca_clusters.html` (filename kept for compatibility with brief.md requirements)

**All visualizations:**
- Saved as interactive HTML in `./outputs/figures/`
- No emojis or excessive symbols in notebook output
- Clean, minimal print statements

### Output Style
- Use `=` and `-` separators for section headers
- No emojis in print output
- Format numbers with commas for thousands
- Currency with EUR symbol
- Percentages to 1 decimal place

---

## K-Means Configuration

```python
KMeans(n_clusters=4, n_init=10, max_iter=300, random_state=42)
```

### Results
- **Silhouette score:** 0.431
- **Cluster sizes:** 36.2%, 23.0%, 11.0%, 29.9% (all within 10-50%)
- **Clustering variables:** Bedrooms (standardized) + Floor (standardized)

### Cluster Naming Logic
Programmatic, not hardcoded -- based on centroid positions:
- Bedrooms above/below median -> "Family Apartments" / "Small Apartments"
- Floor rank (1-4) -> "Traditional Buildings" / "Low-Rise Modern" / "High-Rise Modern"

---

## Data Manipulation Details

### Outlier Caps (IQR + 99th percentile safeguard)
| Variable | Capped At |
|----------|-----------|
| Rent | ~4,825 |
| Sq.Mt | ~270 |
| Bedrooms | ~4.5 |
| Floor | ~9.5 |

### Imputation
| Variable | Missing % | Method |
|----------|-----------|--------|
| Area | 0.2% | Fill 'Unknown' (needed as grouping key) |
| Bedrooms | 4.3% | Contextual median: District+Area -> District -> Global |
| Floor | 6.8% (+1 error) | Contextual median: District+Area -> District -> Global |
| Elevator | 6.4% | Mode = 1 (88.8% prevalence) |
| Outer | 7.8% | Mode = 1 (87.7% prevalence) |

**Implementation note:** Contextual imputation uses precomputed lookup tables (not row-by-row apply) for performance.

---

## Phase 2 Preparation

**Target Variable:** Rent (EUR/month)

**Predictor Variables:**
- Sq.Mt -- quantify space premium
- District (one-hot encoded) -- quantify location premium
- Bedrooms -- additional size granularity within clusters
- Elevator, Outer, Penthouse, Cottage -- amenity value contributions

**Approach:** Build regression models per cluster (or combined model with cluster dummies) to identify undervalued properties.

---

## Known Issues and Gotchas

1. **pca_clusters.html is not PCA:** The scatter plot uses Rent vs Sq.Mt axes (both continuous, better visualization). The filename is kept as `pca_clusters.html` for compatibility with the 6 required HTML files listed in brief.md.

2. **Floor histogram:** The raw Floor data contains a 43,039 entry. In Cell 7, Floor data is filtered (`data < 1000`) before plotting. Do NOT use `update_xaxes(range=...)` on the histogram -- filter the data instead.

3. **Donut chart structure:** Each donut must be a SINGLE `go.Pie` trace per variable with all labels/values. Do NOT use individual traces per category (Yes/No/Missing) -- this breaks the Pie chart layout in subplots.

4. **Discrete variable scatter:** Bedrooms and Floor are integers, so scatter plots using them as axes show massive overplotting (dots stack on grid points). Use Rent vs Sq.Mt instead for cluster scatter visualization.

5. **Contextual imputation uses precomputed lookups:** The notebook computes district-level and district+area-level medians upfront, then uses `map()` and `fillna()` chains instead of `df.apply()`. This is much faster than row-by-row imputation.

6. **12 scenarios structure:**
   - Scenarios 1-6: Bedrooms base (alone, +Elevator, +Outer, +Penthouse, +Cottage, +ALL amenities)
   - Scenarios 7-12: Sq.Mt base (same amenity additions)

7. **Dominated cluster detection:** A cluster is "dominated" if any binary variable is >99% or <1% within that cluster. Higher silhouette with amenities is artificial -- the algorithm splits on binary yes/no rather than creating meaningful segments.

---

## Running the Notebook

```bash
# Activate environment
source /Users/nicolaswilches/MBDS/.venv/bin/activate

# Execute programmatically (run from notebooks/ so relative paths resolve)
cd notebooks && python3 -c "
import nbformat
from nbclient import NotebookClient
with open('clustering_regression.ipynb', 'r') as f:
    nb = nbformat.read(f, as_version=4)
client = NotebookClient(nb, timeout=300, kernel_name='python3')
client.execute()
with open('clustering_regression.ipynb', 'w') as f:
    nbformat.write(nb, f)
print('Done')
"
```

All cells should execute without errors. Output files are written to `../outputs/`.
