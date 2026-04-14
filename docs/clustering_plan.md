## Context

Build a complete Jupyter notebook for Phase 1 K-Means clustering of 2,089 Madrid rental properties. The notebook follows the 7-stage structure defined in `clustering_workflow.md` (the authority document). Final clustering variables:  **Bedrooms + Floor** ,  **k=4** , `random_state=42`. Formatting follows `brief.md`: Arial font, `plotly_white`, `COLOR_PALETTE = ['#636EFA', '#EF553B', '#00CC96', '#AB63FA']`.

## Target File

`/Users/nicolaswilches/MBDS/term2/ml/group assignment/phase1.ipynb` — currently has 3 cells (title, setup header, imports). Build all remaining cells from here.

## Implementation

### Cell 2 (modify existing): Fix Setup

* Change `OUTPUT_DIR = Path('/mnt/user-data/outputs')` → `Path('./outputs')`
* Add font size constants: `FONT_SIZE_TITLE=16`, `FONT_SIZE_AXIS=12`, `FONT_SIZE_TEXT=11`

---

### Stage 1: Data Exploration (9 cells)

**Markdown** — Stage header: "Stage 1: Data Exploration"

**Code** — Load `data.xlsx`, print shape, `head(10)`, `describe()`, missing values summary sorted descending

**Code** — Histograms: `make_subplots(2,2)` with `go.Histogram` for Rent, Sq.Mt, Bedrooms, Floor

**Code** — Bar charts: `make_subplots(1,2)` horizontal bars for District and Area (top 20) frequencies

**Code** — Donut charts: `make_subplots(2,3, specs=domain)` with `go.Pie(hole=0.4)` for 6 binary variables (Elevator, Outer, Penthouse, Cottage, Duplex, Semidetached) — include NaN as "Missing" slice

**Markdown** — Variable classification table: Primary (Bedrooms, Sq.Mt, Floor), Secondary (Elevator, Outer, Penthouse, Cottage), Reserved for Phase 2 (District, Rent), Excluded (Id, Address, Number, Area, Duplex, Semidetached)

**Code** — Flag imbalanced binaries + missing values table: prevalence %, missing count/pct for each binary and continuous variable

**Markdown** — Stage 1 findings summary

---

### Stage 2: Data Manipulation (10 cells)

**Markdown** — Stage header: "Stage 2: Data Manipulation"

**Code** — Create `df_clean = df.copy()`. Drop rows where Rent is missing (these are useless for Phase 2 regression). Fix Floor=43039 error → set to NaN. Print counts for each operation.

**Code** — Define `cap_outliers_iqr(series, percentile=0.99)`: upper = min(Q3+1.5*IQR, 99th pctile), lower = max(Q1-1.5*IQR, 1st pctile). Uses `.clip()` which preserves NaN.

**Code** — Apply capping to Rent(4825), Sq.Mt(270), Bedrooms(4.5), Floor(9.5). Print before/after summary table with counts affected.

**Markdown** — Imputation strategy table:

* Rent: drop missing rows (unusable for Phase 2)
* Area: fillna 'Unknown' (needed for contextual imputation)
* Bedrooms: contextual median (District+Area → District → Global), same approach as Floor
* Floor: contextual median (District+Area → District → Global)
* Elevator/Outer: mode (1)

**Code** — Impute Area (fillna 'Unknown'). Must happen first since it's used in contextual lookups.

**Code** — Contextual Bedrooms imputation: precompute group medians as lookup tables (district+area median, district median, global median). 3-tier fallback. Same pattern as Floor.

**Code** — Contextual Floor imputation: precompute group medians as lookup tables (district+area, district, global) BEFORE applying — avoids row-order dependency. 3-tier fallback.

**Code** — Impute Elevator and Outer with mode=1.

**Code** — Verification: assert zero missing in critical columns `['Rent','Bedrooms','Sq.Mt','Floor','Outer','Elevator','Penthouse','Cottage','District']`. Print clean dataset shape.

---

### Stage 3: Statistical Screening (6 cells)

