---
name: nmr-metabolomics-feature-processing
description: Use when input is a raw NMR metabolomics table with sample IDs and metabolite
  columns containing missing values, and you need an imputed, label-annotated matrix
  for disease-prediction fine-tuning or AUC evaluation.
when_to_use_negative:
- Input is already an imputed, complete feature matrix with no missing metabolite
  values — skip imputation and go straight to train-binary or predict-binary
- Data comes from non-NMR platforms (e.g. mass spectrometry metabolomics) — the 249-column
  schema and pre-trained checkpoints are specific to UK Biobank NMR plasma metabolomics
- The task requires predicting disease more than 10 years ahead via MetaboRS stratification
  rather than short-horizon binary classification — use the risk-score workflow instead
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3474
tools:
- name: Metabolome-Transformer
  role: CLI for NMR metabolomics missing-value imputation and binary disease-prediction
    fine-tuning/prediction
  repo: Qiu-Shizheng/Metabolome-Transformer
- name: MetaboLM
  role: Transformer language model consuming the processed NMR feature tables for
    pre-training and disease-prediction fine-tuning
  repo: github.com/Qiu-Shizheng/MetaboLM
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:64f3f2328fba36ee1064ead0e4f2430f2dd2a1b260056ee8a2e3049a3cd9d75b
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
  - MetaboLM
  - MetaboLM (github.com/Qiu-Shizheng/MetaboLM)
  - Metabolome-Transformer
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nmr-metabolomics-feature-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare UK Biobank NMR plasma metabolomics feature tables (249 metabolite columns per sample) for MetaboLM pre-training and fine-tuning, including missing-value imputation and label attachment, using the Metabolome-Transformer command-line tools.

## When to use

Use when input is a raw NMR metabolomics table with sample IDs and metabolite columns containing missing values, and you need an imputed, label-annotated matrix for disease-prediction fine-tuning or AUC evaluation.

## When NOT to use

- Input is already an imputed, complete feature matrix with no missing metabolite values — skip imputation and go straight to train-binary or predict-binary
- Data comes from non-NMR platforms (e.g. mass spectrometry metabolomics) — the 249-column schema and pre-trained checkpoints are specific to UK Biobank NMR plasma metabolomics
- The task requires predicting disease more than 10 years ahead via MetaboRS stratification rather than short-horizon binary classification — use the risk-score workflow instead

## Inputs

- CSV table with one sample_id column plus the 249 NMR metabolite columns listed in metabolite_columns.txt (missing cells allowed)
- Training CSV with sample_id column, binary label column (e.g. 3-year incident-disease onset), and metabolite columns
- Held-out UK Biobank NMR test sets with 3-year incident-disease labels for AUC evaluation

## Outputs

- Imputed metabolomics CSV with all 249 metabolite columns complete (e.g. results/imputed.csv)
- Fine-tuned binary classifier checkpoint (e.g. results/current_task/classifier.pt)
- Prediction CSV with sample_id, probability, and binary prediction columns (e.g. results/predictions.csv)

## How to apply

Start from a CSV containing one sample ID column and the 249 metabolite columns listed in assets/pretrained/metabolite_columns.txt of the Metabolome-Transformer repository. Run the 'metabolome-transformer impute' command with --id-column and --batch-size (e.g. 12) to fill missing metabolite values, optionally forcing CPU with --device cpu. For fine-tuning, construct a training CSV that additionally carries a binary label column (e.g. 3-year incident-disease onset label or chronic disease diagnosis), then run 'metabolome-transformer train-binary' with --task current (diagnosed disease) or --task future (future onset), --epochs 10, and --batch-size 12. Verify that imputed outputs preserve all sample IDs and all 249 columns with no remaining empty cells, and that downstream MetaboLM checkpoints reproduce reported AUCs such as type 2 diabetes 0.9221 within ±0.01.

## Related tools

- **Metabolome-Transformer** (CLI for NMR metabolomics missing-value imputation and binary disease-prediction fine-tuning/prediction) — Qiu-Shizheng/Metabolome-Transformer
- **MetaboLM** (Transformer language model consuming the processed NMR feature tables for pre-training and disease-prediction fine-tuning) — github.com/Qiu-Shizheng/MetaboLM

## Examples

```
metabolome-transformer impute --input examples/example_input.csv --output results/imputed.csv --id-column sample_id --batch-size 12
```

## Evaluation signals

- Imputed output CSV contains every input sample_id and all 249 metabolite columns with no empty cells remaining
- Fine-tuned classifier predictions produce a valid probability column in [0,1] with a 0/1 prediction column
- Computed 3-year risk AUCs match Supplementary Table 6 references (type 2 diabetes 0.9221, heart failure 0.8167, obesity 0.8106) within ±0.01 via value_in_range check
- train-binary runs complete without device errors when --device cpu is set on CPU-only machines

## Limitations

- Input tables must exactly match the 249 metabolite columns enumerated in assets/pretrained/metabolite_columns.txt; other column sets are unsupported
- NMR-based metabolomics may overlook certain metabolites detectable by other analytical methods
- Downstream model generalizability is limited because the UK Biobank cohort is predominantly of European descent, healthier, and of higher socioeconomic status

## Evidence

- [readme] README: imputation input schema: [evidence span withheld — anchor status: external_doc]
