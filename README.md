# AML Alert Triage Analytics

**INFO 320 Final Project | Anti-Money Laundering (AML) Transaction Monitoring**

An investigation-capacity simulation comparing rule-based alerts with machine-learning models, network-context features, and a feedback loop that retrains on previously reviewed cases.

## Business question

A bank's investigators have limited time. Which alerts should they review first, and how can the monitoring system adapt as patterns change?

This project evaluates detection **at a fixed daily review capacity**, rather than ranking models only by traditional classification scores. The intended audience is a fictional bank's BSA Officer, CFO, and Model Risk Committee.

## Data and approach

- **Data:** Synthetic SAML-D transactions (approximately 9.5 million transactions).
- **Unit of evaluation:** A sender account on a particular day (sender-day); this is **not** the same as an individual transaction or confirmed SAR.
- **Test period:** January 1–August 23, 2023 (235 calendar days).
- **Positive sender-days in test period:** 6,861.
- **Review policy:** At 50 cases/day, the model prioritizes 45 high-scoring cases and selects five additional cases for exploration.
- **Models:** Rules baseline, logistic regression, and LightGBM, evaluated as static or monthly-adaptive systems with nine behavioral or 11 behavioral-plus-network features.
- **Feedback simulation:** Monthly retraining uses previously selected, simulated investigator outcomes; labels come from synthetic ground truth.

## Results

All rows below use **11,750 total reviews over 235 days**:

| System | Cases caught | Recall | Precision |
| --- | ---: | ---: | ---: |
| Rules baseline | 163 | 2.38% | 1.39% |
| Static LightGBM, 9 features | 225 | 3.28% | 1.91% |
| Adaptive LightGBM, 9 features | 455 | 6.63% | 3.87% |
| Adaptive LightGBM, 11 features | **716** | **10.44%** | **6.09%** |

Adding the two network-context features to adaptive LightGBM increased detections from 455 to 716 at the same review volume.

**Workload scenario:** In a separate capacity simulation, adaptive LightGBM with 11 features caught 236 positive sender-days at 10 reviews/day, versus 163 for rules at 50 reviews/day. This would mean 9,400 fewer reviews across the test period. At an illustrative 37 minutes/review and $40/hour, that represents approximately **5,797 hours** or **$232,000 in the value of investigator time potentially freed**—**not necessarily a reduction in cash expenses**.

## Repository structure

```text
notebooks/
  01_data_audit.ipynb
  02_data_cleaning.ipynb
  03_EDA.ipynb
  04_modeling.ipynb
  05_feedback_loop_network_features.ipynb

tableau/
  model_comparison.csv
  monthly_performance.csv
  typology_performance.csv
  review_capacity.csv
  labor_savings.csv

reports/                 # Final report / presentation when available
README.md
.gitignore
```

### Notebook workflow

1. **01 — Data Audit:** Inspect data quality, class imbalance, transaction labels, and coverage.
2. **02 — Data Cleaning:** Normalize labels, datatypes, and timestamps.
3. **03 — EDA & Features:** Explore activity, design sender-day features, and examine laundering types.
4. **04 — Modeling:** Build rule-based and static machine-learning baselines.
5. **05 — Feedback & Network Features:** Simulate daily review queues, monthly retraining, network features, capacity sensitivity, and Tableau exports.

### Tableau

The five exported CSVs support three planned dashboards: **Detection Performance**, **Learning and Detection Gaps**, and **Operational Impact**. Tableau Public publishing and the final workbook are in progress.

## Important limitations

- **Synthetic data:** Reported results do not demonstrate performance at a real financial institution.
- **Simulated feedback:** Synthetic labels are assumed available after investigation; actual bank investigations can take longer and have uncertain outcomes.
- **Selection bias:** Retraining on reviewed alerts can reinforce which cases get labeled. The five exploratory reviews help but do not remove this risk.
- **Model risk and drift:** Real deployment would require independent validation, monitoring, model change controls, human oversight, and operational testing.
- **Capacity experiments:** Every tested limit reserves five exploration reviews per day, so the fraction of exploration cases changes as capacity changes.
- **Computing/reproducibility:** The original notebooks use Google Colab and paths to files stored in Google Drive. Large raw/processed datasets and serialized models are excluded from this repository.

## Running the notebooks

The notebooks were developed in **Google Colab** with Python, pandas, scikit-learn, and LightGBM. They expect the SAML-D source data and intermediate artifacts in Google Drive; adapt the Drive folder paths if reproducing. The complete 9.5-million-row dataset is intentionally not committed.

## Project status

Modeling, feedback-loop experiments, and CSV exports are complete. Tableau dashboards and the presentation/report are being prepared.