Purpose: Screen for multicollinearity and flag problematic variable combinations. This stage provides **intuition** about which combos are viable — the final variable selection happens in Stage 4 after empirical scenario testing.

**Markdown** — Stage header: "Stage 3: Statistical Screening". Note that this stage narrows the candidate pool; Stage 4 makes the final selection.

**Code** — Pairwise correlation heatmap (`go.Heatmap`, `RdBu_r`, `zmid=0`, text annotations) for 7 candidate vars. Highlight Sq.Mt↔Bedrooms r=0.787.

**Code** — Define `calculate_vif(df, variables)` function. Test 3 sets:

* Full 3-var Bedrooms+Sq.Mt+Floor (FAIL: Bedrooms VIF~13) → cannot use Bedrooms and Sq.Mt together
* Bedrooms+Floor (PASS: VIF~2.29) → viable combo
* Sq.Mt+Floor (PASS: VIF~2.12) → viable combo
* Conclusion: two viable base combos to test in Stage 4

**Code** — Binary standardization distances: for each binary var, compute `1/std` to show 3+ std dev separation after z-score. Print table. Flag domination risk for amenities.

**Markdown** — Stage 3 conclusions: Sq.Mt+Bedrooms eliminated (VIF>10). Two viable base combos identified: Bedrooms+Floor and Sq.Mt+Floor. Binary amenities flagged as domination risk. All combos proceed to Stage 4 for empirical testing.

**Code** — Summary print: viable combos, eliminated combos, domination warnings.

---

### Stage 3.5: K Selection (5 cells)

**Markdown** — Stage header: "Stage 3.5: K Selection"

**Code** — Standardize Bedrooms+Floor with `StandardScaler`. Store as `X_scaled`. Print means/stds to verify.

**Code** — Loop k=2..7: `KMeans(n_clusters=k, n_init=10, max_iter=300, random_state=42)`, store inertia + silhouette. Save `cluster_evaluation_metrics.csv`.

**Code** — **Elbow plot** (viz 1/6): `go.Scatter` lines+markers, annotate k=4. Save `figures/elbow_plot.html`.

**Code** — **Silhouette plot** (viz 2/6): `go.Scatter` lines+markers, annotate k=4. Save `figures/silhouette_scores.html`.

**Markdown** — K=4 justification: silhouette plateau, elbow point, 4 segments match market structure, actionable for business.

---

### Stage 4: Exploratory Clustering (7 cells)

**Markdown** — Stage header: "Stage 4: Exploratory Clustering"

**Code** — Define `test_scenario(df, variables, k=4)`: standardize → KMeans → return silhouette, max VIF, cluster size range, dominated check (any cluster has >99% or <1% for a binary var).

**Code** — Define all 12 scenarios dict:

1. Bedrooms+Floor (baseline)
2. Sq.Mt+Floor
   3-6. Bedrooms+Floor + each amenity (Elevator, Outer, Penthouse, Cottage)
3. Bedrooms+Floor + ALL 4 amenities
   8-11. Sq.Mt+Floor + each amenity
4. Sq.Mt+Floor + ALL 4 amenities

**Code** — Execute all 12, print results line by line, store in `scenario_df`.

**Code** — Display styled comparison table: highlight best silhouette, lowest VIF, flag dominated rows in red. Mark Scenario 1 as final selection.

**Markdown** — Scenario analysis: amenities increase silhouette but create dominated clusters. Bedrooms+Floor is best non-dominated result.

**Code** — Run FINAL model: `StandardScaler` + `KMeans(k=4)` on Bedrooms+Floor. Assign `df_clean['cluster']`. Print silhouette + distribution. Save `scaler.pkl` and `kmeans_model.pkl`.

---

### Stage 5: Quality Evaluation (8 cells)

**Markdown** — Stage header: "Stage 5: Quality Evaluation"

**Code** — Cluster size balance check: verify all clusters 10-50%. Print counts + percentages + PASS/FAIL.

**Code** — Within-cluster Rent CV: `std/mean` per cluster. Target 0.35-0.45.

