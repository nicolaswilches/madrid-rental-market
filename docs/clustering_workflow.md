# Madrid Rental Properties Clustering - Workflow Layout

## Project Overview

**Objective:** Segment 2,089 Madrid rental properties using K-Means clustering for Phase 2 pricing analysis

**Approach:** 7-stage systematic methodology with empirical testing

**Final Variables:** Bedrooms + Floor (k=4)

---

## Stage 1: Data Exploration

### Tasks

- Execute initial data exploration (head, describe, info)
- Analyze distributions (histograms for continuous variables)
- Categorical frequencies (bar charts for District, Area)
- Binary distributions (donut charts for amenities)
- Identify primary vs secondary candidates
- Flag imbalanced binaries and missing values

### Decisions Made

- **Primary candidates:** Bedrooms, Sq.Mt, Floor
- **Secondary candidates:** Elevator, Outer, Penthouse, Cottage
- **Reserved for Phase 2:** District, Rent
- **Excluded:** Id, Address, Number, Area, Duplex, Semidetached
- **Flagged imbalanced:** Elevator (88.8%), Outer (87.7%), Penthouse (8.1%)
- **Missing values:** Bedrooms (4.3%), Floor (6.8%), Elevator (6.4%), Outer (7.8%)

### Output

- Candidate pool: 7 variables for testing
- Visual understanding of data distributions
- Initial quality assessment

---

## Stage 2: Data Manipulation

### Tasks

- Handle Floor = 43,039 error (treat as missing)
- Cap outliers using IQR + 99th percentile method
- Impute missing values (contextual for Floor, median for Bedrooms, mode for amenities)
- Verify data quality post-cleaning

### Decisions Made

- **Outlier caps:** Rent (€4,825), Sq.Mt (270m²), Bedrooms (4.5), Floor (9.5)
- **Imputation methods:**
  - Bedrooms: median (2.0)
  - Floor: district + area median fallback
  - Elevator/Outer: mode (1)
- **Verification:** Zero missing in critical columns

### Output

- Clean dataset: 2,089 properties, all critical columns complete
- Outliers capped preserving market reality
- Ready for statistical analysis

---

## Stage 3: Statistical Screening

### Tasks

- Calculate pairwise correlation matrix
- Run VIF analysis (threshold: VIF < 10)
- Check standardization distances for binary variables
- Address multicollinearity

### Decisions Made

- **Multicollinearity detected:** Sq.Mt + Bedrooms (VIF = 13.16, correlation = 0.787)
- **Resolution:** Must choose ONE
  - Tested Bedrooms + Floor: silhouette = 0.4455, VIF = 2.29
  - Tested Sq.Mt + Floor: silhouette = 0.4061, VIF = 2.12
- **Choice:** Bedrooms + Floor (9.7% better silhouette, aligns with customer search)
- **Binary distances:** All amenities create 3+ std dev separation (potential dominance)

### Output

- **Clustering variables selected:** Bedrooms, Floor
- **Profiling variables:** Sq.Mt, Rent, District, amenities
- **Phase 2 predictors:** Sq.Mt, District (one-hot), amenities
- VIF clean: all < 3

---

## Stage 3.5: K Selection

### Tasks

- Test k=2 to k=7 with final variables (Bedrooms + Floor)
- Calculate inertia and silhouette for each k
- Generate elbow and silhouette plots
- Select optimal k based on metrics + interpretability

### Decisions Made

- **Optimal k:** 4
- **Justification:**
  - Silhouette plateau at k=4
  - Elbow visible at k=4
  - 4 segments match market structure (small/large × old/modern)
  - Actionable number for business strategy

### Output

- Optimal k = 4
- Elbow plot saved
- Silhouette scores plot saved
- Ready for scenario testing

---

## Stage 4: Exploratory Clustering

### Tasks

- Define test scenarios (12 combinations)
- Standardize variables within each scenario
- Run K-Means (k=4, random_state=42) for each
- Calculate silhouette, VIF, check dominated clusters, cluster balance
- Document all results

### Decisions Made

- **Test scenarios:**
  1. Bedrooms + Floor (baseline)
  2. Sq.Mt + Floor (comparison)
  3. Bedrooms + Floor + Elevator
  4. Bedrooms + Floor + Outer
  5. Bedrooms + Floor + Penthouse
  6. Bedrooms + Floor + Cottage
  7. Bedrooms + Floor + ALL amenities
  8-12. Similar tests with Sq.Mt base

