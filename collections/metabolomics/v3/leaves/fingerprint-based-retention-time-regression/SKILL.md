---
name: fingerprint-based-retention-time-regression
description: Use when you have small-molecule structures (SDF/molfiles) with computed
  ECFP fingerprints and experimentally acquired reverse-phase chromatography retention
  times, and need an ML model that predicts RT in seconds or ranks candidate identities
  by predicted RT.
when_to_use_negative:
- No molecular structures or fingerprints are available for the input compounds —
  the models take ECFP fingerprint vectors as their only features, so a structure-free
  compound list (e.g., only m/z values) cannot be used
- The target chromatography is not reverse-phase LC; the SMRT dataset is 'an experimentally
  acquired reverse-phase chromatography retention time dataset', so projecting predicted
  RT onto other chromatographic methods requires the separate transfer analysis, not
  direct application
- Only a tiny training set is available — the model is calibrated on a large-scale
  dataset (80,038 molecules) and reported error metrics assume that scale
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Keras for R
  role: Builds and trains the four-hidden-layer deep-learning regression model (1000/500/200/100
    relu units, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35) on ECFP
    fingerprints
- name: scikit-learn
  role: Trains the RandomForestRegressor baseline on ECFP fingerprints to reproduce
    the 66 s mean / 42 s median absolute error benchmark
- name: Dragon 7 (Kode Chemoinformatics)
  role: Calculates the ECFP fingerprints from molecular structures that serve as model
    input features
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:f5bbe4e7ae8a554e2767fca2ca24ba6f9b81710b50d8fc4584f3576e7e4293c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dragon 7 (Kode Chemoinformatics)
  - Keras for R
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fingerprint-based-retention-time-regression

## Summary

Trains machine-learning regression models (a Keras deep-learning model and a random-forest baseline) on molecular ECFP fingerprints from the METLIN SMRT dataset to predict reverse-phase LC retention times in seconds, reporting mean/median absolute and relative errors on a 25% validation split of the 80,038-molecule dataset.

## When to use

Use when you have small-molecule structures (SDF/molfiles) with computed ECFP fingerprints and experimentally acquired reverse-phase chromatography retention times, and need an ML model that predicts RT in seconds or ranks candidate identities by predicted RT.

## When NOT to use

- No molecular structures or fingerprints are available for the input compounds — the models take ECFP fingerprint vectors as their only features, so a structure-free compound list (e.g., only m/z values) cannot be used
- The target chromatography is not reverse-phase LC; the SMRT dataset is 'an experimentally acquired reverse-phase chromatography retention time dataset', so projecting predicted RT onto other chromatographic methods requires the separate transfer analysis, not direct application
- Only a tiny training set is available — the model is calibrated on a large-scale dataset (80,038 molecules) and reported error metrics assume that scale

## Inputs

- METLIN SMRT dataset: RT values in seconds for 80,038 small molecules
- ECFP molecular fingerprints calculated with Dragon 7
- Molecular structures (molfiles/SDF) and PubChem numbers keyed to each compound

## Outputs

- Trained Keras model saved as model.keras
- Validation error statistics (mean/median relative and absolute errors) as validation_errors.json
- Random-forest baseline mean and median absolute errors as rf_rt_errors.csv
- Per-molecule predicted RT and absolute errors (seconds) for the 25% validation set

## How to apply

Load the METLIN SMRT dataset (RT in seconds, PubChem numbers, molfiles/SDF structures, and ECFP fingerprints calculated with Dragon 7) and randomly split the 80,038 molecules into a 75% training set and 25% validation set. Build the Keras-for-R deep-learning regression model with four fully connected hidden layers of 1000, 500, 200, and 100 relu units, a single-unit linear output, L2 regularization (0.0001), the adam optimizer (learning rate 0.01), and mean squared error loss; train on ECFP inputs for 20 epochs with batch size 35. As a non-DL baseline, fit scikit-learn's RandomForestRegressor on the same fingerprints. Predict RT for the validation set, compute absolute errors |predicted - experimental| in seconds, and exclude non-retained molecules (those eluting within roughly the first minute before gradient start) when reporting retained-only statistics, consistent with the paper's treatment. The rationale is that fingerprints alone, at the scale of the 80,038-molecule SMRT dataset, are sufficient to rank candidate identities by predicted RT — in 70% of cases the correct identity appeared among the top 3 candidates.

## Related tools

- **Keras for R** (Builds and trains the four-hidden-layer deep-learning regression model (1000/500/200/100 relu units, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35) on ECFP fingerprints)
- **scikit-learn** (Trains the RandomForestRegressor baseline on ECFP fingerprints to reproduce the 66 s mean / 42 s median absolute error benchmark)
- **Dragon 7 (Kode Chemoinformatics)** (Calculates the ECFP fingerprints from molecular structures that serve as model input features)

## Examples

```
from sklearn.ensemble import RandomForestRegressor; import numpy as np; rf = RandomForestRegressor().fit(X_ecfp_train, rt_train); err = np.abs(rf.predict(X_ecfp_val) - rt_val); print(np.mean(err), np.median(err))
```

## Evaluation signals

- Deep-learning validation-set errors should match the reported 57 s mean and 35 s median absolute errors (47.8% mean, 4.6% median relative errors) within tolerance
- Random-forest baseline errors should match the reported 66 s mean and 42 s median absolute errors within reasonable tolerance
- Split integrity: exactly 75% of the 80,038 molecules in training and 25% in validation
- Non-retained molecules (eluting within roughly the first minute before gradient start) are excluded when computing retained-only error statistics, and included for the full-set figures
- Output artifacts exist and are well-formed: model.keras loads, validation_errors.json contains mean/median relative and absolute errors, rf_rt_errors.csv contains the two baseline metrics

## Limitations

- Relative-error statistics are strongly skewed by non-retained molecules with RT near the gradient start; the paper reports both full-set (47.8% mean / 4.6% median) and retained-only figures
- The model was trained on reverse-phase chromatography RTs; its predictive ability when projected onto other chromatographic methods was analyzed separately and is not guaranteed
- Absolute errors are in seconds and depend on the specific gradient/LC method used to acquire the METLIN SMRT dataset; transferring to other methods introduces additional error

## Evidence

- [abstract] SMRT dataset scale and chromatographic mode: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [abstract] Candidate ranking by predicted RT: "the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [intro] Dataset provenance: "a large-scale dataset consisting of experimentally acquired chromatographic RT covering 80,038 small molecules from the METLIN library"
- [intro] Model evaluation scope: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
