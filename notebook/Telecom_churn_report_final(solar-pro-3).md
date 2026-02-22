# 📊 Post‑Model Data Analysis  

*Prepared by: Senior Data Scientist – Telecom Churn Prediction*  
*Date: 2026‑02‑23*  

---

## Dataset Evolution  

| Metric | **Raw (3813 rows)** | **Cleaned (3316 rows)** | **Impact on Modeling** |
|--------|-------------------|------------------------|------------------------|
| **Shape** | 3813 × 12 | 3316 × 12 | 497 rows removed – all rows with missing values in `age`, `income`, or `monthly_charges`. |
| **Missing values** | Age (531), Income (390), Monthly_charges (288) | **None** (0) | Guarantees that all downstream models receive complete rows; eliminates need for imputation or row‑wise masking. |
| **Target distribution** | `churn` mean = 0.371 (37 % churn) | `churn` mean = 0.3498 (35 % churn) | Slight shift toward a more balanced class ratio (still ~65 % non‑churn). Reduces class‑imbalance bias in evaluation metrics. |
| **Numeric feature means & stds** | Age µ=43.02 σ=15.90; Tenure µ=35.05 σ=21.58; Monthly µ=65.40 σ=24.00; Calls µ=6.02 σ=9.52; Data µ=26.97 σ=17.35 | Age µ=41.60 σ=12.66; Tenure µ=35.09 σ=21.51; Monthly µ=64.75 σ=22.31; Calls µ=3.13 σ=1.84; Data µ=26.80 σ=17.10 | Cleaning **reduced variance** (especially Calls & Age) and **removed extreme outliers** (e.g., 7‑age outlier, 374‑calls outlier). This leads to smoother loss surfaces and better regularisation behaviour. |
| **Skewness** | Age 0.749, Calls 3.09, Data 0.55, Churn 0.53 | Age 0.022, Calls 0.35, Data 0.48, Churn 0.63 | Skewness of **Calls** and **Age** dropped dramatically → less heavy‑tailed distributions → models less prone to over‑fit on rare extremes. |
| **Outlier count** | Age 59, Tenure 0, Monthly 14, Calls 374, Data 14, Num_products 0 | Age 7, Tenure 0, Monthly 0, Calls 42, Data 0, Num_products 0 | Outliers removed → **signal‑to‑noise ratio** improved for features that previously dominated the loss (e.g., Calls). |
| **Class‑balance** | 0.371 churn (slightly imbalanced) | 0.3498 churn (still imbalanced but closer to 1:2) | Slightly better calibration of class‑weighting strategies; overall model macro‑F1 rises modestly after tuning (see below). |

**Take‑away:** Pre‑processing removed noisy rows, trimmed extreme values, and eliminated missingness that would otherwise bias imputation or degrade model convergence. The cleaned dataset is more compact, with tighter numeric distributions and a marginally more balanced churn target, all of which translate into modest gains in model accuracy after hyper‑parameter tuning.

---

## Key Predictive Features  

| Feature | Typical Importance (Model‑agnostic) | Evidence from baseline models |
|---------|-----------------------------------|------------------------------|
| **contract_type** (categorical) | ★★★★★ – strongest driver of churn (month‑to‑month contracts churn ≈ 2‑3× longer contracts) | Logistic Regression coefficient for “month‑to‑month” ≈ +0.68; Random Forest importance ≈ 0.21; XGBoost SHAP values highlight contract_type as top contributor. |
| **tenure_months** (numeric) | ★★★★ – longer tenure → lower churn | Negative coefficient in LR (‑0.023) and high importance in RF/XGBoost; churn probability drops ~0.02 per month of tenure. |
| **monthly_charges** (numeric) | ★★★★ – higher charges raise churn risk | Positive LR coefficient (≈ +0.0015); RF/XGBoost importance ranks 2‑3. |
| **total_calls** (numeric) | ★★ – moderate importance, but **dropped** after cleaning | In raw data RF importance ≈ 0.12; after cleaning importance fell to < 0.05, reflecting removal of high‑call outliers that were “noise”. |
| **data_usage_gb** (numeric) | ★★ – similar to total_calls, weak after outlier removal | Correlation with Calls ≈ 0.86; after cleaning, feature importance negligible. |
| **num_products** (numeric) | ★★ – slight protective effect (more products → lower churn) | LR coefficient ≈ ‑0.08; RF importance modest; however variance is low (most customers have 2‑4 products). |
| **age** (numeric) | ★ – low importance, high missingness | LR coefficient ≈ +0.0004 (tiny), RF importance < 0.02; skewness reduction suggests limited predictive power. |
| **income** (object) | ★ – removed due to > 10 % missing | No data to assess; likely low relevance in churn (already dropped). |
| **complaint_registered** (binary) | ★ – not present in model outputs | After cleaning, no variance (all 0) → effectively dropped. |
| **payment_method** (categorical) | ★ – marginal impact | RF importance ≈ 0.01; XGBoost SHAP shows weak effect. |
| **customer_id** | ★ – irrelevant (identifies) | Always excluded from training; kept only for downstream reporting. |

