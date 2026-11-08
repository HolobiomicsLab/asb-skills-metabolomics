---
name: hilic-chromatography-rt-modeling
description: Use when you need to predict HILIC retention times for lipid species
  (class/subclass + fatty acid sum composition level) from an experimental HILIC-IM-MS
  lipid database, or to reproduce/reference RT prediction metrics like MAE 0.11 min.
when_to_use_negative:
- Your lipids belong to classes outside the 23 classes with experimental HILIC retention
  times in the database — the model is only trained on those classes
- You need CCS prediction rather than HILIC retention time prediction (use the separate
  CCS model, trained for 22 lipid classes)
- Input is raw IM-MS data or feature lists needing lipid identification — use the
  lipydomics `identification` module instead of the RT prediction model
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Python package whose `data` module stores the lipid database (including the
    HILIC retention time subset) from which training features and targets are drawn
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Fits the multivariate linear regression used to predict HILIC retention time
    from the 26-feature lipid encoding
- name: Python
  role: Implementation language for the lipydomics package and the model training/validation
    script
  repo: https://github.com/dylanhross/lipydomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:eacd1943951529c469b523bdd503dab12cacb6b94ff2f1dfe14f261d2ac8570c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hilic-chromatography-rt-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and validate a multivariate linear regression that predicts HILIC chromatographic retention times of lipids from a 26-feature encoding of lipid class, fatty acid modifier, and sum composition, using the lipydomics experimental lipid database and scikit-learn.

## When to use

Use when you need to predict HILIC retention times for lipid species (class/subclass + fatty acid sum composition level) from an experimental HILIC-IM-MS lipid database, or to reproduce/reference RT prediction metrics like MAE 0.11 min.

## When NOT to use

- Your lipids belong to classes outside the 23 classes with experimental HILIC retention times in the database — the model is only trained on those classes
- You need CCS prediction rather than HILIC retention time prediction (use the separate CCS model, trained for 22 lipid classes)
- Input is raw IM-MS data or feature lists needing lipid identification — use the lipydomics `identification` module instead of the RT prediction model

## Inputs

- 596-lipid HILIC retention time subset from the lipydomics experimental lipid database (23 of 45 lipid classes have HILIC retention times)
- 26-feature encoding per lipid: lipid class, fatty acid modifier, and sum composition
- Documented training/test partition of the encoded subset

## Outputs

- Fitted scikit-learn multivariate linear regression model serialized via pickle
- Test-set metrics table (MAE 0.11 min, MDAE 0.08 min, RMSE 0.15 min) e.g. rt_model_metrics.csv
- Predicted HILIC retention times in minutes for held-out lipids

## How to apply

Load the 596-lipid HILIC retention time training subset from the lipid database stored in the lipydomics `data` module, which covers the 23 lipid classes that have experimental HILIC retention times in the 45-class m/z and CCS database. Encode each lipid as a 26-feature vector covering lipid class, fatty acid modifier, and sum composition — the 'simple but specialized feature sets' the authors used for ML prediction. Partition into documented training and held-out test subsets, then fit a scikit-learn multivariate linear regression on the training subset. Score the test set in minutes and record MAE, MDAE, and RMSE; the reference model achieves 0.11, 0.08, and 0.15 min respectively. Serialize the fitted model with pickle and export the metrics table (e.g., rt_model_metrics.csv) so test-set metrics can be validated against the reported values.

## Related tools

- **LiPydomics** (Python package whose `data` module stores the lipid database (including the HILIC retention time subset) from which training features and targets are drawn) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Fits the multivariate linear regression used to predict HILIC retention time from the 26-feature lipid encoding)
- **Python** (Implementation language for the lipydomics package and the model training/validation script) — https://github.com/dylanhross/lipydomics

## Examples

```
from sklearn.linear_model import LinearRegression; import pickle; model = LinearRegression().fit(X_26feat, rt_min); pickle.dump(model, open('rt_model.pkl','wb'))
```

## Evaluation signals

- Held-out test-set MAE, MDAE, and RMSE fall within tolerance of the reported 0.11, 0.08, and 0.15 min
- Training set contains exactly the 596-lipid HILIC retention time subset drawn from the lipydomics `data` module database
- Each lipid is represented by the 26-feature encoding (lipid class, fatty acid modifier, sum composition) before regression
- Fitted model is serializable via pickle and metrics are written to a metrics table (rt_model_metrics.csv)

## Limitations

- Prediction models were trained only for the 23 lipid classes that have experimental HILIC retention times (22 classes for CCS), so coverage is bounded by the experimental database
- The database is experimental; the package (including the lipid database and prediction models) is 'built to be highly extensible and customizable, allowing easy expansion as more data becomes available', implying current coverage gaps in lipid chemical space
- The regression is a linear model on simple specialized features; it supports Level 3-style identifications (lipid class/subclass and fatty acid sum composition), not more granular structural resolution

## Evidence
