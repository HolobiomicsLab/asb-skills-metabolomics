---
name: hilic-retention-time-prediction
description: Use when HILIC-IM-MS lipidomics features have measured retention times
  and you need to match them against predicted HILIC retention times for lipid classes
  that lack experimental RT entries, or when regenerating/extending the LiPydomics
  RT prediction models.
when_to_use_negative:
- The analytes are separated by reversed-phase LC rather than HILIC — the models are
  trained only on HILIC retention times and will not transfer
- You need CCS predictions instead of retention times — use the 22-class CCS prediction
  models, which are trained separately
- The lipid class of interest is among the 45 experimental database classes lacking
  RT models (only 23 classes have trained RT models), with no training data available
  to extend the model
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LiPydomics
  role: Provides the lipid database (596-lipid HILIC RT training subset in the `data`
    module) and the feature/identification schema into which predicted RTs are written
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Fits and scores the multivariate linear regression used for HILIC retention
    time prediction
- name: Python
  role: Execution environment for the lipydomics API, model training, and database
    serialization
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:d265888e0ac7df255af5a62c4bcf9a2850525ec8f6c108015ab3efb72a3f54fc
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
  - LipidCCS
  - Python
  - SciPy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hilic-retention-time-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict HILIC retention times for lipid species using class-specific multivariate linear regression models trained on the LiPydomics 596-lipid experimental HILIC-IM-MS reference subset, enabling retention-time-guided lipid identification in the 145,388-species predicted database.

## When to use

Use when HILIC-IM-MS lipidomics features have measured retention times and you need to match them against predicted HILIC retention times for lipid classes that lack experimental RT entries, or when regenerating/extending the LiPydomics RT prediction models.

## When NOT to use

- The analytes are separated by reversed-phase LC rather than HILIC — the models are trained only on HILIC retention times and will not transfer
- You need CCS predictions instead of retention times — use the 22-class CCS prediction models, which are trained separately
- The lipid class of interest is among the 45 experimental database classes lacking RT models (only 23 classes have trained RT models), with no training data available to extend the model

## Inputs

- 596-lipid HILIC retention time training subset from the lipydomics `data` module lipid database
- 26-feature per-lipid encoding (lipid class, fatty acid modifier, sum composition)
- Enumerated lipid species from the predicted database (fatty acyl chains 10–30 carbons, even and odd; 0–6 degrees of unsaturation; 31 LipidMass-defined lipid classes)

## Outputs

- Fitted scikit-learn multivariate linear regression model (pickled)
- rt_model_metrics.csv with test-set MAE, MDAE, and RMSE in minutes
- Predicted HILIC retention time values (106,020 across 23 lipid classes) written into SQLite/JSON database tables matching the package schema

## How to apply

Load the 596-lipid HILIC retention time training subset from the lipid database stored in the lipydomics `data` module. Encode each lipid into the 26-feature representation covering lipid class, fatty acid modifier, and sum composition. Partition into the documented training/test subsets, then fit a scikit-learn multivariate linear regression to predict HILIC retention time from the 26 features; the same feature scheme (fatty acyl chain length, unsaturation, class) is used to populate predicted RTs for the 23 lipid classes having trained RT models within the predicted database of 145,388 lipid species. Score the held-out test set and compare MAE, MDAE, and RMSE (in minutes) against the reported 0.11/0.08/0.15 min; serialize the fitted model with pickle and write metrics to rt_model_metrics.csv. Rationale: experimental RT collections do not cover the full lipid chemical space, so ML-predicted RTs (106,020 predicted HILIC RT values across 23 classes) fill the gaps and support Level 3 identification on HILIC-IM-MS data.

## Related tools

- **LiPydomics** (Provides the lipid database (596-lipid HILIC RT training subset in the `data` module) and the feature/identification schema into which predicted RTs are written) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Fits and scores the multivariate linear regression used for HILIC retention time prediction)
- **Python** (Execution environment for the lipydomics API, model training, and database serialization)

## Examples

```
from sklearn.linear_model import LinearRegression; import pickle; model = LinearRegression().fit(X26, rt_hilic); pickle.dump(model, open('rt_model.pkl','wb'))
```

## Evaluation signals

- Test-set MAE/MDAE/RMSE fall within tolerance of the reported 0.11/0.08/0.15 min
- Predicted RTs are populated for all 23 lipid classes with trained RT models and none of the 22 CCS-only classes
- Database totals match the reported 106,020 predicted HILIC RT values within the 145,388-species database
- Training subset size equals the documented 596 lipids with HILIC retention times
- Serialized model reproduces the metrics table in rt_model_metrics.csv when re-scored on the held-out test subset

## Limitations

- RT models exist for only 23 of the 45 lipid classes in the experimental database; species in other classes cannot receive predicted RTs
- Predictions cover the enumerated chemical space only (fatty acyl chains of 10–30 carbons, both even and odd, with 0–6 degrees of unsaturation)
- Multivariate linear regression is a simple, specialized feature-set model; accuracy for out-of-scope or unusual lipid compositions is uncharacterized

## Evidence
