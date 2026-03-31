# Racial Disparities in Breast Cancer Survival: A Multi-Dataset Analysis
## Independent Research Project | NIH TCGA & SEER Data | Python

---

## Background

Breast cancer is the most commonly diagnosed malignancy among women in the United States,
comprising approximately 1 in 3 new female cancer diagnoses annually. Despite declining
mortality rates over recent decades, survival improvements have not been distributed equally
across racial groups.

Black women experience roughly 40% higher breast cancer mortality than White women despite
lower overall incidence. The mechanisms underlying this disparity remain incompletely
understood. Proposed explanations include later stage at diagnosis, higher prevalence of
aggressive tumor subtypes such as triple-negative breast cancer, and differential access
to treatment. This project quantifies the racial survival gap using two independent NIH
datasets and tests whether it persists after controlling for stage at diagnosis.

---

## Research Question

Do racial disparities in breast cancer survival persist after controlling for cancer stage,
and does race function as an independent predictor of survival in a machine learning model?

---

## Data Sources

| Dataset | Source | Patients | Years |
|---------|--------|----------|-------|
| TCGA-BRCA | GDC Data Portal | 4,828 | Various |
| SEER Incidence | NIH SEER, 17 Registries | 203,057 | 2000-2022 |

TCGA (The Cancer Genome Atlas) provides clinical and genomic data across 33 cancer types.
SEER (Surveillance, Epidemiology, and End Results) is a population-level cancer registry
covering approximately 35% of the US population.

Raw data files are not included in this repository per SEER data use agreements.
See reproduction steps below.

---

## Methods

**Survival Analysis**
Kaplan-Meier estimators were used to model survival probability over time for each racial
group. Multivariate log-rank tests assessed statistical significance of between-group
differences. All analyses were conducted independently in both TCGA and SEER to evaluate
reproducibility across datasets.

**Stage-Controlled Analysis**
Stage was simplified into broad categories (TCGA: Stage I/II/III/IV; SEER:
Localized/Regional/Distant). Kaplan-Meier curves and log-rank tests were repeated within
each stage stratum to determine whether observed disparities were attributable to
differences in stage at diagnosis.

**Machine Learning Model**
A Random Forest classifier was trained to predict 5-year survival using age, race, stage,
and sex. The model was evaluated on a held-out test set (80/20 split) using accuracy,
ROC-AUC, and a full classification report. Feature importance scores were extracted to
quantify the independent contribution of each variable.

**Tools:** Python 3.11, pandas, numpy, lifelines, scikit-learn, matplotlib, seaborn

---

## Results

**Overall Survival**
Black patients had significantly worse survival than White patients in both datasets:
- TCGA (n=4,828): log-rank p < 0.0001
- SEER (n=203,057): log-rank p < 0.0001

Asian/Pacific Islander patients had the best outcomes across both datasets.

**Stage-Stratified Analysis**
The disparity persisted after stratification by stage. TCGA results:

| Stage | p-value | Significant? |
|-------|---------|--------------|
| Stage I | 0.0358 | Yes |
| Stage II | < 0.0001 | Yes |
| Stage III | 0.0926 | No |
| Stage IV | 0.0002 | Yes |

SEER results:

| Stage | p-value | Significant? |
|-------|---------|--------------|
| Localized | < 0.0001 | Yes |
| Regional | < 0.0001 | Yes |
| Distant | < 0.0001 | Yes |

**Machine Learning Model**
- Accuracy: 81% on test set (n=29,855)
- ROC-AUC: 0.824
- Feature importance: Stage (0.68) > Age (0.30) > Race (0.02) > Sex (0.002)

Race contributed independently to survival predictions after accounting for stage and age.

---

## Implications

The persistence of racial survival disparities across stage groups in two independent
datasets indicates the gap is not fully explained by differences in stage at diagnosis.
The independent contribution of race in the ML model further supports this conclusion.
Likely contributing factors include differential access to treatment, higher prevalence
of aggressive tumor subtypes among Black patients, and socioeconomic barriers not
captured in these datasets. Findings suggest that interventions targeting screening
alone are insufficient; systemic factors at the treatment level warrant further
investigation.

---

## Limitations

- Neither dataset includes treatment variables such as chemotherapy regimen, hormone
  therapy, or surgical approach, which are significant confounders
- Socioeconomic variables including income, insurance status, and proximity to care
  are unavailable at the patient level
- TCGA overrepresents academic medical centers and may not generalize to the broader
  population
- Race is self-reported and broadly categorized, potentially masking within-group variation
- The ML model is limited to 4 features; inclusion of tumor subtype and treatment data
  would likely improve predictive performance

---

## Repository Structure
```
cancer-analysis/
├── notebooks/
│   ├── 01_tcga_exploration.ipynb
│   ├── 02_seer_exploration.ipynb
│   └── 03_ml_model.ipynb
├── figures/
│   ├── survival_by_race.png
│   ├── survival_by_race_and_stage.png
│   ├── seer_survival_by_race.png
│   ├── seer_survival_by_race_and_stage.png
│   └── feature_importance.png
└── README.md
```

---

## How to Reproduce

1. Request SEER access at seer.cancer.gov/data
2. Download TCGA-BRCA clinical data at portal.gdc.cancer.gov
3. Place files in `data/raw/tcga/` and `data/raw/seer/`
4. Install dependencies: `pip install pandas numpy matplotlib seaborn lifelines scikit-learn`
5. Run notebooks in order: 01 -> 02 -> 03

---

## Author

Rishit Nagpal | Denmark High School, Alpharetta, GA
Pre-Medicine | Interests: Oncology, Health Disparities, Biomedical Data Science
