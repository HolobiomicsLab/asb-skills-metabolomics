---
name: quantitative-structure-retention-relationship-modeling
description: Use when you need to predict reverse-phase LC-MS retention times from
  molecular structures (e.g., molfiles/SDF, PubChem numbers) or rank/filter putative
  annotations from accurate-mass search by predicted RT, and you have structural fingerprints
  as model inputs.
when_to_use_negative:
- Retention times were measured on a different chromatographic method than the reverse-phase
  conditions the SMRT dataset was acquired under, and you have no means to project/transform
  predicted RT onto that method
- The task is spectra prediction (e.g., MS/MS in silico spectra) rather than chromatographic
  RT prediction; this skill models RT only
- Structural fingerprints cannot be computed for the query molecules (no usable molfile/SDF
  or PubChem identifier), so no ECFP input exists for the model
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Keras for R
  role: Framework for constructing and training the four-layer fully connected regression
    network (1000/500/200/100 relu units, linear output, L2 regularization 0.0001,
    adam optimizer lr 0.01, MSE loss, 20 epochs, batch size 35) and saving it as model.keras
- name: Dragon 7 (Kode Chemoinformatics)
  role: Used to calculate the ECFP fingerprints from molecular structures that serve
    as the model's input features
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:b391a2ef1a7142e5eef343c0a04bd5735ea63ca0676770b9a436a81ca59c354d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dragon 7 (Kode Chemoinformatics)
  - Keras for R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# quantitative-structure-retention-relationship-modeling

## Summary

Train a deep-learning quantitative structure-retention relationship (QSRR) model that predicts reverse-phase LC chromatographic retention time (RT) from molecular ECFP fingerprints, using the METLIN SMRT dataset of 80,038 small molecules and a four-layer Keras regression network.

## When to use

Use when you need to predict reverse-phase LC-MS retention times from molecular structures (e.g., molfiles/SDF, PubChem numbers) or rank/filter putative annotations from accurate-mass search by predicted RT, and you have structural fingerprints as model inputs.

## When NOT to use

- Retention times were measured on a different chromatographic method than the reverse-phase conditions the SMRT dataset was acquired under, and you have no means to project/transform predicted RT onto that method
- The task is spectra prediction (e.g., MS/MS in silico spectra) rather than chromatographic RT prediction; this skill models RT only
- Structural fingerprints cannot be computed for the query molecules (no usable molfile/SDF or PubChem identifier), so no ECFP input exists for the model

## Inputs

- METLIN SMRT dataset: experimentally acquired reverse-phase chromatography RT values in seconds for 80,038 small molecules
- Molecular structures as molfiles/SDF with PubChem numbers
- ECFP fingerprints calculated with Dragon 7
- 75%/25% random training/validation split definition

## Outputs

- Trained Keras RT regression model saved as model.keras
- Per-molecule RT predictions for the validation set
- Validation error statistics saved as validation_errors.json (mean/median absolute errors in seconds; mean/median relative errors)
- RT-ranked candidate lists for downstream annotation filtering

## How to apply

Load the METLIN SMRT dataset (RT values in seconds, PubChem numbers, molfiles/SDF structures) and compute ECFP fingerprints (as done with Dragon 7) for the 80,038 molecules, then randomly split into a 75% training set and 25% validation set. Build the regression model in Keras for R with four fully connected hidden layers of 1000, 500, 200, and 100 relu-activated units, a single-unit linear output, L2 regularization (parameter 0.0001), the adam optimizer with learning rate 0.01, and mean squared error as the loss function. Train on the ECFP inputs for 20 epochs with a batch size of 35. Predict RT for the validation set; when computing error statistics, exclude non-retained molecules eluting within roughly the first minute before gradient start, consistent with the paper's treatment. Evaluate with mean/median absolute and relative errors (reference values: 57 s / 35 s absolute, 47.8% / 4.6% relative), save the trained model as model.keras and statistics as validation_errors.json.

## Related tools

- **Keras for R** (Framework for constructing and training the four-layer fully connected regression network (1000/500/200/100 relu units, linear output, L2 regularization 0.0001, adam optimizer lr 0.01, MSE loss, 20 epochs, batch size 35) and saving it as model.keras)
- **Dragon 7 (Kode Chemoinformatics)** (Used to calculate the ECFP fingerprints from molecular structures that serve as the model's input features)

## Examples

```
R: model <- keras_model_sequential() %>% layer_dense(1000, activation='relu', kernel_regularizer=regularizer_l2(0.0001)) %>% layer_dense(500, activation='relu') %>% layer_dense(200, activation='relu') %>% layer_dense(100, activation='relu') %>% layer_dense(1); model %>% compile(optimizer=optimizer_adam(lr=0.01), loss='mse') %>% fit(ecfp_train, rt_train, epochs=20, batch_size=35)
```

## Evaluation signals

- Validation-set mean absolute error is 57 s and median absolute error is 35 s on the full validation set including non-retained molecules
- Validation-set mean relative error is 47.8% and median relative error is 4.6% (median relative error far lower than mean, reflecting heavy-tailed error on a minority of molecules)
- Split sizes check out: ~60,028 training molecules and ~20,010 validation molecules from the 80,038-molecule SMRT dataset (75%/25%)
- Model architecture check: 4 hidden dense layers of 1000, 500, 200, 100 units, relu activation, L2 0.0001, adam lr 0.01, MSE loss, trained 20 epochs at batch size 35
- Downstream utility check: in ~70% of cases the correct molecular identity is ranked among the top 3 candidates when candidates are ranked by predicted RT

## Limitations

- Non-retained molecules (eluting within roughly the first minute before gradient start) inflate error statistics and are excluded when computing overall error statistics, consistent with the paper's treatment
- The model was trained specifically on experimentally acquired reverse-phase chromatography RT; applying it to other chromatographic methods requires projection of predicted RT onto those methods, which is a separate step
- Mean relative error (47.8%) is much higher than the median (4.6%), indicating a minority of molecules with very large relative errors

## Evidence

- [abstract] SMRT dataset abstract evidence: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [abstract] candidate ranking finding: "in 70\documentclass[12pt]{minimal} ... \begin{document}$$\%$$\end{document}% of the cases, the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [intro] intro scope statement: "a large-scale dataset consisting of experimentally acquired chromatographic RT covering 80,038 small molecules from the METLIN library"
