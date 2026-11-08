---
name: masked-metabolite-transformer-pretraining
description: Use when a large-scale, mostly-healthy metabolomic feature matrix (e.g.,
  168 NMR metabolite biomarkers per participant) is available and a self-supervised
  foundation model of normative plasma metabolome patterns is needed prior to disease
  prediction fine-tuning.
when_to_use_negative:
- The goal is already fine-tuning a pre-trained checkpoint on a labeled disease cohort
  (binary label column) rather than self-supervised pre-training — the Metabolome-Transformer
  CLI's train-binary path is the right tool then
- The metabolite panel or measurement platform differs substantially (e.g., metabolites
  detectable by other analytical methods that NMR misses), so the pre-trained representation
  of the 168 NMR biomarkers is not applicable
- Only a small labeled dataset is available without the large healthy-cohort metabolomics
  matrix needed to learn normative metabolite patterns
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implements the transformer encoder, AdamW optimizer (lr 1e-4), masked-metabolite
    reconstruction loss, and 200-epoch training loop
- name: pandas
  role: Loads the 168-column NMR metabolomics feature matrix for the 83,744 UK Biobank
    participants
- name: NumPy
  role: Array operations for masking 10% of metabolite feature positions per sample
- name: scikit-learn
  role: Train/validation partitioning and computation of MSE, MAE, and R2 validation
    metrics
- name: MetaboLM / Metabolome-Transformer
  role: Reference implementation and release repository for the pre-trained metabolomic
    language model
  repo: https://github.com/Qiu-Shizheng/Metabolome-Transformer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:48a471637a2240b631b2e167417cdd1afe932dce262ddefee9534ac7ef9c3acc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  repo_url: https://github.com/Qiu-Shizheng/Metabolome-Transformer
  tools:
  - MetaboLM / Metabolome-Transformer
  - NumPy
  - PyTorch
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# masked-metabolite-transformer-pretraining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-train a BERT-style transformer language model (MetaboLM) on plasma NMR metabolomics from 83,744 relatively healthy UK Biobank participants using a masked-metabolite reconstruction objective with Hadamard-product embeddings and a [CLS] token, capturing normative metabolite expression patterns before downstream fine-tuning.

## When to use

Use when a large-scale, mostly-healthy metabolomic feature matrix (e.g., 168 NMR metabolite biomarkers per participant) is available and a self-supervised foundation model of normative plasma metabolome patterns is needed prior to disease prediction fine-tuning.

## When NOT to use

- The goal is already fine-tuning a pre-trained checkpoint on a labeled disease cohort (binary label column) rather than self-supervised pre-training — the Metabolome-Transformer CLI's train-binary path is the right tool then
- The metabolite panel or measurement platform differs substantially (e.g., metabolites detectable by other analytical methods that NMR misses), so the pre-trained representation of the 168 NMR biomarkers is not applicable
- Only a small labeled dataset is available without the large healthy-cohort metabolomics matrix needed to learn normative metabolite patterns

## Inputs

- NMR plasma metabolomics feature matrix (168 metabolite biomarker columns) for 83,744 relatively healthy UK Biobank participants
- Train/validation partition of participants with numeric metabolite concentration values per sample

## Outputs

- Pre-trained MetaboLM transformer weights (e.g., metabolm_pretrained.pt)
- Validation-set metrics dictionary: loss 0.0684, reconstruction accuracy 0.9533, MSE 0.0686, MAE 0.1291, R2 0.9305

## How to apply

Load the NMR metabolomics feature matrix (168 metabolite biomarkers per participant, 83,744 relatively healthy UK Biobank individuals) with pandas/NumPy and split into training and validation sets. Build each sample's input by applying the Hadamard-product feature embedding to the 168-dim metabolite vector and prepending a [CLS] token, then randomly mask 10% of metabolite feature positions to create the reconstruction objective. Train the transformer encoder with AdamW at learning rate 1e-4 for 200 epochs in PyTorch, minimizing reconstruction loss only on masked positions. Evaluate on the held-out validation split and serialize the weights (e.g., metabolm_pretrained.pt) plus a metrics dictionary for downstream fine-tuning on the 16 chronic-disease tasks.

## Related tools

- **PyTorch** (Implements the transformer encoder, AdamW optimizer (lr 1e-4), masked-metabolite reconstruction loss, and 200-epoch training loop)
- **pandas** (Loads the 168-column NMR metabolomics feature matrix for the 83,744 UK Biobank participants)
- **NumPy** (Array operations for masking 10% of metabolite feature positions per sample)
- **scikit-learn** (Train/validation partitioning and computation of MSE, MAE, and R2 validation metrics)
- **MetaboLM / Metabolome-Transformer** (Reference implementation and release repository for the pre-trained metabolomic language model) — https://github.com/Qiu-Shizheng/Metabolome-Transformer

## Examples

```
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-4)  # mask 10% of the 168 metabolite positions; train 200 epochs; save metabolm_pretrained.pt
```

## Evaluation signals

- Validation loss converges to approximately 0.0684 after 200 epochs
- Reconstruction accuracy on masked positions reaches approximately 0.9533
- Validation MSE ~0.0686, MAE ~0.1291, and R2 ~0.9305 on the held-out split
- Exactly ~10% of metabolite feature positions are masked per sample during each training step
- Serialized checkpoint (metabolm_pretrained.pt) loads and reproduces validation metrics with a fixed seed

## Limitations

- Pre-training cohort is predominantly of European descent, more racially homogeneous, healthier, and of higher socioeconomic status, which may limit generalizability
- NMR-based metabolomics may overlook metabolites detectable by other analytical methods
- Downstream release CLI (Metabolome-Transformer) documents imputation and binary fine-tuning with 249 metabolite columns; the original 168-biomarker pre-training configuration must be reconstructed from the paper workflow

## Evidence

- [abstract] Pre-training a transformer-based language model on plasma metabolomics data from 83,744 relatively healthy UK Biobank participants: "a transformer-based language model pre-trained on plasma metabolomics data from 83,744 relatively healthy UK Biobank participants"
- [intro] Purpose of pre-training phase: "The purpose of the pre-training phase is to enable the model to capture the normative metabolite expression patterns of the general population."
- [intro] Pre-training dataset described in intro: "pre-trained with nuclear magnetic resonance (NMR) metabolomics data of 83,744 relatively healthy individuals from the UK biobank"
- [readme] README input format for metabolite table: [evidence span withheld — anchor status: external_doc]
- [discussion] Generalizability limitation of the cohort: "this cohort is more racially homogeneous, typically healthier, and has a higher socioeconomic status, which may limit the generalizability of our findings"
