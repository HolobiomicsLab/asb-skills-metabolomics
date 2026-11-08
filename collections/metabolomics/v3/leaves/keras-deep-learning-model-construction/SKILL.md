---
name: keras-deep-learning-model-construction
description: Use when building a regression model to predict continuous molecular
  properties (e.g., retention time in seconds) from binary molecular fingerprints
  (ECFP), following the METLIN SMRT deep learning model (DLM) architecture.
when_to_use_negative:
- The prediction target is categorical (e.g., compound identity ranking) rather than
  a continuous RT value in seconds — the single-unit linear output and MSE loss described
  here are for regression only.
- Input features are not molecular fingerprints (e.g., raw spectra or chromatographic
  peaks); this architecture was designed for ECFP fingerprint inputs.
- A pretrained RT prediction model is already available and only inference is needed
  — rebuilding and retraining the network is unnecessary.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: Keras for R
  role: Framework used to construct and train the deep-learning regression model (four
    hidden layers of 1000/500/200/100 relu units, L2 regularization 0.0001, adam optimizer
    lr 0.01, MSE loss, 20 epochs, batch size 35).
- name: Dragon 7 (Kode Chemoinformatics)
  role: Software used to calculate the ECFP molecular fingerprints that serve as model
    input features.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:0fa05ec4ad0e41cd92ad6c0eda8e6d3e355a8551f2df927800261a4c2dde24aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dragon 7 (Kode Chemoinformatics)
  - Keras for R
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# keras-deep-learning-model-construction

## Summary

Construct and train a Keras deep-learning regression model that predicts LC-MS retention times from ECFP molecular fingerprints, as demonstrated on the METLIN SMRT dataset of 80,038 small molecules with reverse-phase chromatography RTs.

## When to use

Use when building a regression model to predict continuous molecular properties (e.g., retention time in seconds) from binary molecular fingerprints (ECFP), following the METLIN SMRT deep learning model (DLM) architecture.

## When NOT to use

- The prediction target is categorical (e.g., compound identity ranking) rather than a continuous RT value in seconds — the single-unit linear output and MSE loss described here are for regression only.
- Input features are not molecular fingerprints (e.g., raw spectra or chromatographic peaks); this architecture was designed for ECFP fingerprint inputs.
- A pretrained RT prediction model is already available and only inference is needed — rebuilding and retraining the network is unnecessary.

## Inputs

- METLIN SMRT dataset with RT values in seconds for 80,038 small molecules (PubChem numbers, molfiles/SDF structures)
- ECFP molecular fingerprints calculated with Dragon 7
- 75/25 random train/validation split assignment

## Outputs

- Trained Keras regression model saved as model.keras
- Validation-set predictions of retention time in seconds
- Validation error statistics saved as validation_errors.json (mean/median absolute errors and mean/median relative errors)

## How to apply

Load molecular structures and compute ECFP fingerprints (the paper used Dragon 7), then randomly split the dataset (e.g., 80,038 METLIN SMRT molecules) into 75% training and 25% validation sets. In Keras for R, build a regression network of four fully connected hidden layers with 1000, 500, 200, and 100 units activated by relu, a single-unit linear output, L2 regularization with parameter 0.0001, the adam optimizer with learning rate 0.01, and mean squared error as the loss. Train on the ECFP inputs with 20 epochs and batch size 35, then predict RT for the held-out validation set. Evaluate with mean/median absolute errors (target: 57 s / 35 s) and mean/median relative errors (target: 47.8% / 4.6%), excluding non-retained molecules eluting within roughly the first minute before gradient start when computing retained-set statistics, and save the fitted model (e.g., model.keras) plus error statistics (e.g., validation_errors.json).

## Related tools

- **Keras for R** (Framework used to construct and train the deep-learning regression model (four hidden layers of 1000/500/200/100 relu units, L2 regularization 0.0001, adam optimizer lr 0.01, MSE loss, 20 epochs, batch size 35).)
- **Dragon 7 (Kode Chemoinformatics)** (Software used to calculate the ECFP molecular fingerprints that serve as model input features.)

## Examples

```
Rscript train_dlm.R --input smrt_ecfp.csv --split 0.75 --layers 1000,500,200,100 --activation relu --l2 0.0001 --lr 0.01 --epochs 20 --batch_size 35 --out model.keras --errors validation_errors.json
```

## Evaluation signals

- Validation-set mean absolute error should match the reported 57 s and median absolute error 35 s within tolerance.
- Validation-set relative errors should match the reported mean 47.8% and median 4.6% within tolerance.
- Dataset split must be a random 75% training / 25% validation partition of the 80,038 molecules.
- Model architecture check: exactly four hidden layers (1000, 500, 200, 100 units, relu), linear single-unit output, L2 regularization parameter 0.0001, adam with learning rate 0.01, MSE loss, 20 epochs, batch size 35.
- Non-retained molecules (eluting within roughly the first minute before gradient start) should be excluded when computing retained-molecule error statistics, consistent with the paper's treatment.

## Limitations

- Relative-error statistics are heavily skewed: the mean relative error (47.8%) far exceeds the median (4.6%), so mean relative error alone can misrepresent typical performance.
- Non-retained molecules eluting near the gradient start inflate error statistics unless excluded, as in the paper's treatment.
- Model performance depends on fingerprints computed with a specific tool (Dragon 7); different fingerprint implementations may not reproduce the reported errors.

## Evidence

- [abstract] SMRT dataset coverage: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [abstract] Candidate ranking performance: "in 70\% of the cases, the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [methods] Task card workflow (training and evaluation): [evidence span withheld — anchor status: not_found]
- [methods] Task card workflow (evaluation): [evidence span withheld — anchor status: not_found]
- [results] Task card reported metrics: [evidence span withheld — anchor status: not_found]
