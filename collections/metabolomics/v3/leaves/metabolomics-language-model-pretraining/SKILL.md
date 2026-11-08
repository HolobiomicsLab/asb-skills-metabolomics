---
name: metabolomics-language-model-pretraining
description: Use when you have large-scale population NMR metabolomics profiles (e.g.,
  UK Biobank plasma NMR panels with ~249 metabolite features) and need a pre-trained
  foundation model for downstream multi-disease prediction or risk stratification.
when_to_use_negative:
- Input is not NMR-platform plasma metabolomics (e.g., mass-spectrometry metabolomics)
  — the article notes NMR-based metabolomics may miss metabolites detectable by other
  analytical methods
- Input features are not the 249 metabolite columns expected by the pretrained asset
  (e.g., a generic feature table from a different cohort/platform)
- The goal is single-disease clinical risk scoring with validated equations (e.g.,
  ASCVD) rather than metabolomics-based multi-disease prediction
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: MetaboLM
  role: Transformer-based metabolomic language model whose pre-training over NMR metabolomics
    data this skill executes; ablation of the metabolite correlation matrix in its
    attention scores is done in this codebase
  repo: github.com/Qiu-Shizheng/MetaboLM
- name: Metabolome-Transformer
  role: Command-line tool providing imputation, fine-tuning (train-binary for current/future
    disease prediction), and prediction (predict-binary) around the pre-trained checkpoint
  repo: https://github.com/Qiu-Shizheng/Metabolome-Transformer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:fdcfc330f2aea103db73e5f0fc6a0663effb100a0c4df4f502a5ad59755ab792
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - MetaboLM
  - MetaboLM (github.com/Qiu-Shizheng/MetaboLM)
  - Metabolome-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-language-model-pretraining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-train a transformer-based metabolomic language model (MetaboLM) on NMR plasma metabolomics data from 83,744 relatively healthy UK Biobank participants to capture normative metabolite expression and interaction patterns, then fine-tune the checkpoint on chronic-disease prediction tasks.

## When to use

Use when you have large-scale population NMR metabolomics profiles (e.g., UK Biobank plasma NMR panels with ~249 metabolite features) and need a pre-trained foundation model for downstream multi-disease prediction or risk stratification.

## When NOT to use

- Input is not NMR-platform plasma metabolomics (e.g., mass-spectrometry metabolomics) — the article notes NMR-based metabolomics may miss metabolites detectable by other analytical methods
- Input features are not the 249 metabolite columns expected by the pretrained asset (e.g., a generic feature table from a different cohort/platform)
- The goal is single-disease clinical risk scoring with validated equations (e.g., ASCVD) rather than metabolomics-based multi-disease prediction

## Inputs

- CSV table with a sample ID column plus the 249 NMR metabolite columns listed in assets/pretrained/metabolite_columns.txt (UK Biobank plasma NMR metabolomics)
- Preprocessed NMR plasma metabolomics data from 83,744 relatively healthy UK Biobank participants for pre-training
- Labeled training table (sample_id, binary label, metabolite columns) for fine-tuning on chronic disease outcomes

## Outputs

- Pre-trained MetaboLM transformer checkpoint capturing normative metabolite expression patterns
- Imputed metabolomics CSV (missing values filled per sample row)
- Fine-tuned binary classifier (classifier.pt) per disease task
- Prediction CSV with sample_id, probability, and binary prediction columns

## How to apply

Load the preprocessed NMR plasma metabolomics table for the 83,744 relatively healthy UK Biobank participants, ensuring each sample row carries the 249 metabolite columns listed in the repo's assets/pretrained/metabolite_columns.txt. Impute missing values with the metabolome-transformer 'impute' subcommand before training. Pre-train the transformer attention mechanism over metabolite tokens, optionally incorporating the metabolite-metabolite correlation matrix into the attention scores (an ablation removing it significantly reduced downstream disease-prediction performance, t = 3.14, P = 0.00395, with 6/9 hyperparameter configurations declining). The purpose of pre-training is to capture the normative metabolite expression patterns of the general population. Fine-tune the resulting checkpoint on binary disease labels via 'train-binary' (current or future disease prediction tasks), then score held-out samples with 'predict-binary'.

## Related tools

- **MetaboLM** (Transformer-based metabolomic language model whose pre-training over NMR metabolomics data this skill executes; ablation of the metabolite correlation matrix in its attention scores is done in this codebase) — github.com/Qiu-Shizheng/MetaboLM
- **Metabolome-Transformer** (Command-line tool providing imputation, fine-tuning (train-binary for current/future disease prediction), and prediction (predict-binary) around the pre-trained checkpoint) — https://github.com/Qiu-Shizheng/Metabolome-Transformer

## Examples

```
metabolome-transformer train-binary --input train.csv --output-dir results/future_task --id-column sample_id --label-column label --task future --epochs 10 --device cpu
```

## Evaluation signals

- Input table must contain exactly the 249 metabolite columns listed in assets/pretrained/metabolite_columns.txt plus the sample ID column; a schema mismatch means wrong input format
- Ablation check: excluding the metabolite correlation matrix from attention while keeping all other architecture/hyperparameters unchanged should reproduce a significant downstream performance drop (paired t-test t = 3.14, P = 0.00395; 6/9 configurations decline)
- Prediction output must contain the ID column plus 'probability' and 'prediction' columns, with probabilities in [0,1] and predictions in {0,1}
- Fine-tuning on the 16 common chronic diseases should improve prediction over a non-pre-trained baseline, consistent with the reported excellent performance across the 16 diseases
- Missing-value imputation should remove empty metabolite entries in the imputed CSV (no blank cells in metabolite columns)

## Limitations

- Cohort generalizability: the UK Biobank study population is predominantly of European descent, more racially homogeneous, typically healthier, and of higher socioeconomic status, which may limit generalizability
- NMR-based metabolomics may overlook certain metabolites detectable by other analytical methods
- The fine-tuning CLI exposed in the README covers binary current/future disease prediction tasks; the original pre-training run used 9 hyperparameter configurations, so replication requires sweeping all 9 to compare fairly

## Evidence

- [readme] Expected input format for the CLI: [evidence span withheld — anchor status: external_doc]
- [readme] Fine-tuning input schema for disease prediction: [evidence span withheld — anchor status: external_doc]
- [results] Ablation finding on the metabolite correlation matrix: [evidence span withheld — anchor status: not_found]
