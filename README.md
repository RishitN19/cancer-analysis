# Racial Disparities in Breast Cancer Survival: A Multi-Dataset Analysis
### Independent Research Project | NIH TCGA & SEER Data | Python

---

## Background

Breast cancer is the most commonly diagnosed cancer among women in the United States,
making up about 1 in 3 new female cancer diagnoses each year. Survival rates have improved
significantly over the past few decades, but those gains have not reached every group equally.

Black women are roughly 40% more likely to die from breast cancer than White women, even
though they are diagnosed with it less often. That gap has been documented for years, but
the reasons behind it are still not fully understood. Some studies point to later stage at
diagnosis, others to higher rates of triple-negative breast cancer, which is more aggressive
and harder to treat. Others point to gaps in insurance coverage, proximity to specialized
care, or delays between diagnosis and treatment. Most likely it is some combination of all
of these.

This project uses two large NIH datasets to measure the racial survival gap in breast cancer
and test whether it holds up after controlling for cancer stage, treatment, tumor biology,
and socioeconomic factors.

---

## Research Question

Do racial disparities in breast cancer survival persist after controlling for cancer stage,
treatment, and socioeconomic variables, and does race function as an independent predictor
of survival in a machine learning model?

---

## Data Sources

| Dataset | Source | Patients | Years |
|---------|--------|----------|-------|
| TCGA-BRCA | GDC Data Portal | 4,828 | Various |
| SEER Incidence | NIH SEER, 17 Registries | 969,577 | 2000-2017 |

TCGA (The Cancer Genome Atlas) provides clinical and genomic data across 33 cancer types.
SEER (Surveillance, Epidemiology, and End Results) is a population-level cancer registry
covering approximately 35% of the US population.

Raw data files are not included in this repository per SEER data use agreements.
See reproduction steps below.

---

## Methods

**Survival Analysis**
Kaplan-Meier estimators were used to model survival probability over time for each racial
group. This is the standard method in clinical research for comparing survival curves —
it accounts for patients who are still alive at the end of the study period (censored
observations) rather than treating them as non-survivors. Multivariate log-rank tests
assessed whether differences between racial groups were statistically significant.
All analyses were run independently in both TCGA and SEER to test reproducibility.

**Stage-Controlled Analysis**
Stage was simplified into broad categories (TCGA: Stage I/II/III/IV; SEER:
Localized/Regional/Distant). Kaplan-Meier curves and log-rank tests were repeated
within each stage group separately. This controls for the possibility that Black patients
simply present at later stages — if the disparity disappears after controlling for stage,
late diagnosis would be a sufficient explanation. It did not disappear.

**Machine Learning Model**
A Random Forest classifier was trained to predict 5-year survival using 13 features:
age, race, cancer stage, sex, radiation, chemotherapy, tumor grade, histologic type,
axillary node status, neoadjuvant response, median household income, rurality score,
and year of diagnosis. Preprocessing used a sklearn Pipeline with OneHotEncoder for
categorical variables, OrdinalEncoder for stage, and class_weight='balanced' to handle
class imbalance between survivors and non-survivors. The model was evaluated on a
held-out stratified test set (80/20 split) using ROC-AUC rather than accuracy, since
accuracy is misleading on imbalanced datasets — a model predicting everyone survives
would achieve ~75% accuracy without learning anything useful. ROC-AUC measures true
discriminative ability across all classification thresholds and is the standard metric
in clinical prediction modeling.

**SHAP Analysis**
SHAP (SHapley Additive exPlanations) is a method from game theory that explains
individual model predictions by calculating how much each feature contributed to a
specific outcome. Unlike standard feature importance scores which give a single global
number per feature, SHAP produces a value for every patient and every feature — so you
can see not just that race matters on average, but exactly how and in which direction
it pushed each individual prediction. This is important for medical AI because it makes
the model interpretable rather than a black box. SHAP interaction values were also
computed to identify how pairs of features jointly influence predictions, for example
whether the effect of race on survival changes depending on the patient's age.

**Cox Proportional Hazards Regression**
The Cox PH model is the standard statistical model for survival analysis in clinical
research. Unlike the Random Forest which predicts a binary outcome (survived 5 years
or not), Cox regression models the actual time to death and produces hazard ratios —
a hazard ratio above 1.0 means a higher risk of death, below 1.0 means a protective
effect. Cox regression was used here as a complement to the machine learning model
because it provides interpretable, statistically validated coefficients with confidence
intervals and p-values, which is the format clinicians and researchers expect. It also
adjusts for all covariates simultaneously, so the hazard ratio for race reflects its
independent effect after accounting for stage, treatment, grade, income, and all other
variables in the model.

