---
name: multivariate-linear-regression-model-training
description: Use when you have an assembled experimental lipid database (e.g., m/z,
  CCS, HILIC retention time entries across lipid classes) with gaps in chemical space
  and need to train regression models to predict CCS or HILIC retention times for
  unseen lipid species.
when_to_use_negative:
- When the lipid classes of interest already have comprehensive experimental CCS or
  retention time coverage — the models exist to fill gaps, not to re-predict known
  values.
- When input is raw MS/IM-MS data rather than a curated lipid database with measured
  retention times/CCS — use the stats/plotting/identification modules instead.
- When lipid species lack representation in the 45-lipid-class experimental database
  (e.g., unannotated bacterial lipid subclasses) — feature encoding and model transfer
  will be invalid.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the `data` module holding the lipid database/training subsets and
    the extensible framework into which the trained models are serialized; packages
    the full pipeline.
  repo: github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Fits the multivariate linear regression models for CCS and HILIC retention
    time prediction on the encoded lipid features.
- name: Python
  role: Implementation language for the pipeline; LiPydomics is an open-source Python
    package with pickle-based model serialization.
  repo: https://github.com/dylanhross/lipydomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:02537c226b4ee8f238e7974e6e747d0f3da6a348578a0542ee7793c5c80be5e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multivariate-linear-regression-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains multivariate linear regression models (e.g., via scikit-learn) on specialized lipid feature sets — covering lipid class, fatty acid modifier, and sum composition — to predict HILIC retention times or CCS values, and evaluates them on held-out test data with MAE, MDAE, and RMSE metrics.

## When to use

Use when you have an assembled experimental lipid database (e.g., m/z, CCS, HILIC retention time entries across lipid classes) with gaps in chemical space and need to train regression models to predict CCS or HILIC retention times for unseen lipid species.

## When NOT to use

- When the lipid classes of interest already have comprehensive experimental CCS or retention time coverage — the models exist to fill gaps, not to re-predict known values.
- When input is raw MS/IM-MS data rather than a curated lipid database with measured retention times/CCS — use the stats/plotting/identification modules instead.
- When lipid species lack representation in the 45-lipid-class experimental database (e.g., unannotated bacterial lipid subclasses) — feature encoding and model transfer will be invalid.

## Inputs

- experimental lipid database subset (e.g., 596-lipid HILIC retention time entries or CCS entries across lipid classes) with measured target values
- lipid feature encoding vectors (26 features: lipid class, fatty acid modifier, sum composition)
- train/test partition specification

## Outputs

- fitted multivariate linear regression model (serialized pickle)
- test-set regression metrics table (MAE, MDAE, RMSE) e.g. rt_model_metrics.csv
- predicted HILIC retention times or CCS values for new lipid species

## How to apply

Load the relevant subset of the experimental lipid database — e.g., the 596-lipid HILIC retention time training subset from the 145,388-entry predicted lipid database or the 23 classes containing HILIC retention times. Encode each lipid into a specialized feature set (26 features covering lipid class, fatty acid modifier, and sum composition) rather than raw chemical descriptors, per the paper's 'simple but specialized feature sets'. Partition data into training and held-out test subsets, fit a scikit-learn multivariate linear regression, then score the test set recording MAE, MDAE, and RMSE in minutes — for the HILIC retention time reference model these were 0.11, 0.08, and 0.15 min respectively. Serialize the fitted model (pickle) and write metrics to a CSV for downstream identification or database expansion.

## Related tools

- **LiPydomics** (Provides the `data` module holding the lipid database/training subsets and the extensible framework into which the trained models are serialized; packages the full pipeline.) — github.com/dylanhross/lipydomics
- **scikit-learn** (Fits the multivariate linear regression models for CCS and HILIC retention time prediction on the encoded lipid features.)
- **Python** (Implementation language for the pipeline; LiPydomics is an open-source Python package with pickle-based model serialization.) — https://github.com/dylanhross/lipydomics

## Examples

```
from lipydomics import data; from sklearn.linear_model import LinearRegression; X_train, y_train = data.load_hilic_rt_subset(n=596); X_test, y_test = data.load_hilic_rt_test(); reg = LinearRegression().fit(X_train, y_train); import pickle; pickle.dump(reg, open('rt_model.pkl','wb'))
```

## Evaluation signals

- Test-set metrics must match the reported reference values: HILIC retention time model yields MAE = 0.11 min, MDAE = 0.08 min, RMSE = 0.15 min on held-out data.
- Serialized pickle of the fitted model must be loadable and reproduce predictions consistent with rt_model_metrics.csv entries.
- Feature count check: each encoded lipid row must contain exactly 26 features (lipid class, fatty acid modifier, sum composition).
- Training set must correspond to the documented 596-lipid HILIC subset (or the analogous CCS subset across 22 lipid classes); mismatched lipid-class coverage indicates wrong partition.
- Metrics CSV must contain all three columns (MAE, MDAE, RMSE) in minutes with values within tolerance of the paper-reported reference.

## Limitations

- Models were trained on a specific 22 (CCS) / 23 (HILIC retention time) lipid-class collection; accuracy degrades for classes outside this experimental database.
- The database/feature encoding is 'highly extensible and customizable' — retargeting it to new lipid classes or modifications requires retraining and revalidation.
- Linear regression on simple feature sets may underfit complex lipid structures; the paper does not report non-linear alternatives.

## Evidence

- [supplementary] Regression metrics reference (MAE/MDAE/RMSE) from the source task card: [evidence span withheld — anchor status: not_found]
