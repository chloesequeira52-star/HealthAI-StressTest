# HealthAI StressTest

### Stress-testing healthcare prediction models for hidden equity failures under realistic data degradation

Healthcare prediction models can appear accurate overall while performing very differently across patient populations.

**HealthAI StressTest** is an experimental health equity project that asks a different question from a standard machine-learning analysis:

> **What happens when we deliberately break the data used to train a healthcare prediction model?**

Using nationally representative U.S. healthcare data, I built a baseline model to identify adults with high annual healthcare expenditures and systematically introduced five realistic data-quality failures.

The objective was to determine whether aggregate model performance could remain relatively stable while errors became concentrated within specific demographic or socioeconomic populations.

---

## Key Finding

The model was relatively resilient to several isolated forms of training-data degradation.

However, **systematic bias in the outcome labels used to train the model produced a major equity failure.**

When high healthcare expenditures among lower-income adults were increasingly misclassified in the training data:

- Lower-income false-negative rate increased from **26.5% to 83.3%**
- Higher-income false-negative rate remained approximately **25%**
- The income-group FNR gap increased from **1.3 to 58.0 percentage points**
- The first tested condition exceeding the project's **10-percentage-point equity warning threshold occurred at 25% label error**
- At that point, overall FNR increased by only **1.6 percentage points**, even though the lower-income FNR reached **38.1%**

This demonstrates how **aggregate performance metrics can conceal substantial subgroup harm**.



![Systematic Label Bias Equity Failure](figures/healthai_label_bias_equity_failure.png)

---

## Research Question

> **How rapidly do subgroup performance disparities worsen when healthcare training data are systematically degraded?**

A secondary question was:

> **Can overall predictive performance remain apparently acceptable while subgroup equity deteriorates?**

---

## Data

This project uses the **2024 Medical Expenditure Panel Survey (MEPS) Full-Year Consolidated File**, released by the Agency for Healthcare Research and Quality in 2026.

MEPS contains nationally representative information on:

- healthcare expenditures
- healthcare utilization
- demographics
- income and poverty status
- insurance coverage
- physical health
- mental health
- chronic conditions

### Analytic Sample

After restricting the dataset to adults age 18+ with valid values for the baseline predictors:

- **Total analytic sample:** 14,666
- **Training sample:** 10,266
- **Testing sample:** 4,400

The MEPS person-level survey weight was incorporated into model fitting and performance calculations.

The raw MEPS dataset is **not included in this repository**. It can be obtained directly from AHRQ.

---

## Outcome

The model classified whether an adult had **high annual healthcare expenditures**.

High-cost status was defined as:

> Annual healthcare expenditures at or above the **survey-weighted 80th percentile** of expenditures in the training sample.

The resulting expenditure threshold was:

**$11,223**

The weighted prevalence of the high-cost outcome was:

- Training: **20.0%**
- Testing: **20.6%**

Because predictors and expenditures came from the same year, this project should be interpreted as a **risk-classification and model robustness experiment**, rather than prospective prediction of future expenditures.

---

## Baseline Model

A survey-weighted **logistic regression model** was used as the primary baseline classifier.

Predictors included:

- age
- sex
- poverty category
- education
- insurance coverage
- self-rated physical health
- self-rated mental health
- hypertension
- diabetes

### Important Equity Design Choice

**Race/ethnicity was not included as a model predictor.**

Instead, race/ethnicity was retained exclusively for subgroup performance auditing.

This allowed the project to examine whether a model that does not explicitly use race could still exhibit substantial racial/ethnic performance differences.

### Baseline Performance

- Training AUC: **0.777**
- Testing AUC: **0.760**

The classification threshold was selected using **Youden's J statistic on the training sample** and then frozen for subsequent stress tests.

---

## Baseline Equity Audit

Model performance was evaluated separately across racial/ethnic groups using:

- AUC
- sensitivity
- specificity
- positive predictive value
- false-negative rate
- Brier score
- mean predicted-versus-observed risk gap

A substantial disparity was already present before any artificial data degradation was introduced.

| Group | AUC | Sensitivity | False-Negative Rate |
|---|---:|---:|---:|
| Overall | 0.760 | 74.6% | 25.4% |
| Hispanic | 0.783 | 67.5% | 32.5% |
| NH White | 0.741 | 75.8% | 24.2% |
| NH Black | 0.834 | 87.2% | 12.8% |
| NH Asian | 0.645 | 52.0% | 48.0% |
| NH Other/Multiple | 0.764 | 62.8% | 37.2% |

The maximum racial/ethnic false-negative-rate difference was approximately:

**35.3 percentage points**

The NH Asian subgroup had the highest baseline false-negative rate at **48.0%**.

These results should be interpreted cautiously for smaller subgroups, particularly NH Asian and NH Other/Multiple adults.

---

# The Five Crash Tests

## Crash Test 1 — Selective Poverty-Data Missingness

### Question

What happens when socioeconomic information becomes increasingly incomplete specifically for lower-income adults?