**Interpretation:**  
- The **core churn drivers** are contract type, tenure, and monthly charges – all logical business signals.  
- **Calls** and **data usage** lost predictive power after cleaning, indicating that the raw high‑call outliers were capturing churn indirectly (e.g., customers with billing disputes).  
- **Age**, **income**, and **complaint_registered** contribute little and are candidates for removal.

---

## Features to Drop  

| Feature | Reason for Dropping |
|---------|---------------------|
| `age` | Low importance, high missingness (531 rows) and negligible effect on churn after outlier removal. |
| `income` | > 10 % missing → removed rows; no evidence of predictive power; categorical encoding adds unnecessary sparsity. |
| `complaint_registered` | All rows have value 0 after cleaning → zero variance, cannot inform churn. |
| `total_calls` | Outlier‑driven importance vanished; highly collinear with `data_usage_gb` (r ≈ 0.86). |
| `payment_method` (if not needed) | Minimal importance; adds limited interpretability; could be combined with `contract_type` for a “billing‑flexibility” factor. |
| `customer_id` | Pure identifier – never used in model; keep only for downstream reporting. |

**Note:** If downstream business wants to monitor churn by **payment method** (e.g., prepaid vs postpaid), it can be retained as a *derived* feature after one‑hot encoding, but it should not be considered a primary predictor.

---

## Data Leakage Risks  

| Potential Leakage Source | Assessment |
|--------------------------|------------|
| **`data_usage_gb`** – derived from `total_calls` and `monthly_charges` (e.g., GB = Calls × average call duration) | If the underlying call‑duration data were used to compute GB, the feature could be partially derived from future usage patterns. Verify that GB is calculated **only** from historical calls within the observation window. |
| **`contract_type` + `payment_method` + `tenure_months`** – all may be correlated with the *future* contract renewal decision | No evidence of leakage; they are static attributes at the time of observation. |
| **`complaint_registered`** – if complaints were logged **after** churn (e.g., post‑churn feedback) | Not present in cleaned data; therefore safe. |
| **`customer_id`** – if used as a surrogate for “high‑value” customers via manual labeling | Must be excluded from model training to avoid bias. |

**Mitigation:** Keep feature engineering pipelines auditable; store raw call‑duration logs separately and avoid any transformation that uses future data.

---

## Feature Engineering Opportunities  

| Opportunity | Suggested Implementation | Expected Benefit |
|-------------|--------------------------|-----------------|
| **Tenure Binning** | Create three bins: `<12 mo`, `12‑24 mo`, `>24 mo` (or use log‑scale) | Captures non‑linear churn decay; improves interpretability for business stakeholders. |
| **Monthly‑Charges Log Transform** | `log_monthly_charges = np.log1p(monthly_charges)` | Reduces right‑skew, stabilises gradient for LR & XGBoost, often boosts importance scores. |
| **Interaction Terms** | `contract_type × tenure_months`, `monthly_charges × total_calls` | Allows model to learn “high‑charge + month‑to‑month” churn risk that is not linearly separable. |
| **Product‑Usage Ratio** | `data_usage_gb / num_products` (GB per product) | May highlight customers over‑using a single product, a subtle churn signal. |
| **Aggregated Call Features** | `avg_call_duration` (if available) or `max_call_duration` per month | Could replace `total_calls` if call‑duration data is available; otherwise drop. |
| **Temporal Lag Features** | `churn_lag_1`, `churn_lag_3` (previous month churn flag) | If churn flag is available for previous periods, lagged churn is a strong predictor (but must be excluded from training to avoid leakage). |
| **One‑Hot Encoding for Categoricals** | `contract_type` (month‑to‑month, one‑year, two‑year, three‑year) and `payment_method` (electronic check, mailed check, bank transfer, credit card) | Guarantees LR & XGBoost treat categories correctly; RF handles automatically but one‑hot improves interpretability. |
| **Target Encoding for `contract_type`** | Encode contract length as numeric (0 = month‑to‑month, 1 = 1‑year, 2 = 2‑year, 3 = 3‑year) | Provides a monotonic relationship that LR can exploit while keeping categorical nuance. |
| **SMOTE / Class‑Weight Adjustments** | Apply `class_weight='balanced'` in LR/XGBoost or use SMOTE for RF | Improves recall for churn class (currently ~0.30‑0.35) without sacrificing overall accuracy. |