- **Findings:**
  - Bedrooms + Floor: silhouette 0.4455, balanced clusters, no domination
  - Adding amenities: higher silhouette BUT dominated clusters (100% vs 0%)
  - Amenities better as profiling variables

- **Final selection:** Bedrooms + Floor (clean, interpretable, best non-dominated result)

### Output

- Comparison table (12 scenarios)
- Best approach identified: Bedrooms + Floor
- Amenities excluded from clustering
- StandardScaler and K-Means model saved

---

## Stage 5: Quality Evaluation

### Tasks

- Check cluster size balance
- Calculate within-cluster coefficient of variation for Rent
- Measure between-cluster rent separation
- Generate PCA visualization

### Decisions Made

- **Cluster sizes:** 11%, 29%, 44%, 16% (all within 10-50% range)
- **Within-cluster CV:** 0.35-0.45 (moderate homogeneity)
- **Between-cluster gaps:** Adequate separation in rent levels
- **PCA:** 69% variance explained in 2D

### Output

- Quality metrics confirmed
- PCA scatter plot saved
- Clusters are well-formed

---

## Stage 6: Business Interpretability

### Tasks

- Name each cluster clearly
- Profile each cluster (size, floor, rent, amenities, top districts)
- Map to target demographics
- Create business characterization

### Decisions Made

- **Cluster names:**
  - Cluster 0: Small Apartments, Traditional Buildings
  - Cluster 1: Family Apartments, Traditional Buildings
  - Cluster 2: Small Apartments, Low-Rise Modern
  - Cluster 3: Small Apartments, High-Rise Modern

- **Target demographics identified:**
  - Cluster 0: Students, young professionals
  - Cluster 1: Established families
  - Cluster 2: Young couples, professionals
  - Cluster 3: Urban professionals, modern living

### Output

- Cluster profiles saved
- Business characterization complete
- Clear stakeholder communication ready

---

## Stage 7: Final Selection & Documentation

### Tasks

- Create decision matrix (technical + business + Phase 2 impact)
- Write full justification
- Document alternatives tested
- Define Phase 2 variable strategy
- Save all outputs

### Decisions Made

- **Final clustering variables:** Bedrooms + Floor
- **Technical quality:** Silhouette 0.4455, VIF 2.29
- **Business alignment:** Customer search behavior (bedroom count primary filter)
- **Phase 2 strategy:** Use Sq.Mt, District, amenities as predictors

### Output Files

- madrid_rentals_clustered.csv (full dataset with cluster assignments)
- cluster_profiles.csv (comprehensive profiles)
- cluster_evaluation_metrics.csv (k selection data)
- scaler.pkl (StandardScaler for reproducibility)
- kmeans_model.pkl (trained model)
- All visualizations (HTML format)

---

## Key Decisions Summary

| Decision Point | Choice | Rationale |
|----------------|--------|-----------|
| **Bedrooms vs Sq.Mt** | Bedrooms | 9.7% better silhouette, aligns with search behavior |
| **Include amenities?** | No | Create dominated clusters, better as profiling |
| **Optimal k** | 4 | Elbow + silhouette + business interpretability |
| **VIF threshold** | < 10 | Professor guidance, industry standard |
| **Standardization** | Z-score (StandardScaler) | Equal variable weighting in distance |
| **Random state** | 42 | Reproducibility |

---

## Phase 2 Preparation

**Reserved variables for regression:**

- Sq.Mt (quantify space premium)
- District (one-hot encoded, quantify location premium)
- Elevator, Outer, Penthouse, Cottage (quantify amenity values)

**Target variable:** Rent

**Approach:** Build separate regression models per cluster or combined model with cluster dummies

---

## Workflow Execution Time

- Stage 1: 30 min (exploration)
- Stage 2: 20 min (cleaning)
- Stage 3: 30 min (VIF analysis)
- Stage 3.5: 15 min (k selection)
- Stage 4: 45 min (scenario testing)
- Stage 5: 20 min (quality check)
- Stage 6: 30 min (profiling)
- Stage 7: 30 min (documentation)

**Total:** ~3.5 hours for complete clustering analysis
