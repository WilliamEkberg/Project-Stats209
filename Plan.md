# Stats209: Causal Effect of Instant Book on Airbnb Ratings

## Project Overview
Estimate the ATE of `instant_bookable` on `review_scores_rating` for SF Airbnb listings using AIPW with cross-fitting.

---

## Phase 1: Address Instructor Feedback

### 1.1 Resolve Estimand Problem (Feedback #1)
**Issue:** Single snapshot cannot support "next-period" causal estimand.

**Solution:** Reframe estimand as:
> "Effect of enabling Instant Book on a listing's current average rating, interpreted as summarizing past guest experiences under that booking policy."

**Alternative (if time permits):** Download two snapshots (3-6 months apart) to construct panel with treatment at t, outcome at t+1.

### 1.2 Justify Pre-Treatment Covariates (Feedback #2)
Create justification for each covariate:

| Covariate | Pre-treatment? | Justification |
|-----------|----------------|---------------|
| `neighbourhood` | Yes | Fixed at listing creation |
| `room_type` | Yes | Structural property feature |
| `accommodates`, `bedrooms` | Yes | Physical property features |
| `bathrooms` | Yes | Physical property feature |
| `amenities_count` | Yes | Largely fixed at listing creation |
| `price` | Likely Yes | Usually set before IB decision |
| `listing_age` | Yes | Determined by first_review date |
| `host_is_superhost` | **Drop** | Could be affected by IB (review volume) |
| `host_response_rate` | **Drop** | May change with IB (fewer pre-booking messages) |
| `host_acceptance_rate` | **Drop** | Directly affected by IB |
| `calculated_host_listings_count` | Yes | Host portfolio size, pre-dates IB decision |

**Action:** Run analysis with and without questionable covariates as sensitivity check.

### 1.3 Add Literature Context (Feedback #4)
Write 1 paragraph noting:
- No prior causal evidence on Instant Book → ratings specifically
- Existing descriptive work on Airbnb ratings (Zervas et al., Fradkin et al.)
- Our contribution: modern causal methods (AIPW) to fill this gap

### 1.4 Practical Decision Rule (Feedback #3)
Plan final presentation format:
> "Hosts enabling Instant Book can expect rating change of X ± Y. This is robust unless unmeasured confounding explains >Z% of outcome variance (partial R²) or Γ > W (Rosenbaum bounds)."

---

## Phase 2: Data Acquisition & Preprocessing

### 2.1 Download Data
- [ ] Download SF `listings.csv.gz` from [InsideAirbnb](http://insideairbnb.com/get-the-data/)
- [ ] Save to `data/raw/listings.csv`

### 2.2 Data Preprocessing (`scripts/01_data_prep.py`)
- [ ] Load CSV with pandas
- [ ] Parse `bathrooms_text` → numeric (regex)
- [ ] Compute `listing_age_months` from `first_review` and `calendar_last_scraped`
- [ ] Parse `amenities` JSON → count
- [ ] Convert `instant_bookable` to binary (t/f → 1/0)
- [ ] Clean `price` (remove $, commas → float)
- [ ] Handle missing values (document strategy: drop vs impute)
- [ ] Filter to rows with valid `review_scores_rating`
- [ ] Create neighbourhood dummy encoding
- [ ] Save to `data/processed/listings_clean.parquet`

### 2.3 Define Final Variables
**Treatment:** `instant_bookable` (binary)

**Outcome:** `review_scores_rating` (continuous, 0-5 scale)

**Covariates (baseline set):**
- `neighbourhood` (categorical → dummies)
- `room_type` (categorical)
- `accommodates` (numeric)
- `bedrooms` (numeric)
- `bathrooms` (numeric)
- `amenities_count` (numeric)
- `price` (numeric, log-transformed)
- `listing_age_months` (numeric)
- `calculated_host_listings_count` (numeric)

---

## Phase 3: Exploratory Analysis & Propensity Scores

### 3.1 Descriptive Statistics (`scripts/02_eda.py`)
- [ ] Sample sizes: n_treated, n_control
- [ ] Outcome distribution by treatment (histograms, means)
- [ ] Covariate summary table (mean/SD by treatment group)
- [ ] Save tables to `output/tables/`

### 3.2 Propensity Score Estimation (`scripts/03_propensity.py`)
- [ ] Fit logistic regression: `instant_bookable ~ covariates`
- [ ] Predict propensity scores ê(X)
- [ ] Plot propensity score distributions (overlap check)
- [ ] Compute Standardized Mean Differences (target: |SMD| < 0.1)
- [ ] Report effective sample size
- [ ] Save figures to `output/figures/`

---

## Phase 4: AIPW Estimation

### 4.1 Cross-Fitted AIPW (`scripts/04_aipw.py`)
- [ ] Split data into K=5 folds
- [ ] For each fold: fit propensity and outcome models on other folds, predict on held-out
- [ ] Compute AIPW estimator:
  ```
  τ̂ = (1/n) Σ [ μ̂₁(Xᵢ) - μ̂₀(Xᵢ) + Tᵢ(Yᵢ - μ̂₁(Xᵢ))/ê(Xᵢ) - (1-Tᵢ)(Yᵢ - μ̂₀(Xᵢ))/(1-ê(Xᵢ)) ]
  ```
- [ ] Compute neighbourhood-clustered standard errors
- [ ] Bootstrap 95% CIs (1000 replicates)

### 4.2 Alternative Estimators (Robustness)
- [ ] Overlap-weighted regression adjustment
- [ ] Caliper-matched propensity score matching
- [ ] Compare estimates across methods

**Libraries:** `econml`, `sklearn`, `statsmodels`

---

## Phase 5: Sensitivity Analysis

### 5.1 Rosenbaum Bounds (`scripts/05_sensitivity.py`)
- [ ] Compute critical Γ where p > 0.05
- [ ] Interpret: "Result robust to hidden bias of magnitude Γ"

### 5.2 Partial R² / Omitted Variable Bias
- [ ] Use `sensemakr`-style analysis (Python: manual implementation or `pysensemakr`)
- [ ] Report: confounding strength needed to nullify effect

---

## Phase 6: Final Report

### 6.1 Write-up
- [ ] Update estimand language per feedback
- [ ] Add covariate justification table
- [ ] Add literature paragraph
- [ ] Present ATE with practical decision rule interpretation
- [ ] Include all diagnostics (overlap, balance, sensitivity)

---

## File Structure
```
Project Stats209/
├── data/
│   ├── raw/
│   │   └── listings.csv
│   └── processed/
│       └── listings_clean.parquet
├── scripts/
│   ├── 01_data_prep.py
│   ├── 02_eda.py
│   ├── 03_propensity.py
│   ├── 04_aipw.py
│   └── 05_sensitivity.py
├── output/
│   ├── figures/
│   └── tables/
├── report/
│   └── stats209_final.tex
├── requirements.txt
└── Plan.md
```

## Python Dependencies
```
pandas>=2.0
numpy
scikit-learn
statsmodels
econml
matplotlib
seaborn
pyarrow
```

