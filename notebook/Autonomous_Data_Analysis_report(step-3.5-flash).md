# Titanic Dataset Basic Analysis Summary

## Dataset Overview
- **Shape**: 891 rows × 12 columns  
- **Target Variable**: `Survived` (binary: 0 = No, 1 = Yes)  
- **Survival Rate**: 38.4% (mean of `Survived`)  
- **Data Types**:  
- **Numerical**: `PassengerId`, `Pclass`, `Age`, `SibSp`, `Parch`, `Fare`  
- **Categorical**: `Name`, `Sex`, `Ticket`, `Cabin`, `Embarked`  

---

## Feature-by-Feature Analysis

### 1. **Numerical Features**
| Feature   | Missing | Outliers | Skewness | Key Observations |
|-----------|---------|----------|----------|------------------|
| `Age`     | 177 (19.9%) | 11       | 0.39 (moderate +) | Mean ≈ 29.7, median 28. Right-skewed. Missing values need imputation. Likely important (children/elderly survival patterns). |
| `Fare`    | 0       | 116      | 4.79 (high +) | Mean ≈ 32.2, but median 14.5. Extreme right-skew due to few high fares (e.g., first-class suites). May require transformation (log). |
| `SibSp`   | 0       | 46       | 3.70 (high +) | Mostly 0–1. Right-skewed; many traveled alone. Could be aggregated into `FamilySize`. |
| `Parch`   | 0       | 213      | 2.75 (high +) | Similar to `SibSp`; mostly 0. Right-skewed. Combine with `SibSp` for family size. |
| `Pclass`  | 0       | 0        | -0.63 (moderate -) | Ordinal (1=1st, 2=2nd, 3=3rd). Mean 2.3, median 3 → more third-class passengers. Strong suspected predictor. |
| `PassengerId` | 0   | 0        | 0.0      | Pure identifier. **No predictive value**. |

### 2. **Categorical Features**
| Feature   | Missing | Unique Values (est.) | Key Observations |
|-----------|---------|----------------------|------------------|
| `Sex`     | 0       | 2 (male/female)      | Likely strong predictor (historical "women and children first"). |
| `Embarked`| 2 (0.2%)| 3 (C/Q/S)           | Only 2 missing; easy imputation. May correlate with `Pclass`/port. |
| `Cabin`   | 687 (77.2%) | High (many rooms) | **Too many missing** (mostly first-class). Unreliable; drop or engineer "HasCabin" binary. |
| `Ticket`  | 0       | High (681 unique)    | High cardinality, unclear pattern (mix of numbers/letters). Likely noise. |
| `Name`    | 0       | 891 (all unique)     | All unique, but contains **titles** (Mr/Mrs/Miss/Master) which could be engineered. Raw `Name` not useful. |

---

## Initial Observations & Relationships
1. **Class & Survival**: `Pclass` likely inversely related to survival (1st class higher survival).  
2. **Gender & Survival**: `Sex` expected to show females had much higher survival rates.  
3. **Age & Survival**: Children (`Age` < 12) possibly had higher survival; elderly lower.  
4. **Family & Survival**: `SibSp`/`Parch` → create `FamilySize` (alone vs. with family) might reveal patterns (e.g., medium families survived better?).  
5. **Fare & Class**: `Fare` strongly correlated with `Pclass`; may be redundant if `Pclass` is used.  
6. **Embarked & Class**: Port `C` (Cherbourg) had more first-class passengers; may indirectly affect survival.  

---

## Feature Importance & Recommendations

### ✅ **Features to KEEP (with possible engineering)**
- `Pclass` (ordinal, strong predictor)  
- `Sex` (categorical, strong predictor)  
- `Age` (impute missing; likely non-linear relationship with survival)  
- `Embarked` (impute 2 missing; may have indirect effect)  
- `SibSp` + `Parch` → engineer `FamilySize` or `IsAlone`  
- `Fare` (apply log transform due to skew; may proxy wealth/class)  

### ❌ **Features to DROP**
1. **`PassengerId`** – Pure identifier, zero predictive power.  
2. **`Name`** – All unique; raw text not useful. *Optional*: Extract titles (Mr/Mrs/Miss/Master) as new feature, then drop raw `Name`.  
3. **`Ticket`** – High cardinality, no clear signal. Likely random or system noise.  
4. **`Cabin`** – 77% missing. Even if engineered as `HasCabin`, it mostly reflects `Pclass` (first-class only). High risk of overfitting.  

---

## Key Insights for Modeling
- **Class and gender dominate**: Expect `Pclass` and `Sex` to be top features.  
- **Handle missing `Age` carefully**: Impute with median/regression (consider `Pclass`/`Sex` groups).  
- **Address skewness**: Apply log to `Fare`; consider binning `Age` or using splines.  
- **Engineer family features**: Combine `SibSp` and `Parch` into `FamilySize` (`SibSp + Parch + 1`).  
- **Categorical encoding**: One-hot encode `Sex`, `Embarked`; treat `Pclass` as ordinal or categorical.  
- **Drop redundant/noisy features**: As listed above to reduce dimensionality and noise.  

---

## Next Steps Suggested
1. Impute missing `Age` (e.g., median by `Pclass`/`Sex`).  
2. Create `FamilySize` from `SibSp` + `Parch`.  
3. Extract `Title` from `Name` (then drop `Name`).  
4. Drop `PassengerId`, `Ticket`, `Cabin`.  
5. Transform `Fare` (log1p) to reduce skew.  
6. Explore interactions: e.g., `Pclass` × `Sex`, `Age` × `Pclass`.  

This streamlined feature set should improve model interpretability and performance while reducing noise from high-missing or high-cardinality features.