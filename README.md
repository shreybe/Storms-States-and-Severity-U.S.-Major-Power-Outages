# Storms, States, and Severity: U.S. Major Power Outages

**By Shreya Bharathwajan**

*Project for DSC 80 at UCSD*

---

## Introduction

This project examines major power outages in the continental U.S. from January 2000 to July 2016. The dataset comes from [Purdue University](https://engineering.purdue.edu/LASCI/research-data/outages) and includes outage details along with geographic, climate, land use, and economic indicators for the affected states. Understanding the drivers of long outages helps utilities prioritize mitigation and resilience planning.

**Research question:** How do climate region and outage cause relate to outage duration?

The raw dataset contains **1534 rows** (outage events) and 56 columns. Relevant columns include:

| Column | Description |
|--------|-------------|
| `OUTAGE.DURATION` | Duration of the outage in minutes |
| `CAUSE.CATEGORY` | Main cause category for the outage |
| `CLIMATE.REGION` | U.S. climate region |
| `OUTAGE.START.DATE` | Date the outage started |
| `OUTAGE.START.TIME` | Time the outage started |
| `U.S._STATE` | State where the outage occurred |
| `MONTH` | Month of the outage |
| `YEAR` | Year of the outage |
| `CUSTOMERS.AFFECTED` | Number of customers affected |
| `DEMAND.LOSS.MW` | Estimated demand loss in megawatts |

---

## Data Cleaning and Exploratory Data Analysis

### Cleaning

I combined `OUTAGE.START.DATE` with `OUTAGE.START.TIME` into a single `OUTAGE.START` timestamp, and did the same for restoration time using `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME`. I converted outage duration into hours (`OUTAGE.DURATION_HR`) for easier interpretation. I also created a `SEASON` feature from the outage month and a log-transformed population feature (`LOG_POPULATION`) to reduce skew for modeling.

### Univariate Analysis

The distribution of outage duration (in hours) shows that most outages are relatively short, with a long right tail of longer-duration events.

<iframe
  src="assets/duration-histogram.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

Outage duration varies noticeably by climate region. The Southeast and Northeast tend to have more prolonged outages on average.

<iframe
  src="assets/duration-by-region.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

Average outage duration and customers affected by cause category:

| CAUSE.CATEGORY | OUTAGE.DURATION_HR | CUSTOMERS.AFFECTED |
|----------------|--------------------|--------------------|
| fuel supply emergency | 224.73 | 0.14 |
| severe weather | 65.49 | 0.20 |
| equipment failure | 30.78 | 0.13 |
| ... | ... | ... |

---

## Assessment of Missingness

### MNAR Analysis

I believe **`DEMAND.LOSS.MW`** is likely **MNAR**. Large, well-documented outages are more likely to have demand loss estimates reported, while smaller or less documented events may omit it. This suggests the missingness could depend on the unobserved demand loss itself. Additional data on reporting practices or utility measurement protocols could make this missingness MAR.

### Missingness Dependency

I tested whether missingness in `DEMAND.LOSS.MW` depends on `OUTAGE.DURATION_HR` (it does) and on `MONTH` (it does not). The plot below shows the distribution of outage duration when demand loss is missing vs. not missing.

<iframe
  src="assets/missingness-duration.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Hypothesis Testing

**Null hypothesis:** The distribution of outage duration is the same in winter and summer.

**Alternative hypothesis:** Winter outages have larger typical duration than summer outages.

**Test statistic:** Difference in medians (winter − summer)

**Significance level:** α = 0.05

**Result:** [Report your p-value and conclusion here after running the test]

---

## Framing a Prediction Problem

**Prediction problem:** Predict outage duration (in hours) from characteristics known at the time of prediction.

**Type:** Regression

**Response variable:** `OUTAGE.DURATION_HR` — I chose this because duration is a key measure of outage severity and is useful for utility planning.

**Evaluation metric:** RMSE — I chose RMSE because it is in the same units as the target and penalizes large errors.

**Features:** Month, year, climate region, cause category, anomaly level, population (all known at prediction time).

---

## Baseline Model

**Features:** `MONTH` (ordinal), `YEAR` (ordinal), `ANOMALY.LEVEL` (quantitative), `POPULATION` (quantitative), `CAUSE.CATEGORY` (nominal), `CLIMATE.REGION` (nominal). Categorical features are one-hot encoded; numeric features use median imputation.

**Model:** Linear regression in an sklearn Pipeline.

**Performance:** [Report train and test RMSE here]

---

## Final Model

**New features added:** `SEASON` (nominal), `OUTAGE.START.HOUR` (ordinal), `LOG_POPULATION` (quantitative) — season captures seasonal patterns, hour of day may relate to restoration speed, and log population reduces skew.

**Model:** RandomForestRegressor with GridSearchCV for `n_estimators`, `max_depth`, and `min_samples_leaf`.

**Best hyperparameters:** [Report here]

**Performance:** [Report train and test RMSE; describe improvement over baseline]

---

## Fairness Analysis

**Groups:** Climate region A vs. Climate region B (the two regions with most test data)

**Metric:** Mean absolute error (MAE)

**Null hypothesis:** The model is fair. MAE is the same across the two climate regions.

**Alternative hypothesis:** The model is unfair. MAE differs significantly between the two regions.

**Result:** [Report p-value and conclusion here]

---

*This site is maintained by shreybe.*
