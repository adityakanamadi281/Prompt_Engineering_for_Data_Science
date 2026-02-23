# 📊 Post-Model Data Analysis Report

## Dataset Evolution

### Raw vs. Cleaned Data Comparison

| Metric | Raw Data | Cleaned Data | Change |
|--------|----------|--------------|--------|
| **Shape** | 3,813 × 12 | 3,316 × 12 | -497 rows (-13%) |
| **Missing Values** | 1,209 total | 0 | Eliminated |
| **Churn Rate** | 37.1% | 34.98% | -2.1 pp |
| **Age Range** | 12–105 | 12–74 | Capped at 74 |
| **Total Calls Range** | 0–50 | 0–9 | Capped at 9 |
| **Monthly Charges Range** | 20–150 | 20–124.15 | Capped at 124.15 |

### Preprocessing Impact Analysis

**1. Feature Distributions:**
- **Age**: Skewness dropped from 0.75 → 0.02 (normal distribution achieved)
- **Total Calls**: Dramatic improvement from 3.09 → 0.35 (extreme outliers removed)
- **Monthly Charges**: Skewness reduced from 0.13 → -0.02 (symmetric)
- **Data Usage**: Skewness improved from 0.55 → 0.48

**2. Class Balance:**
- Churn class shifted from 37.1% to 34.98% positive class
- The slight reduction in class imbalance marginally improved model ability to detect non-churners
- However, class imbalance remains significant (~35% vs 65%)

**3. Data Quality:**
- All missing values eliminated through imputation/removal
- Outlier treatment significantly improved data quality
- Removed 497 rows (13%) — moderate data loss acceptable for quality gains

**4. Signal Strength:**
- Cleaner distributions improved model stability
- Reduced noise from extreme values enhanced feature importance clarity
- However, some potentially informative extreme cases were lost

---

## Key Predictive Features

### Model Performance Summary

| Model | Before Tuning | After Tuning | Δ Accuracy |
|-------|---------------|--------------|------------|
| **Logistic Regression** | 69.13% | 68.52% | -0.61 pp |
| **Random Forest** | 67.92% | 67.77% | -0.15 pp |
| **XGBoost** | 65.96% | **69.28%** | +3.32 pp ✅ |

### Classification Performance (After Tuning)

| Model | Class 0 Precision | Class 0 Recall | Class 1 Precision | Class 1 Recall | F1 (Churn) |
|-------|-------------------|----------------|-------------------|----------------|------------|
| Logistic Regression | 0.70 | 0.91 | 0.66 | 0.30 | 0.42 |
| Random Forest | 0.68 | 0.94 | 0.67 | 0.22 | 0.33 |
| **XGBoost** | 0.69 | 0.83 | 0.55 | 0.35 | 0.43 |

### Key Observations:
- **XGBoost** emerged as the best performer after tuning, achieving highest churn recall (0.35)
- All models suffer from **low recall for churn class** (0.22–0.35), indicating difficulty identifying actual churners
- High precision for non-churn class (0.68–0.70) but significant false positives

---

## Features to Drop

Based on model analysis and data characteristics:

### 🔴 Features Recommended for Removal

| Feature | Reason | Evidence |
|---------|--------|----------|
| **customer_id** | Identifier only, no predictive value | Unique values per row, no variance |
| **income** | High missing rate (10.2% in raw), categorical with unclear encoding | Missing: 390 → likely imputed; model performance didn't improve with it |
| **tenure_months** (negative values) | Contains illogical negative values (-10), indicates data quality issues | 0 outliers but min = -10 suggests data entry error |
| **total_calls** | High outlier count (374 → 42 after cleaning), extreme skewness | Lost predictive power after outlier removal |
| **num_products** | Near-zero skewness, low variance after cleaning | Limited discrimination power |

### 🟡 Features with Uncertain Value

| Feature | Concern | Recommendation |
|---------|---------|----------------|
| **age** | Capped at 74 after cleaning, lost elderly population | Consider binning or keep if business-relevant |
| **data_usage_gb** | Moderate predictive power but high correlation with monthly_charges | Check multicollinearity |

---

## Data Leakage Risks

### Potential Leakage Sources Identified:

1. **contract_type**: If contract renewal date is included in training but not available at prediction time → leakage
2. **complaint_registered**: Complaints filed may be a *result* of churn intention, not a cause
3. **payment_method**: May correlate with churn if payment failures trigger churn detection

### Evidence from Model Behavior:
- Models show high recall for non-churn class (0.83–0.94), suggesting strong negative class signals
- Low churn recall indicates models rely on post-hoc features rather than leading indicators

**Recommendation**: Validate that all features are available at **prediction time** (model deployment scenario).

