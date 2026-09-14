---
layout: default
title: Machine Learning for Search Performance Decline and Content Opportunity Prioritization
---

# Machine Learning for Search Performance Decline and Content Opportunity Prioritization

## Abstract

Search-performance monitoring often produces a large number of pages that require investigation. A practical challenge is deciding which pages should be reviewed first.

This study investigates whether observed search-performance and content signals can be combined into an interpretable machine-learning model that prioritizes potentially declining pages for human review more effectively than a simple hand-written baseline.

The analysis uses the FlyRank ML Internship starter dataset, `content_refresh_anonymized.csv`. The modelling features include click-through rate (CTR), average search position, search volume, 90-day impressions, 90-day clicks, and content age. The target variable, `is_declining_label`, is derived from the observed `trend_direction` field, where `down` represents a declining observation.

To reduce leakage between observations from the same client, the experiment uses a client-level 80/20 holdout split. The primary ranking metric is Precision@50, measuring the proportion of declining observations among the 50 highest-ranked pages.

On the executed validation split, the Logistic Regression model achieved a Precision@50 of **0.68**, compared with **0.52** for the hand-written baseline, a difference of **+0.16**.

The results should be interpreted as evidence about prioritization performance on this validation split. They do not establish causality, guarantee future production performance, or represent a prediction of Google's ranking algorithm.

---

## 1. Research Question

### Research Question

Can observed search-performance signals be combined into an interpretable model that prioritizes pages for human review more effectively than a simple hand-written baseline?

### Decision Supported

The analysis supports the decision of **which pages should be reviewed first** for possible content improvement.

The model is intended as a **decision-support tool for human review**, not as an autonomous system that changes or publishes content.

---

## 2. Data

This analysis uses the FlyRank ML Internship starter dataset:

`content_refresh_anonymized.csv`

The executed dataset contains **30,000 observations and 44 columns**.

The available fields include page-level content attributes and search-performance signals. The modelling experiment uses:

- CTR
- Average search position
- Search volume
- 90-day impressions
- 90-day clicks
- Content age in days

The client identifier used for grouped validation is `client_id`.

The target variable is:

`is_declining_label`

This label is created from `trend_direction`, where:

- `down` → `1`
- other observed values → `0`

The outcome-defining variables `trend_direction` and `trend_pct` are excluded from the model features to reduce leakage.

No client names, URLs, private search queries, page titles, or other identifying information are included in the reported results.

---

## 3. Methodology

### 3.1 Feature Selection

The model uses the following observed features:

| Feature | Description |
|---|---|
| `ctr` | Click-through rate |
| `avg_position` | Average search position |
| `search_volume` | Search demand signal |
| `impressions_90d` | 90-day impressions |
| `clicks_90d` | 90-day clicks |
| `content_age_days` | Content age |

These features were selected from the fields available in the internship dataset and were checked against the target definition.

### 3.2 Leakage Control

The target is derived from `trend_direction`.

Therefore, the following fields are forbidden as predictors:

- `trend_direction`
- `trend_pct`
- `is_declining_label`

The notebook performed an explicit leakage check and confirmed that none of these forbidden fields were included in the modelling feature list.

### 3.3 Validation Design

The data is divided using a client-level 80/20 holdout.

The executed split contained:

- **23,837 training observations**
- **6,163 test observations**
- **25 training clients**
- **7 test clients**

This design reduces the risk that observations from the same client appear in both the training and test sets.

### 3.4 Machine-Learning Model

The primary model is Logistic Regression.

The preprocessing pipeline performs:

1. Median imputation for missing feature values.
2. Standardization of numerical features.
3. Logistic Regression classification.

### 3.5 Baseline

The baseline is a simple hand-written prioritization rule that ranks observations with lower CTR more highly.

This provides an interpretable reference against which the learned model can be compared.

### 3.6 Evaluation Metric

The primary ranking metric is:

**Precision@50**

Precision@50 measures the proportion of truly declining observations among the 50 observations receiving the highest scores.

---

## 4. Results

### Model vs Baseline

| Method | Precision@50 |
|---|---:|
| Hand-written baseline | **0.52** |
| Logistic Regression | **0.68** |