Poverty-category values were removed from progressively larger percentages of lower-income participants in the training set:

- 0%
- 10%
- 25%
- 50%
- 75%
- 100%

Missing values were handled through the model preprocessing pipeline.

The test population remained unchanged.

### Result

At 100% selective poverty-category missingness:

- Lower-income FNR: **26.5% → 29.2%**
- Change: **+2.7 percentage points**

### Interpretation

**No material additional degradation detected.**

The model was relatively resilient to isolated loss of poverty-category information.

---

## Crash Test 2 — Racial/Ethnic Underrepresentation

### Question

Does reducing representation of an already poorly performing subgroup make its model performance substantially worse?

NH Asian participants were progressively removed from the training dataset while the test population remained unchanged.

Training representation levels:

- 100%
- 75%
- 50%
- 25%
- 10%

Each degraded representation scenario was evaluated across **50 repeated simulations**.

### Result

NH Asian performance remained remarkably stable.

| NH Asian Training Representation | Mean AUC | Mean Sensitivity | Mean FNR |
|---:|---:|---:|---:|
| 100% | 0.645 | 52.0% | 48.0% |
| 75% | 0.644 | 52.0% | 48.0% |
| 50% | 0.643 | 52.0% | 48.0% |
| 25% | 0.643 | 52.0% | 48.0% |
| 10% | 0.643 | 52.0% | 48.0% |

### Interpretation

**No material additional degradation detected.**

The baseline NH Asian performance disparity was not explained simply by the number of NH Asian observations represented in the training data.

This result does **not** imply that demographic underrepresentation is harmless in healthcare AI generally. It applies only to this dataset, outcome, predictor set, model, and experimental design.

---

## Crash Test 3 — Measurement Error

### Question

How robust is the model to realistic reporting error in self-rated physical health?

Increasing percentages of `RTHLTH53` values were shifted by one plausible response category.

Measurement-error levels:

- 0%
- 5%
- 10%
- 20%
- 30%
- 40%

Each scenario was repeated across **50 simulations**.

### Result

Even at 40% measurement error, subgroup false-negative rates changed relatively little.

The largest mean subgroup FNR deterioration was approximately:

**+2.0 percentage points**

### Interpretation

**No material additional degradation detected.**

The model appeared relatively robust to isolated measurement error in this predictor.

---

## Crash Test 4 — Population Shift

### Question

What happens when a healthcare model is developed in one insurance population and deployed in another?

Two models were compared on the exact same test population of **1,803 public-insurance or uninsured adults**.

### Model A

Trained using the full representative training population.

### Model B

Trained using **privately insured adults only**.

Insurance status itself was removed from the predictors for this experiment.

| Metric | Representative Training | Private-Only Training |
|---|---:|---:|
| AUC | 0.774 | 0.771 |
| Sensitivity | 83.2% | 82.8% |
| Specificity | 56.5% | 54.3% |
| PPV | 35.7% | 34.5% |
| FNR | 16.8% | 17.2% |
| Brier Score | 0.146 | 0.147 |

### Interpretation

**No material additional degradation detected.**

Insurance-based population shift produced only modest deterioration in this model.

---

# Crash Test 5 — Systematic Label Bias

## Question

What happens when the training data systematically teach the model the wrong outcome for a disadvantaged population?

Among lower-income adults who were genuinely high cost, increasing percentages of positive training labels were deliberately changed from:

`HIGH_COST = 1`

to:

`HIGH_COST = 0`

Label-error levels:

- 0%
- 10%
- 25%
- 50%
- 75%

Each scenario was evaluated across **50 simulations**.

The test dataset remained completely unchanged and retained the correct outcomes.

### Results

| Label Error | Higher-Income FNR | Lower-Income FNR | FNR Gap | Overall FNR |
|---:|---:|---:|---:|---:|
| 0% | 25.2% | 26.5% | 1.3 pp | 25.4% |
| 10% | 25.0% | 30.7% | 5.7 pp | 25.9% |
| 25% | 25.0% | 38.1% | **13.1 pp** | 27.0% |
| 50% | 25.2% | 55.0% | **29.9 pp** | 29.6% |
| 75% | 25.3% | 83.3% | **58.0 pp** | 33.9% |

### Equity Failure Threshold

For this project, a **10-percentage-point subgroup FNR gap** was pre-specified as an exploratory equity-warning threshold.

The first tested scenario exceeding that threshold occurred at:

## **25% systematic label error**

At that level:

- Overall FNR increased only **1.6 percentage points**
- Lower-income FNR increased **11.6 percentage points**
- The income-group FNR gap reached **13.1 percentage points**

This is the central finding of HealthAI StressTest.

---

# Stress-Test Summary

![HealthAI StressTest Summary](figures/healthai_crash_test_summary.png)

