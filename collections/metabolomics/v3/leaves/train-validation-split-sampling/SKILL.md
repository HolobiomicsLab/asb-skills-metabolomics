---
name: train-validation-split-sampling
description: 'Use when reproducing the METLIN SMRT RT-prediction workflow: a large
  RT-labeled molecule table (80,038 compounds) must be partitioned before fitting
  a Keras regression model and reporting validation-set error metrics.'
when_to_use_negative:
- Your goal is to train on all available data for final deployment — the 75/25 split
  intentionally withholds 25% of molecules for validation and is meant for error estimation,
  not maximally fitted models.
- Your dataset is much smaller than 80,038 molecules or comes from a different chromatographic
  method; the paper itself questions whether the model scales to other chromatographic
  methods.
- You need candidate ranking/filtration metrics (e.g., top-3 identification rate)
  rather than split-based regression error evaluation.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Keras for R
  role: Implements the deep-learning regression model that is trained on the 75% split
    and evaluated on the 25% validation split; model is serialized as model.keras.
- name: Dragon 7 (Kode Chemoinformatics)
  role: Computes the ECFP fingerprints from molecular structures (molfiles/SDF) that
    serve as model inputs across both the training and validation partitions.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:76e87a57fbc428eabfb57a1072f9a9b52dec90267237a51ec6d810eb4c043b09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dragon 7 (Kode Chemoinformatics)
  - Keras for R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# train-validation-split-sampling

## Summary

Splits the METLIN SMRT retention time dataset into a 75% training / 25% validation partition so a Keras deep-learning regression model can be trained on ECFP fingerprints and evaluated on held-out molecules.

## When to use

Use when reproducing the METLIN SMRT RT-prediction workflow: a large RT-labeled molecule table (80,038 compounds) must be partitioned before fitting a Keras regression model and reporting validation-set error metrics.

## When NOT to use

- Your goal is to train on all available data for final deployment — the 75/25 split intentionally withholds 25% of molecules for validation and is meant for error estimation, not maximally fitted models.
- Your dataset is much smaller than 80,038 molecules or comes from a different chromatographic method; the paper itself questions whether the model scales to other chromatographic methods.
- You need candidate ranking/filtration metrics (e.g., top-3 identification rate) rather than split-based regression error evaluation.

## Inputs

- METLIN SMRT dataset with RT values in seconds for 80,038 small molecules (PubChem numbers, molfiles/SDF structures)
- ECFP molecular fingerprints calculated with Dragon 7
- Molecule-to-RT mapping table for all 80,038 compounds

## Outputs

- 75% training partition and 25% validation partition of the 80,038-molecule dataset
- Trained Keras model saved as model.keras
- Validation error statistics saved as validation_errors.json (mean/median relative errors 47.8%/4.6%, mean/median absolute errors 57 s/35 s)

## How to apply

Load the METLIN SMRT dataset (RT in seconds, PubChem numbers, molfiles/SDF structures, and ECFP fingerprints calculated with Dragon 7) and randomly split the 80,038 molecules into a 75% training set and a 25% validation set. Train the Keras regression model — four fully connected hidden layers of 1000, 500, 200, and 100 relu units, a single-unit linear output, L2 regularization of 0.0001, adam optimizer with learning rate 0.01, MSE loss — for 20 epochs with batch size 35, using only the training partition. Predict RT for the held-out 25% and exclude non-retained molecules (eluting within roughly the first minute before gradient start) when computing overall error statistics, matching the paper's treatment. The rationale for a single random 75/25 split is that the paper evaluates generalization error on molecules the model never saw, without a separate test set.

## Related tools

- **Keras for R** (Implements the deep-learning regression model that is trained on the 75% split and evaluated on the 25% validation split; model is serialized as model.keras.)
- **Dragon 7 (Kode Chemoinformatics)** (Computes the ECFP fingerprints from molecular structures (molfiles/SDF) that serve as model inputs across both the training and validation partitions.)

## Examples

```
set.seed(1); idx <- sample(seq_len(80038)); train_idx <- idx[1:60028]; val_idx <- idx[60029:80038]; model <- keras_model_sequential() %>% layer_dense(1000, activation='relu', kernel_regularizer=regularizer_l2(1e-4)) %>% layer_dense(500, activation='relu') %>% layer_dense(200, activation='relu') %>% layer_dense(100, activation='relu') %>% layer_dense(1); model %>% compile(optimizer=optimizer_adam(lr=0.01), loss='mse'); model %>% fit(ecfp[train_idx,], rt[train_idx], epochs=20, batch_size=35); save_model_hdf5(model, 'model.keras')
```

## Evaluation signals

- Partition sizes sum to 80,038 with the training set at 75% and the validation set at 25%; no molecule appears in both partitions.
- Validation-set absolute errors approximate the reported mean 57 s and median 35 s.
- Validation-set relative errors approximate the reported mean 47.8% and median 4.6%.
- Non-retained molecules eluting within roughly the first minute before gradient start are excluded when computing overall error statistics.
- validation_errors.json is produced alongside model.keras with matching error fields for the full validation set including non-retained molecules.

## Limitations

- Non-retained molecules (eluting within roughly the first minute before gradient start) must be excluded from overall error statistics, consistent with the paper's treatment, or errors will not match reported values.
- The reported relative-error distribution is skewed (mean 47.8% vs median 4.6%), so mean relative error is dominated by outliers and is a fragile comparison metric.
- The model's ability to project predicted RT onto other chromatographic methods was analyzed but is a separate question; validation errors are specific to the reverse-phase conditions under which the 80,038 RTs were acquired.

## Evidence

- [abstract] SMRT dataset definition: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [intro] Dataset scale from METLIN library: "a large-scale dataset consisting of experimentally acquired chromatographic RT covering 80,038 small molecules from the METLIN library"
- [other] Split and evaluation workflow: [evidence span withheld — anchor status: not_found]
- [other] Reported validation errors: [evidence span withheld — anchor status: not_found]