### Observed Difference

The Logistic Regression model achieved:

**0.68 − 0.52 = +0.16**

Precision@50 improvement over the hand-written baseline on the same client-level held-out test set.

This means that, within this validation split, the model's top-50 review queue contained a higher proportion of observations labelled as declining than the baseline queue.

The result is an evaluation of prioritization performance on the executed validation split rather than a guarantee of future performance.

---

## 5. Interpretation

The experiment provides evidence that the available observed search-performance and content signals can be combined into a useful prioritization model.

The model is designed to answer:

> **Which pages should a human investigate first?**

rather than:

> **Which pages will definitely improve after an optimization?**

A high model score therefore indicates a higher estimated likelihood of belonging to the declining class under the learned model.

The result should be interpreted as **directional decision support**.

---

## 6. Ranked Recommendations

### 1. Review high-priority declining pages first

Start with pages receiving the highest model scores. These pages are the first candidates for investigation because the model estimates a higher likelihood of the declining label.

### 2. Investigate low-CTR pages

Review pages with low click-through rates, particularly when they continue to receive search impressions.

Potential areas for human investigation include title relevance, snippet quality, search intent alignment, and content relevance.

### 3. Review stale content

Pages with older content and declining signals should be checked for outdated information, missing updates, or changes in search intent.

### 4. Prioritize high-search-volume opportunities

When a declining page also has substantial search demand, it may deserve earlier investigation because it represents a larger amount of observed search activity.

### 5. Investigate conflicting signals manually

Pages with weak or conflicting signals should not be acted on automatically. Human review should determine whether an optimization is actually appropriate.

### Human Review Rule

The model score must not be used by itself to rewrite, delete, redirect, or publish a page.

A human should review the page, context, and supporting evidence before taking action.

---

## 7. Limitations

### Validation Design

The evaluation uses a client-level holdout rather than a fully time-based future prediction test. Therefore, the measured result should not be interpreted as guaranteed future performance.

### Feature Coverage

The model uses only the observed fields available in the internship dataset. Other factors that may influence search performance are not represented.

### Evaluation Cutoff

Precision@50 evaluates performance at one ranking cutoff and does not describe performance for every possible review-queue size.

### No Causal Conclusion

The analysis identifies patterns in the available data. It does not prove that changing a page will cause its search performance to improve.

### No Google Ranking Claim

The model should not be interpreted as a prediction of Google's ranking algorithm.

### Human Review Required

A high model score is a prioritization signal, not a final decision.

---

## 8. Reproducibility

The experiment is implemented in the accompanying Jupyter/Colab notebook:

`work/notebooks/capstone.ipynb`

The analysis uses Python and scikit-learn.

The notebook records:

- Dataset dimensions
- Feature selection
- Target construction
- Leakage controls
- Client-level train/test split
- Model training
- Baseline comparison
- Precision@50 evaluation
- Ranked human-review queue
- Experimental artifacts

The raw dataset is not redistributed through this repository.

---

## 9. Public-Safe Reporting

The reported results intentionally avoid publishing:

- Client names
- Domains
- URLs
- Private search queries
- Page titles
- Other identifying information

Only public-safe aggregate methodology and experimental results are reported.

---

## 10. Conclusion

This study tested whether observed search-performance signals could help prioritize potentially declining pages for human review.

The Logistic Regression model achieved a measured Precision@50 of **0.68**, compared with **0.52** for the simple hand-written baseline on the same client-level held-out test set.

The result provides evidence that the available signals can support a more effective prioritization workflow than the simple baseline within this validation experiment.

The findings should be interpreted carefully because the analysis is not a causal study, does not predict Google's ranking algorithm, and does not guarantee future production performance.

The resulting workflow is best used as **human decision-support**: the model identifies pages for investigation, while a human reviews the evidence and decides whether any content action is appropriate.

---

## 11. Acknowledgments & Data Credit

This work was developed as part of the **FlyRank ML Internship**.

The analysis uses the approved FlyRank ML Internship dataset:

`content_refresh_anonymized.csv`

Data credit:

[FlyRank](https://flyrank.ai)