| Experiment | Primary Equity Result | Status |
|---|---|---|
| Baseline Audit | 35.3 pp max racial/ethnic FNR gap | **BASELINE DISPARITY** |
| Poverty Missingness | +2.7 pp lower-income FNR | No material additional degradation |
| Underrepresentation | +0.0 pp NH Asian FNR | No material additional degradation |
| Measurement Error | +2.0 pp largest subgroup FNR | No material additional degradation |
| Population Shift | +0.4 pp deployment FNR | No material additional degradation |
| **Systematic Label Bias** | **+56.8 pp lower-income FNR** | **EQUITY FAILURE** |

---

## What This Experiment Suggests

The stress tests produced two major observations.

### 1. Baseline equity auditing matters

The model exhibited substantial subgroup differences **before artificial degradation was introduced**.

A model can therefore perform reasonably well overall while providing very different levels of reliability across patient populations.

### 2. Not all data-quality failures are equally harmful

In this experiment:

- predictor missingness produced limited additional deterioration
- subgroup underrepresentation produced limited additional deterioration
- isolated measurement error produced limited additional deterioration
- insurance-based population shift produced limited additional deterioration
- **systematic bias in training labels produced severe subgroup harm**

This suggests that evaluating healthcare AI requires attention not only to which populations appear in the data, but also to whether outcomes are measured and recorded consistently across populations.

---

## Why Aggregate Metrics Were Not Enough

Crash Test 5 illustrates the central problem.

At 25% systematic label error:

**Overall FNR**

25.4% → 27.0%

Only a **1.6 percentage-point increase**.

But:

**Lower-income FNR**

26.5% → 38.1%

An **11.6 percentage-point deterioration**.

A model-monitoring process based only on aggregate performance could therefore miss an emerging subgroup failure.

---

## Methods Summary

### Modeling

- Survey-weighted logistic regression
- 70/30 train/test split
- Training-only definition of the high-cost expenditure threshold
- Training-only selection of classification threshold
- Frozen classification threshold during stress testing

### Equity Metrics

- Area under the ROC curve
- Sensitivity
- Specificity
- Positive predictive value
- False-negative rate
- Brier score
- Predicted-versus-observed risk gap
- Between-group FNR differences

### Robustness Design

Where random sampling or corruption was involved, repeated simulations were used to distinguish systematic effects from individual random draws.

---

## Repository Structure

```text
HealthAI-StressTest/
│
├── README.md
├── HealthAI_StressTest.ipynb
├── LICENSE
├── .gitignore
│
├── figures/
│   ├── healthai_crash_test_summary.png
│   └── healthai_label_bias_equity_failure.png
│
└── results/
    ├── healthai_final_summary.csv
    ├── healthai_baseline_equity_audit.csv
    └── healthai_label_bias_results.csv

## Tools

- Python
- pandas
- NumPy
- scikit-learn
- Matplotlib
- Google Colab
- MEPS survey data

---

## Skills Demonstrated

This project demonstrates experience with:

- healthcare analytics
- predictive modeling
- logistic regression
- responsible AI
- algorithmic fairness
- health equity analysis
- model evaluation
- subgroup performance auditing
- robustness testing
- simulation experiments
- survey-weighted healthcare data
- feature preprocessing
- missing-data handling
- population shift analysis
- data visualization
- reproducible research

---

## Limitations

This project is an exploratory portfolio analysis rather than a validated clinical prediction model.

Important limitations include:

- The analysis uses one year of MEPS data.
- The outcome and predictors are measured within the same year.
- Some racial/ethnic subgroup test samples were relatively small.
- The baseline model uses a limited predictor set.
- Logistic regression was used rather than comparing multiple machine-learning architectures.
- The 10-percentage-point equity-warning threshold is a **project-defined exploratory threshold**, not an established clinical fairness standard.
- The calibration-gap measure used here summarizes mean predicted versus observed risk and is not a complete calibration analysis.
- Artificial stress tests approximate possible data failures and should not be interpreted as estimates of the prevalence of these errors in real healthcare systems.
- Results should not be generalized to healthcare AI systems beyond this experimental setting.

---

## Potential Extensions

Future versions of HealthAI StressTest could include:

- XGBoost and random forest models
- calibration curves and calibration slope/intercept
- confidence intervals for subgroup performance
- bootstrap uncertainty estimation
- intersectional subgroup analysis
- additional healthcare outcomes
- temporal validation
- external validation
- automated stress-test configuration
- interactive Streamlit dashboard
- downloadable model equity reports

A future application could allow users to:

1. Upload a healthcare dataset
2. Select an outcome
3. Select demographic or protected groups
4. Choose stress-test scenarios
5. Set degradation levels
6. Automatically calculate subgroup performance
7. Identify potential equity-failure thresholds
8. Export an equity audit report

---

## Project Takeaway

> **A healthcare prediction model does not have to fail overall to fail a population.**

HealthAI StressTest demonstrates why healthcare model evaluation should examine not only average predictive performance, but also how data quality failures affect specific populations.

In this experiment, systematic label bias created severe socioeconomic disparities long before aggregate model performance appeared catastrophic.