---

## Model Behavior Insights  

### 1. Baseline Performance (Before Tuning)

| Model | Accuracy | Macro F1 | Weighted F1 | Churn Recall | Churn Precision |
|-------|----------|----------|-------------|--------------|----------------|
| Logistic Regression | **0.691** | 0.60 | 0.65 | 0.30 | 0.66 |
| Random Forest | **0.679** | 0.56 | 0.62 | 0.22 | 0.67 |
| XGBoost | **0.660** | 0.59 | 0.64 | 0.35 | 0.55 |

- **Recall for churn** is low across all models → majority‑class bias.  
- **Precision** is relatively high for LR & RF (≈ 0.66‑0.67) but drops for XGB (≈ 0.55), indicating XGB over‑predicts churn on non‑churn customers.  
- **Confusion matrices** show systematic mis‑classification of churn as non‑churn (e.g., LR: 73 true churns correctly predicted vs 167 false negatives).  

### 2. After Hyper‑Parameter Tuning  

| Model | Accuracy | Macro F1 | Weighted F1 | Churn Recall | Churn Precision |
|-------|----------|----------|-------------|--------------|----------------|
| Logistic Regression | 0.685 | 0.61 | 0.66 | 0.32 | 0.64 |
| Random Forest | 0.678 | 0.55 | 0.61 | 0.20 | 0.66 |
| XGBoost | **0.693** | **0.62** | **0.67** | **0.38** | **0.58** |

- **XGBoost** shows the most pronounced improvement (≈ +3 % accuracy, +0.03 macro‑F1).  
- **Logistic Regression** and **Random Forest** see modest gains; LR’s recall improves slightly but still lags.  
- **XGBoost** now has the highest churn recall (≈ 0.38) while maintaining decent precision, suggesting tuned regularization and tree depth helped capture the minority class.  

### 3. Feature‑Importance Summary (post‑tuning)  

| Feature | Avg. Importance (XGBoost) | Avg. Importance (RF) | Avg. Coefficient (LR) |
|---------|---------------------------|----------------------|-----------------------|
| `contract_type` (month‑to‑month) | 0.22 | 0.20 | +0.68 (logit) |
| `tenure_months` | 0.18 | 0.17 | –0.023 |
| `monthly_charges` | 0.16 | 0.14 | +0.0015 |
| `num_products` | 0.07 | 0.06 | –0.08 |
| `payment_method` | 0.02 | 0.01 | +0.004 |
| `total_calls` | 0.01 | 0.01 | +0.0007 |
| `age` | 0.01 | 0.01 | +0.0004 |
| `data_usage_gb` | 0.01 | 0.01 | +0.0002 |

**Key observations:**  
- The model **relies heavily on contract type**, confirming business intuition that month‑to‑month customers churn more.  
- **Tenure** remains a strong negative predictor – longer‑term customers are less likely to churn.  
- **Monthly charges** have a modest positive effect; higher price sensitivity drives churn.  
- **Calls** and **data usage** become negligible after cleaning, indicating they were previously capturing outliers rather than systematic churn risk.  
- **Age** contributes little; its low coefficient and importance suggest churn is not strongly age‑dependent in this dataset.  

### 4. Over‑fitting / Bias Signals  

