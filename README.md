# AML Alert Triage Analytics

**INFO 320 Final Project | Anti-Money Laundering (AML) Transaction Monitoring**

A simulation of a bank's AML review team with a fixed daily capacity. It compares rule-based alerts with machine-learning models, then tests two upgrades: network-derived features and a feedback loop that retrains on investigator outcomes.

## Business question

Kestrel, a fictional bank, can review only 50 alerts a day. Which alerts should its investigators see, and which monitoring upgrades are worth their added cost?

Systems are judged by how much laundering they catch within that fixed capacity, not by accuracy. Laundering is rare enough that a system flagging nothing would be 99.75% accurate. The audience is Kestrel's BSA Officer, CFO and Model Risk Committee.

## Data and approach

- **Data:** Synthetic SAML-D transactions (approximately 9.5 million transactions).
- **Unit of evaluation:** A sender account on a particular day (sender-day); this is **not** the same as an individual transaction or confirmed SAR.
- **History:** Oct 7–Dec 31, 2022, used for feature design, rule calibration and initial training.
- **Test period:** January 1–August 23, 2023 (235 calendar days).
- **Positive sender-days in test period:** 6,861.
- **Review policy:** At 50 cases/day, the model prioritizes 45 high-scoring cases plus 5 random cases from just below the cutoff (46-200) to find near-misses.
- **Models:** Rules baseline, logistic regression, and LightGBM, evaluated as static or monthly-adaptive systems with nine behavioral or 11 behavioral-plus-network features.
- **Feedback simulation:** Monthly retraining uses previously selected, simulated investigator outcomes.
- **Hypotheses**
**H1:** A machine-learning model catches more laundering than the rules at 50 reviews a day.
**H2:** Network-derived features improve detection.
**H3:** Monthly retraining on investigator outcomes beats the same model left unchanged.
**H4 (future pilot):** An AI investigation layer improves on the model alone at an acceptable cost per added case. Designed, not run.

## Results

All rows below use **11,750 total reviews 50 a day over 235 days**:

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

T### Tableau

The five exported CSVs support three interactive dashboards: **Detection Performance**, **Learning and Detection Gaps**, and **Operational Impact**.

**[View Interactive Tableau Workbook](https://us-east-1.online.tableau.com/#/site/info320_003/workbooks/5985947?:origin=card_share_link)**

## Important limitations

- **Synthetic data:** Reported results do not demonstrate performance at a real financial institution.
- **Simulated feedback:** reviewed outcomes are always correct and arrive at month-end. Real investigations take longer and are less certain.
- **Selection bias:** Retraining on reviewed alerts can reinforce which cases get labeled. The five exploratory reviews help but do not remove this risk.
- **Model risk and drift:** Real deployment would require independent validation, monitoring, model change controls, human oversight, and operational testing.
- **Capacity experiments:** Every tested limit reserves five exploration reviews per day, so the fraction of exploration cases changes as capacity changes.
- **Governance:** real deployment would need independent validation, ongoing monitoring and model change controls. Under SR 26-2 (April 2026), monthly retraining counts as a model change.
- **Computing/reproducibility:** The notebooks were developed in Google Colab using Google Drive for storage. Notebook 01 includes a dataset setup process to simplify access to the original SAML-D data. Large processed datasets and trained models are excluded from GitHub because of their size and must be regenerated by running the notebooks in order.

## Running the notebooks

The notebooks were built in **Google Colab** with Python, pandas, scikit-learn and LightGBM. They read SAML-D and intermediate files from Google Drive. Notebook 01 automatically sets up the required project folders and downloads the original dataset, while subsequent notebooks save and access files within that structure. The raw and processed datasets and the saved models are not in this repo because of their size.
