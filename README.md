
# Breast Cancer Survival Analysis
### SQL + Python Data Analysis Portfolio Project


## Project Overview

This project analyzes survival outcomes in breast cancer patients using a combination of **SQL (SQLite)** and **Python** to identify which clinical factors most strongly predict patient survival. The dataset contains 4,024 real patient records from the SEER (Surveillance, Epidemiology, and End Results) database.

**Tools used:** DB Browser for SQLite · Python · Pandas · Scikit-learn · SciPy · Jupyter Notebook


## Research Question

> *Which clinical and biological factors independently predict breast cancer survival outcomes, and are these differences statistically significant?*


## Dataset

- **Source:** [SEER Breast Cancer Dataset — Kaggle](https://www.kaggle.com/datasets/reihanenamdari/breast-cancer)
- **Records:** 4,024 patients
- **Key columns:** Age, Race, Marital Status, T/N/6th Stage, Tumor Size, Estrogen Status, Progesterone Status, Regional Nodes Examined, Survival Months, Status (Alive/Dead)


## Database Schema

The raw CSV was imported into SQLite and split into 3 relational tables:

```
patients       — age, race, stage, tumor size, grade
treatments     — estrogen status, progesterone status, nodes examined/positive
outcomes       — survival months, survived (1/0)
```

All three tables join on `patient_id`.

---

## SQL Analysis — Key Queries & Findings

### 1. Survival by Estrogen Receptor Status
```sql
SELECT t.estrogen_status,
       COUNT(*) AS total_patients,
       ROUND(AVG(o.survived) * 100, 1) AS survival_rate_pct,
       ROUND(AVG(o.survival_months), 1) AS avg_survival_months
FROM treatments t
JOIN outcomes o ON t.patient_id = o.patient_id
GROUP BY t.estrogen_status
ORDER BY survival_rate_pct DESC;
```

| Estrogen Status | Patients | Survival Rate | Avg Survival Months |
|----------------|----------|---------------|-------------------|
| Positive | 3,755 | 86.5% | 72.1 |
| Negative | 269 | 59.9% | 60.3 |

### 2. Survival by Cancer Stage
```sql
SELECT p.stage,
       COUNT(*) AS total_patients,
       ROUND(AVG(o.survived) * 100, 1) AS survival_rate_pct,
       ROUND(AVG(o.survival_months), 1) AS avg_survival_months
FROM patients p
JOIN outcomes o ON p.patient_id = o.patient_id
GROUP BY p.stage
ORDER BY survival_rate_pct DESC;
```

| Stage | Patients | Survival Rate | Avg Survival Months |
|-------|----------|---------------|-------------------|
| IIA | 1,305 | 92.6% | 74.4 |
| IIB | 1,130 | 88.1% | 72.2 |
| IIIA | 1,050 | 82.5% | 70.2 |
| IIIB | 67 | 70.1% | 69.4 |
| IIIC | 472 | 61.7% | 63.2 |

### 3. Hormone Receptor Combination Analysis
| ER Status | PR Status | Patients | Survival Rate |
|-----------|-----------|----------|---------------|
| Positive | Positive | 3,299 | 87.7% |
| Negative | Positive | 27 | 77.8% |
| Positive | Negative | 456 | 77.6% |
| Negative | Negative | 242 | 57.9% |

### 4. Stage × Estrogen Status Interaction
| Stage | ER+ Survival | ER− Survival | Gap |
|-------|-------------|-------------|-----|
| IIA | 92.9% | 86.4% | 6.5% |
| IIB | 89.3% | 66.7% | 22.6% |
| IIIA | 84.3% | 58.7% | 25.6% |
| IIIB | 76.8% | 36.4% | 40.4% |
| IIIC | 65.9% | 32.8% | **33.1%** |

> The gap between ER+ and ER− patients widens dramatically as cancer advances — at Stage IIIC, ER-negative patients have only a 32.8% survival rate.

### 5. Survival by Age Group
| Age Group | Patients | Survival Rate |
|-----------|----------|---------------|
| 40–49 | 1,124 | 87.9% |
| 50–59 | 1,390 | 86.4% |
| 60–69 | 1,280 | 80.9% |
| Under 40 | 230 | 79.6% |

> Patients under 40 have the lowest survival rate despite being youngest — likely due to more aggressive tumor types at younger ages.

### 6. Survival by Tumor Size
| Tumor Size | Patients | Survival Rate |
|------------|----------|---------------|
| Small (≤20mm) | 1,620 | 89.9% |
| Medium (21–50mm) | 1,824 | 82.6% |
| Large (>50mm) | 580 | 76.7% |

---

## Python Statistical Analysis

### Chi-Square Test — Statistical Significance

```python
from scipy.stats import chi2_contingency

contingency = pd.crosstab(df['Estrogen Status'], df['Status'])
chi2_stat, p_value, dof, expected = chi2_contingency(contingency)
```

| Factor | Chi-Square Statistic | P-Value | Result |
|--------|---------------------|---------|--------|
| Estrogen Status | 135.16 | 0.000000 | **Significant** |
| Progesterone Status | — | 0.000000 | **Significant** |
| Cancer Stage | — | 0.000000 | **Significant** |
| Differentiation Grade | — | 0.000000 | **Significant** |

All four factors are statistically significant at p < 0.0001 — the observed survival differences are not due to chance.

### Logistic Regression — Multivariate Analysis

```python
from sklearn.linear_model import LogisticRegression

features = ['Age', 'Tumor Size', 'Estrogen Status',
            'Progesterone Status', '6th Stage']
model = LogisticRegression(max_iter=1000)
model.fit(X, y)
```

| Feature | Coefficient | Odds Ratio | Effect |
|---------|-------------|------------|--------|
| Estrogen Status | +0.861 | **2.364** | Helps survival |
| Progesterone Status | +0.629 | **1.875** | Helps survival |
| Tumor Size | −0.003 | 0.997 | Slightly hurts |
| Age | −0.023 | 0.978 | Slightly hurts |
| Cancer Stage | −0.468 | **0.626** | Hurts survival |

**Model Accuracy: 85.5%**

> An odds ratio of 2.36 for Estrogen Status means ER-positive patients are **2.36 times more likely to survive** even after controlling for age, stage, and tumor size.

---

## Conclusions

1. **Estrogen receptor status is the strongest independent predictor of survival.** ER-positive patients are 2.36× more likely to survive, regardless of age, stage, or tumor size. This finding is statistically significant (χ² = 135.16, p < 0.0001).

2. **The combination of ER and PR status matters.** Double-positive patients (ER+ PR+) have an 87.7% survival rate compared to 57.9% for double-negative patients — a 30-point gap.

3. **Cancer stage is the strongest negative predictor.** Survival drops from 92.6% at Stage IIA to 61.7% at Stage IIIC. ER status amplifies this — Stage IIIC ER-negative patients have only a 32.8% survival rate.

4. **Younger patients (<40) have unexpectedly poor outcomes.** Despite being the youngest cohort, under-40 patients have the lowest survival rate (79.6%), likely due to more aggressive tumor biology at younger ages.

5. **Early detection of hormone receptor status is critical.** These findings suggest hormone receptor testing at the time of diagnosis should be prioritized — it is the single most important biological marker for predicting outcomes.

---

## Limitations

- The dataset does not include an explicit treatment type column (chemotherapy, surgery, radiation, hormone therapy). This analysis therefore measures **prognostic factors** rather than true treatment effectiveness.
- No time-based survival analysis (e.g. Kaplan-Meier curves) was performed — average survival months were used as a proxy.
- The logistic regression model did not include interaction terms between variables, which could improve accuracy.
- All data comes from a single dataset — findings should be validated on an independent dataset before drawing clinical conclusions.

---

## Project Structure

```
breast-cancer-sql-analysis/
│
├── Breast_cancer.db          # SQLite database with 3 relational tables
├── breast_cancer.csv         # Raw dataset (from SEER/Kaggle)
│
├── sql/
│   ├── 01_create_tables.sql
│   ├── 02_survival_by_er_status.sql
│   ├── 03_survival_by_stage.sql
│   ├── 04_hormone_receptor_combo.sql
│   ├── 05_stage_x_er_interaction.sql
│   ├── 06_survival_by_age_group.sql
│   └── 07_survival_by_tumor_size.sql
│
├── python/
│   └── statistical_analysis.ipynb   # Chi-square + logistic regression
│
└── README.md
```

---

## How to Reproduce

1. Download `breast_cancer.csv` from [Kaggle](https://www.kaggle.com/datasets/reihanenamdari/breast-cancer)
2. Open DB Browser for SQLite → import CSV as `breast_cancer_raw`
3. Run SQL scripts in the `sql/` folder in order
4. Open `python/statistical_analysis.ipynb` in Jupyter Notebook and run all cells

---

## Author

**Samiksha** — Data Analyst Portfolio Project  
Tools: SQL · Python · Pandas · SciPy · Scikit-learn · Jupyter · DB Browser for SQLite