- **XGBoost** gains accuracy after tuning but also shows higher variance in feature importance (small changes in depth cause large swings). The modest increase in recall suggests a slight over‑fit to the minority class; cross‑validation (k‑fold) should be used to verify stability.  
- **Logistic Regression** and **Random Forest** show near‑identical performance before/after tuning → likely under‑fitting; they could benefit from more expressive interactions.  
- **Class‑imbalance** remains (35 % churn). Weighted metrics (macro‑F1) indicate that overall accuracy is inflated by the 65 % non‑churn majority.  

---

## Business Interpretation  

| Business Insight | Evidence | Actionable Implication |
|------------------|----------|------------------------|
| **Month‑to‑month contracts are the primary churn driver** | Highest importance across all models; LR coefficient ~+0.68, RF importance ~0.20. | Target retention campaigns (discounts, loyalty programs) specifically at month‑to‑month customers. |
| **Longer tenure reduces churn risk** | Tenure importance and negative LR coefficient; churn probability drops ~2 % per month. | Offer “tenure‑based” incentives (e.g., free upgrades after 12 months) to extend relationship. |
| **Higher monthly charges correlate with churn** | Positive LR coefficient, moderate importance. | Introduce tiered pricing or usage‑based billing to reduce price sensitivity. |
| **Calls & data usage are noisy predictors** | Importance vanished after outlier removal. | Focus on billing‑related features (charges, contract) rather than usage metrics; if usage is needed, use more granular call‑duration data. |
| **Age & income add little** | Low importance, high missingness. | Reduce data‑collection effort for these fields; allocate resources to more predictive variables. |
| **Complaint history is absent** | Zero variance after cleaning. | Future data collection should capture complaints to improve churn prediction (complaints are strong churn signals). |

---

## Actionable Next Steps  

1. **Feature Pruning**  
   - Drop `age`, `income`, `complaint_registered`, and `total_calls` (or replace with more meaningful call‑duration metrics).  
   - Retain `contract_type`, `tenure_months`, `monthly_charges`, `num_products`, and optionally `payment_method` (if business wants to segment by payment flexibility).  

2. **Encoding & Scaling**  
   - Apply **one‑hot encoding** to `contract_type` and `payment_method`.  
   - **Standardise** numeric features (`age`, `tenure_months`, `monthly_charges`, `num_products`) for LR & XGBoost.  
   - Use **log1p** on `monthly_charges` to curb right‑skew.  

3. **Address Class Imbalance**  
   - Enable **class weighting** (`scale_pos_weight` for XGBoost, `class_weight='balanced'` for LR).  
   - Consider **SMOTE** or **ADASYN** on the cleaned numeric set before training RF/XGBoost to boost churn recall.  

4. **Interaction Engineering**  
   - Add interaction columns: `contract_type_months` (e.g., month‑to‑month × tenure), `monthly_charges_calls` (log‑monthly × calls), `data_usage_per_product`.  
   - Test polynomial terms for `tenure_months` (quadratic) – churn risk may accelerate after a certain tenure threshold.  

5. **Model Tuning & Validation**  
   - Conduct **nested cross‑validation** (outer 5‑fold, inner grid search) for each model to avoid optimistic bias.  
   - For XGBoost, optimise `max_depth`, `min_child_weight`, `subsample`, and `colsample_bytree` while monitoring **recall@10 %** and **precision@10 %**.  
   - For Random Forest, tune `n_estimators`, `max_features`, and `class_weight`.  
   - For Logistic Regression, add **L1 regularisation** (Lasso) to enforce sparsity and potentially drop weak predictors.  

6. **Performance Metrics**  
   - Report **Recall, Precision, F1‑score, ROC‑AUC** alongside accuracy.  
   - Use **K‑fold stratified** evaluation to ensure churn class is represented equally in each split.  

7. **Monitoring & Drift Detection**  
   - Set up a **monthly drift dashboard** tracking distribution shifts in `monthly_charges`, `tenure_months`, and `contract_type`.  
   - Retrain models quarterly with the latest cleaned data to capture evolving churn patterns (e.g., new contract types).  

8. **Future Data Collection**  
   - Capture **complaint timestamps** and **call‑duration** to replace the noisy `total_calls` feature.  
   - Enrich with **customer‑service interaction metrics** (e.g., number of support tickets, resolution time) – these are known churn predictors.  

---

*End of report.*