**External Validation**
The SEER-trained model was applied to TCGA-BRCA as an independent external validation
dataset. This tests whether the model generalizes beyond the data it was trained on —
a critical step in clinical model development that is often skipped in student projects.

**Tools:** Python 3.11, pandas, numpy, lifelines, scikit-learn, shap, matplotlib, seaborn

---

## Results

**Overall Survival**
Black patients had significantly worse survival than White patients in both datasets:
- TCGA (n=4,828): log-rank p < 0.0001
- SEER (n=203,057): log-rank p < 0.0001

Asian/Pacific Islander patients had the best outcomes across both datasets.

**Stage-Stratified Analysis**

TCGA results:

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
- Patients: 969,577 (training: 775,661, test: 193,916)
- Accuracy: 76.0%
- ROC-AUC: 0.822

Feature importance ranking (grouped):

| Rank | Feature | Notes |
|------|---------|-------|
| 1 | Age | Strongest predictor |
| 2 | Stage | Second strongest |
| 3 | Axillary nodes positive | Lymph node spread |
| 4 | Grade | Tumor differentiation |
| 5 | Radiation | Treatment variable |
| 6 | Primary site | Tumor location |
| 7 | Histologic type | Tumor biology |
| 8 | Neoadjuvant response | Treatment response |
| 9 | Race | Independent of all above |
| 10 | Chemotherapy | Treatment variable |
| 11 | Diagnosis year | Temporal trends |
| 12 | Median household income | Socioeconomic |
| 13 | Rurality | Socioeconomic |

Race remained an independent predictor after the model accounted for all clinical,
treatment, and socioeconomic variables.

**SHAP Analysis**
Race: Black consistently pushed individual survival predictions toward non-survival.
The SHAP interaction analysis identified Age x Race: Black as a significant interaction
pair, suggesting the racial survival gap is more pronounced in older patients. Income
ranked just below race, indicating socioeconomic factors partially but do not fully
explain the disparity.

**Cox Proportional Hazards Model**
Distant stage had the highest hazard ratio for mortality. Race: Black showed a
statistically significant elevated hazard ratio after adjusting for all covariates.
Race: Asian or Pacific Islander showed a protective hazard ratio relative to White
patients, consistent with the Kaplan-Meier findings.

**External Validation**
The SEER-trained model achieved ROC-AUC 0.770 on TCGA-BRCA (n=331 usable cases).
The drop from 0.822 reflects expected domain shift between population-level SEER
data and academic-center-based TCGA data, and missing socioeconomic variables in TCGA.

---

## Implications

The survival gap held up across every stage group in both datasets and across all
analytical approaches — Kaplan-Meier, Cox regression, and machine learning. After
controlling for treatment, tumor biology, income, and rurality, race still contributed
independently to survival predictions.

This makes it difficult to attribute the disparity to any single factor. The most
likely explanation involves a combination of differential access to high-quality
treatment, higher prevalence of aggressive tumor subtypes among Black patients, and
socioeconomic barriers that these datasets only partially capture. The Age x Race
interaction suggests the gap may widen with age, which warrants further investigation.

---

## Limitations

- ER, PR, and HER2 receptor status are available only in SEER Research Plus, which
  requires a separate data access request. These variables would allow breast cancer
  subtype classification and would likely improve model performance significantly
- Socioeconomic variables are county-level averages, not patient-level data
- TCGA overrepresents academic medical centers and is not population-representative
- TCGA external validation was limited to 331 usable cases due to feature mapping
  constraints between datasets
- Race is self-reported and broadly categorized, potentially masking within-group
  variation
- This is predictive modeling, not causal evidence of disparity mechanisms

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
│   ├── feature_importance.png
│   ├── shap_summary.png
│   ├── shap_bar.png
│   ├── shap_interaction_bar.png
│   ├── shap_interaction_heatmap.png
│   ├── cox_hazard_ratios.png
│   ├── tcga_external_validation_roc.png
│   └── tcga_external_validation_calibration.png
└── README.md
```

## How to Reproduce

1. Request SEER access at seer.cancer.gov/data
2. Download TCGA-BRCA clinical data at portal.gdc.cancer.gov
3. Place files in `data/raw/tcga/` and `data/raw/seer/`
4. Install dependencies: `pip install pandas numpy matplotlib seaborn lifelines scikit-learn shap`
5. Run notebooks in order: 01 -> 02 -> 03

---

## Author

Rishit Nagpal