**Code** — Between-cluster rent separation: median rent per cluster ordered, compute % gaps between adjacent clusters.

**Code** — **PCA scatter** (viz 3/6): `PCA(n_components=2)` on X_final. `px.scatter` colored by cluster with hover (Rent, Bedrooms, Floor, District). Save `figures/pca_clusters.html`.

**Code** — **Centroid heatmap** (viz 4/6): `go.Heatmap` of `final_model.cluster_centers_`, `RdBu_r`, `zmid=0`, text annotations. Save `figures/centroid_heatmap.html`.

**Code** — **Rent boxplot** (viz 5/6): `go.Box` per cluster with `boxmean='sd'`. Save `figures/rent_boxplot.html`.

**Code** — **Price/sqm boxplot** (viz 6/6): compute `price_per_sqm = Rent/Sq.Mt`, `go.Box` per cluster. Save `figures/price_per_sqm_boxplot.html`.

---

### Stage 6: Business Interpretability (5 cells)

**Markdown** — Stage header: "Stage 6: Business Interpretability"

**Code** — Examine centroids (standardized + original scale). Assign cluster names by matching centroid positions to labels:

* Small Apartments, Traditional Buildings (low bedrooms, low floor)
* Family Apartments, Traditional Buildings (high bedrooms, low floor)
* Small Apartments, Low-Rise Modern (low bedrooms, mid floor)
* Small Apartments, High-Rise Modern (low bedrooms, high floor)

Logic: sort centroids by Bedrooms/Floor to match names programmatically rather than hardcoding IDs.

**Code** — Comprehensive cluster profiles: count, pct, Bedrooms/Floor/Rent/Sq.Mt stats, amenity percentages, top 5 districts per cluster.

**Code** — Target demographics mapping + print.

**Code** — Save `cluster_profiles.csv`.

---

### Stage 7: Final Selection & Documentation (6 cells)

**Markdown** — Stage header: "Stage 7: Final Selection & Documentation"

**Code** — Decision matrix: 9 criteria (silhouette, VIF, balance, no domination, rent-contamination-free, interpretability, search alignment, Phase 2 setup, simplicity) with status + weight.

**Markdown** — Full written justification: technical, methodological, business, Phase 2 strategy.

**Code** — Save `madrid_rentals_clustered.csv` with all columns + cluster + cluster_label.

**Code** — Verify all outputs exist: 5 data files + 6 HTML figures. Print EXISTS/MISSING for each.

**Markdown** — Phase 2 variable strategy: target=Rent, predictors=Sq.Mt, District(one-hot), Bedrooms, Elevator, Outer, Penthouse, Cottage.

**Code** — Final summary print: dataset size, variables, k, silhouette, cluster names with counts and median rents.

---

## Output Files (11 total)

**Data** (`./outputs/`): `madrid_rentals_clustered.csv`, `cluster_profiles.csv`, `cluster_evaluation_metrics.csv`, `scaler.pkl`, `kmeans_model.pkl`

**Figures** (`./outputs/figures/`): `elbow_plot.html`, `silhouette_scores.html`, `pca_clusters.html`, `centroid_heatmap.html`, `rent_boxplot.html`, `price_per_sqm_boxplot.html`

## Implementation Notes

1. **Floor imputation** : Precompute group median lookup tables before applying to avoid row-order dependency
2. **Cluster naming** : Match centroid positions to names programmatically (sort by Bedrooms/Floor values) — don't hardcode cluster ID→name
3. **IQR capping** : Compute bounds on `.dropna()` values, apply `.clip()` to full column (NaN survives clip)
4. **12 scenarios** : Use Elevator, Outer, Penthouse, Cottage as the 4 amenities (Duplex/Semidetached already excluded)

## Verification

1. Run notebook top-to-bottom — all cells should execute without errors
2. Check `./outputs/` for all 5 data files and 6 HTML figures
3. Verify silhouette ~0.44, cluster sizes between 10-50%
4. Open HTML figures to confirm Plotly interactivity and correct formatting
5. Confirm `cluster_evaluation_metrics.csv` has k=2 through k=7 rows
