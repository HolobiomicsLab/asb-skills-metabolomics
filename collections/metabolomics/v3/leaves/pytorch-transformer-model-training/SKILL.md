---
name: pytorch-transformer-model-training
description: Use when a large-scale tabular omics matrix (e.g., 168-dim NMR metabolomics
  per participant) must be pre-trained into a foundation model via masked-feature
  reconstruction before downstream fine-tuning.
when_to_use_negative:
- The input already contains disease labels and the goal is supervised prediction
  — use fine-tuning (e.g., the repo's train-binary CLI for current/future disease
  prediction) instead of masked pre-training.
- The metabolite panel differs from the 168 NMR biomarkers the pre-trained model expects
  (the companion repo lists expected columns in assets/pretrained/metabolite_columns.txt);
  retraining from scratch would be needed.
- Only a handful of samples are available — transformer pre-training on 83,744 individuals
  relies on large-scale population data to capture normative metabolite patterns.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: PyTorch
  role: Implements the BERT-style transformer encoder, Hadamard-product embedding,
    masked-metabolite reconstruction loss, and AdamW training loop
- name: pandas
  role: Loads the NMR metabolomics feature matrix and partitions it into training/validation
    sets
- name: NumPy
  role: Applies the Hadamard-product feature embedding and random 10% masking of metabolite
    positions
- name: scikit-learn
  role: 'Computes validation metrics: MSE, MAE, R², and reconstruction accuracy'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:f415c03b76e96ff816ffc18d56ff8d4ee33b9aaedf76dde58bb8aa8a1909dea4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - NumPy
  - PyTorch
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# pytorch-transformer-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a BERT-style transformer encoder in PyTorch on tabular biomarker data using masked-feature reconstruction: here, Hadamard-product embedding of 168 NMR metabolite biomarkers from 83,744 UK Biobank individuals with a [CLS] token and 10% random metabolite masking, optimized with AdamW for 200 epochs.

## When to use

Use when a large-scale tabular omics matrix (e.g., 168-dim NMR metabolomics per participant) must be pre-trained into a foundation model via masked-feature reconstruction before downstream fine-tuning.

## When NOT to use

- The input already contains disease labels and the goal is supervised prediction — use fine-tuning (e.g., the repo's train-binary CLI for current/future disease prediction) instead of masked pre-training.
- The metabolite panel differs from the 168 NMR biomarkers the pre-trained model expects (the companion repo lists expected columns in assets/pretrained/metabolite_columns.txt); retraining from scratch would be needed.
- Only a handful of samples are available — transformer pre-training on 83,744 individuals relies on large-scale population data to capture normative metabolite patterns.

## Inputs

- Feature matrix of 168 NMR metabolite biomarkers per participant for 83,744 relatively healthy UK Biobank individuals (CSV/TSV loadable with pandas)
- Train/validation split definition
- Pre-training hyperparameters: masking rate 10%, AdamW lr 1e-4, 200 epochs

## Outputs

- Pre-trained transformer weights serialized as metabolm_pretrained.pt
- Metrics dictionary (loss, reconstruction accuracy, MSE, MAE, R2) written to pretraining_metrics
- Encoder checkpoint reusable for fine-tuning on 16 chronic-disease prediction tasks and MetaboRS risk-score derivation

## How to apply

Load the NMR metabolomics feature matrix (168 metabolite biomarkers per participant) with pandas/NumPy and split into training and validation sets. Apply the Hadamard-product feature embedding to the 168-dim metabolite vector and prepend a [CLS] token to form the transformer input sequence. Randomly mask 10% of metabolite feature positions per sample to create the masked-metabolite reconstruction objective. Train the BERT-style encoder in PyTorch with AdamW at learning rate 1e-4 for 200 epochs, minimizing reconstruction loss on masked positions only. Evaluate on the validation split and confirm convergence metrics match the reference: loss 0.0684, reconstruction accuracy 0.9533, MSE 0.0686, MAE 0.1291, R² 0.9305. Serialize weights (e.g., metabolm_pretrained.pt) and the metrics dictionary for downstream fine-tuning on the 16 chronic-disease tasks.

## Related tools

- **PyTorch** (Implements the BERT-style transformer encoder, Hadamard-product embedding, masked-metabolite reconstruction loss, and AdamW training loop)
- **pandas** (Loads the NMR metabolomics feature matrix and partitions it into training/validation sets)
- **NumPy** (Applies the Hadamard-product feature embedding and random 10% masking of metabolite positions)
- **scikit-learn** (Computes validation metrics: MSE, MAE, R², and reconstruction accuracy)

## Examples

```
metabolome-transformer train-binary --input train.csv --output-dir results/current_task --id-column sample_id --label-column label --task current --epochs 10 --batch-size 12
```

## Evaluation signals

- Validation loss after 200 epochs converges to ≈ 0.0684
- Reconstruction accuracy on masked metabolite positions reaches ≈ 0.9533
- MSE ≈ 0.0686, MAE ≈ 0.1291, R² ≈ 0.9305 on the validation split
- Serialized checkpoint metabolm_pretrained.pt loads cleanly and pairs with the metrics dictionary in pretraining_metrics
- Masking step masks exactly 10% of the 168 metabolite feature positions per sample

## Limitations

- Pre-training cohort is predominantly of European descent, healthier, and of higher socioeconomic status, which may limit generalizability of the learned metabolite representations
- NMR-based metabolomics may miss metabolites detectable by other analytical platforms, bounding what the encoder can represent
- Reconstruction metrics are specific to the 168-biomarker NMR panel; they will not transfer unchanged to other metabolomic feature sets

## Evidence

- [readme] Repo provides imputation and binary fine-tuning CLI that consumes the pre-trained model assets: [evidence span withheld — anchor status: external_doc]
