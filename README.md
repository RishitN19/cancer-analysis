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

This project tries to get at one specific piece of that question: is the survival gap
explained by the fact that Black patients tend to be diagnosed at later stages, or does
the disparity exist even within the same stage? Using two large NIH datasets, I ran
survival analyses within each stage group separately to test this.

---

## Research Question

**Do racial disparities in breast cancer survival persist after controlling for cancer stage, 
and can a machine learning model identify race as an independent predictor of survival?**

---

## Data Sources

| Dataset | Source | Patients | Years |
|---------|--------|----------|-------|
| TCGA-BRCA | GDC Data Portal (portal.gdc.cancer.gov) | 4,828 | Various |
| SEER Incidence | NIH SEER Program, 17 Registries (seer.cancer.gov) | 203,057 | 2000–2022 |

**TCGA** (The Cancer Genome Atlas) provides detailed clinical and genomic data for cancer 
patients across 33 cancer types. The BRCA dataset covers breast cancer specifically.

**SEER** (Surveillance, Epidemiology, and End Results) is a population-level registry 
covering approximately 35% of the US population, providing incidence and survival data 
going back to 1975.

> Note: Raw data files are not included in this repository per SEER data use agreements. 
> See the Data Access section below for instructions on downloading the data.

---

## Methods

### Survival Analysis
- **Kaplan-Meier estimator** used to model survival probability over time for each racial group
- **Log-rank test** (multivariate) used to assess whether survival differences between 
  racial groups are statistically significant
- Analyses performed separately in TCGA and SEER to test reproducibility across datasets

### Stage-Controlled Analysis
- Cancer stage simplified into broad categories (TCGA: Stage I/II/III/IV; SEER: Localized/Regional/Distant)
- Kaplan-Meier curves and log-rank tests repeated within each stage group independently
- This controls for the possibility that Black patients are simply diagnosed at later stages

### Machine Learning Model
- **Random Forest Classifier** trained to predict 5-year survival (binary outcome)
- Features: age at diagnosis, race, cancer stage, sex
- 80/20 train/test split with fixed random seed for reproducibility
- Model evaluated using accuracy, classification report, and ROC-AUC score
- Feature importance extracted to quantify the independent contribution of race

### Tools & Libraries
Python 3.11 | pandas | numpy | lifelines | scikit-learn | matplotlib | seaborn

---

## Key Findings

### 1. Overall Survival Disparity (Both Datasets)
Black patients had significantly worse breast cancer survival than White patients in both 
independent datasets:
- **TCGA** (n=4,828): Log-rank p < 0.0001
- **SEER** (n=203,057): Log-rank p < 0.0001

Asian/Pacific Islander patients had the best survival outcomes across both datasets.

### 2. Disparity Persists After Controlling for Stage
The racial survival gap was not explained by differences in stage at diagnosis. After 
stratifying by stage, Black patients still had significantly worse survival:

**TCGA Results:**
| Stage | p-value | Significant? |
|-------|---------|--------------|
| Stage I | 0.0358 | Yes |
| Stage II | < 0.0001 | Yes |
| Stage III | 0.0926 | No |
| Stage IV | 0.0002 | Yes |

**SEER Results:**
| Stage | p-value | Significant? |
|-------|---------|--------------|
| Localized | < 0.0001 | Yes |
| Regional | < 0.0001 | Yes |
| Distant | < 0.0001 | Yes |

### 3. Machine Learning Model
- **Accuracy: 81%** on held-out test set (n=29,855)
- **ROC-AUC: 0.824** — strong discriminative performance
- Feature importance ranking: Stage (0.68) > Age (0.30) > Race (0.02) > Sex (0.002)
- Race remained an independent predictor of survival even after the model accounted 
  for stage and age

---

## Implications

The survival gap held up across almost every stage group in both datasets, which makes
it difficult to attribute entirely to later diagnosis. If the disparity were purely a
screening problem, you would expect it to shrink or disappear once patients are matched
by stage. That did not happen here.

What that points to is something happening at or after the treatment level. The datasets
do not include enough variables to pinpoint the cause, but the likely candidates are
differences in treatment quality and access, higher rates of aggressive tumor subtypes
among Black patients, and socioeconomic factors that affect follow-through on care. The
ML model adds to this picture by showing that race contributes independently to survival
predictions even after stage and age are already accounted for.

More granular data, including treatment regimens, insurance status, and tumor subtype,
would be needed to separate these factors out properly. But the pattern across two
independent datasets with over 200,000 patients is consistent enough that it warrants
that kind of deeper investigation.

---

## Limitations

- SEER and TCGA datasets do not include treatment details (chemotherapy regimens, 
  hormone therapy, surgery type), which are important confounders
- Socioeconomic variables (income, insurance status, distance to care) are not available 
  at the patient level in either dataset
- TCGA is not a population-representative sample — it skews toward academic medical centers
- Race is self-reported and categorized broadly, which may mask within-group heterogeneity
- The ML model uses only 4 features; a more complete model would include tumor subtype, 
  comorbidities, and treatment variables

---

## Repository Structure
```
cancer-analysis/
├── notebooks/
│   ├── 01_tcga_exploration.ipynb    # TCGA data cleaning & survival analysis
│   ├── 02_seer_exploration.ipynb    # SEER analysis & stage-controlled curves
│   └── 03_ml_model.ipynb            # Random Forest survival prediction model
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

1. Request SEER data access at [seer.cancer.gov/data](https://seer.cancer.gov/data)
2. Download TCGA-BRCA clinical data at [portal.gdc.cancer.gov](https://portal.gdc.cancer.gov)
3. Place files in `data/raw/tcga/` and `data/raw/seer/` respectively
4. Install dependencies:
```bash
pip install pandas numpy matplotlib seaborn lifelines scikit-learn
```
5. Run notebooks in order: `01` → `02` → `03`

---

## Author

**Rishit Nagpal** | Denmark High School, Alpharetta, GA  