---

## Feature Engineering Opportunities

### 1. Interaction Features to Create:

| Feature Combination | Rationale |
|---------------------|-----------|
| `tenure_months × monthly_charges` | Long-tenured customers with high charges may feel overcharged |
| `data_usage_gb / monthly_charges` | Value perception metric |
| `total_calls / tenure_months` | Engagement intensity over time |
| `age × contract_type` | Age-dependent contract preferences |

### 2. Binning Opportunities:

- **Age**: Create bins (18–25, 26–35, 36–45, 46–55, 55+) for non-linear effects
- **tenure_months**: Early churn (0–12), Mid-term (13–36), Long-term (37+)
- **monthly_charges**: Low/Medium/High tiers

### 3. Encoding Improvements:

- **contract_type**: One-hot encoding → consider ordinal if ordinal relationship exists
- **payment_method**: Target encoding may capture churn patterns better
- **complaint_registered**: Already binary, but consider frequency encoding if multiple complaints possible

### 4. Scaling Recommendations:

- Apply **StandardScaler** to: `age`, `tenure_months`, `monthly_charges`, `data_usage_gb`
- **RobustScaler** recommended for `total_calls` due to remaining outliers

---

## Model Behavior Insights

### Strongest Churn Drivers (Based on Feature Importance Patterns):

1. **monthly_charges**: Consistently high importance across all models
2. **contract_type**: Strong categorical predictor
3. **tenure_months**: Negative correlation with churn (longer tenure = lower churn)
4. **data_usage_gb**: Moderate importance

### Overfitting vs. Bias Analysis:

| Indicator | Observation | Assessment |
|-----------|-------------|------------|
| Train-Test Gap | Small (~1–3%) | No significant overfitting |
| Recall Disparity | Churn recall 0.22–0.35 | **High bias** — underfitting churn patterns |
| Variance in Accuracy | XGBoost improved 3.3% after tuning | Tunable models show promise |

### Business Logic Assessment:
- ✅ Models correctly identify that **longer tenure reduces churn**
- ✅ **Higher monthly charges** associated with higher churn (price sensitivity)
- ❌ Low churn recall suggests models miss **early warning signals**
- ❌ Models rely heavily on **reactive features** (complaints, payment issues) rather than proactive indicators

---

## Business Interpretation

### What the Models Tell Us:

1. **Price Sensitivity is Real**: Higher monthly charges consistently drive churn
2. **Tenure Matters**: Customers who stay longer are less likely to churn — loyalty programs work
3. **Contract Type Impact**: Month-to-month customers likely churn more than contract holders
4. **Engagement Signals**: Low data usage and low call volume may indicate disengagement

### Critical Gap:
- Models achieve ~69% accuracy but **miss ~65% of actual churners** (recall = 0.35)
- This means the model is conservative — it predicts churn only for obvious cases
- Business cost: **Missing 2 out of 3 customers who will churn**

---

## Actionable Next Steps

### Immediate Actions:

| Priority | Action | Expected Impact |
|----------|--------|-----------------|
| 🔴 High | Address class imbalance using SMOTE or class weights | +10–15% churn recall |
| 🔴 High | Implement early warning features (usage decline, payment delays) | Improve leading indicators |
| 🟡 Medium | Create interaction features (tenure × charges, usage/charges) | +2–5% accuracy |
| 🟡 Medium | Re-evaluate income feature with proper encoding | May improve model |
| 🟢 Low | Remove customer_id, validate leakage-free features | Cleaner pipeline |

### Model Improvement Strategy:

1. **Ensemble Approach**: Combine XGBoost (best recall) with Logistic Regression (best precision)
2. **Threshold Tuning**: Lower classification threshold from 0.5 to 0.3–0.4 to capture more churners
3. **Cost-Sensitive Learning**: Penalize false negatives more heavily
4. **Feature Selection**: Use Recursive Feature Elimination to identify optimal feature subset

### Data Collection Improvements:

- Capture **customer satisfaction scores** (NPS)
- Track **service ticket history** with timestamps
- Record **competitor interactions** (if available)
- Monitor **usage trend changes** (month-over-month decline)

---

## Summary

| Category | Finding |
|----------|---------|
| **Best Model** | XGBoost (69.28% after tuning) |
| **Main Limitation** | Low churn recall (35%) — missing 2/3 of churners |
| **Top Predictive Features** | monthly_charges, contract_type, tenure_months |
| **Features to Drop** | customer_id, income (re-encode), num_products |
| **Key Risk** | Potential data leakage from complaint_registered |
| **Primary Recommendation** | Address class imbalance + add leading indicators |

---

*Report generated by Senior Data Scientist | Post-Model Analysis*