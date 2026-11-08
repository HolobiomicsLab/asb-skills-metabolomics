---
name: cross-disease-metabolite-comparison
description: Use when you have multiple fine-tuned transformer checkpoints trained
  on NMR metabolomics (e.g., 249 UK Biobank metabolite features per disease) and need
  to rank metabolite features by attention importance and compare rankings across
  diseases to find recurring biomarkers.
when_to_use_negative:
- When the goal is disease prediction or risk stratification itself (use the predict-binary
  / MetaboRS workflow instead) rather than explaining which metabolites drove predictions
- When input data comes from a non-NMR platform — the article notes NMR metabolomics
  'may overlook certain metabolites detectable by other analytical methods'
- When only a single fine-tuned model exists — the skill's core value is cross-disease
  comparison of rankings across all 16 disease models
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Load the 16 fine-tuned MetaboLM checkpoints, run inference, and register hooks
    on the last transformer layer to capture attention weight matrices
  repo: https://github.com/Qiu-Shizheng/MetaboLM
- name: pandas
  role: Aggregate attention importance scores per metabolite, rank features, compare
    across the 16 diseases, and write metabolite_importance_rankings.csv
- name: Metabolome-Transformer (metabolome-transformer CLI)
  role: Provides the fine-tuned binary disease classifiers and metabolite column mapping
    whose attention layers are probed for interpretability
  repo: https://github.com/Qiu-Shizheng/Metabolome-Transformer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:8b7f677c13b4d790809143b130e41b8bf3f3ca35bd20917d2e35c9fb54703e1e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  repo_url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - Metabolome-Transformer (metabolome-transformer CLI)
  - PyTorch
  - pandas
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-disease-metabolite-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract metabolite importance rankings from attention weights of fine-tuned MetaboLM transformer classifiers across 16 chronic diseases and compare the per-disease rankings to identify recurring multi-disease biomarker metabolites (e.g., acetate, leucine, phenylalanine).

## When to use

Use when you have multiple fine-tuned transformer checkpoints trained on NMR metabolomics (e.g., 249 UK Biobank metabolite features per disease) and need to rank metabolite features by attention importance and compare rankings across diseases to find recurring biomarkers.

## When NOT to use

- When the goal is disease prediction or risk stratification itself (use the predict-binary / MetaboRS workflow instead) rather than explaining which metabolites drove predictions
- When input data comes from a non-NMR platform — the article notes NMR metabolomics 'may overlook certain metabolites detectable by other analytical methods'
- When only a single fine-tuned model exists — the skill's core value is cross-disease comparison of rankings across all 16 disease models

## Inputs

- Fine-tuned MetaboLM disease classifier checkpoints (e.g., classifier.pt from a train-binary run)
- Metabolomics sample table with one sample ID column and the 249 metabolite columns listed in assets/pretrained/metabolite_columns.txt
- Optional pre-trained checkpoint assets (metabolite_columns.txt) to map feature indices back to metabolite names

## Outputs

- metabolite_importance_rankings.csv — per-disease metabolite importance scores and top-20 rankings
- Per-disease lists of key metabolites (e.g., acetate, leucine in 15/16 diseases; phenylalanine in 13/16)
- Pairwise metabolite-metabolite attention interaction scores per disease

## How to apply

Load each of the 16 fine-tuned disease model checkpoints from the MetaboLM repository with PyTorch and run inference on the metabolomics input samples, registering hooks on the last transformer layer to capture attention weight matrices. Aggregate attention mass per metabolite feature (column- and row-wise) to compute an importance score for each of the NMR metabolite features, then rank metabolites within each disease model and record which fall in the top-20 per disease. Additionally extract pairwise attention interaction scores to identify top metabolite-metabolite interactions (e.g., for the dementia model). Finally, compare rankings across the 16 diseases to identify metabolites key in most models — the analysis reported acetate and leucine in 15 of 16 diseases and phenylalanine in 13 of 16 — and save the table as metabolite_importance_rankings.csv.

## Related tools

- **PyTorch** (Load the 16 fine-tuned MetaboLM checkpoints, run inference, and register hooks on the last transformer layer to capture attention weight matrices) — https://github.com/Qiu-Shizheng/MetaboLM
- **pandas** (Aggregate attention importance scores per metabolite, rank features, compare across the 16 diseases, and write metabolite_importance_rankings.csv)
- **Metabolome-Transformer (metabolome-transformer CLI)** (Provides the fine-tuned binary disease classifiers and metabolite column mapping whose attention layers are probed for interpretability) — https://github.com/Qiu-Shizheng/Metabolome-Transformer

## Examples

```
metabolome-transformer predict-binary --input test.csv --classifier results/current_task/classifier.pt --output results/predictions.csv --id-column sample_id --batch-size 12
```

## Evaluation signals

- Acetate and leucine should appear among key metabolites in 15 of the 16 per-disease rankings, and phenylalanine in 13 of 16
- The importance table covers all 249 metabolite features per disease model and includes a top-20 ranked subset for each of the 16 diseases
- Aggregated attention scores are finite, non-negative, and rankable within each disease model (sanity-check against hook outputs from the last transformer layer)
- Feature indices can be mapped back to metabolite names via assets/pretrained/metabolite_columns.txt without mismatches
- Reproduced rankings should match the reported key-metabolite findings when compared per disease before writing metabolite_importance_rankings.csv

## Limitations

- Pre-training and fine-tuning cohorts derive from UK Biobank, which is 'predominantly of European descent' and healthier/higher socioeconomic status, so key-metabolite findings may not generalize to other populations
- NMR-based metabolomics covers only the 249 NMR features and 'may overlook certain metabolites detectable by other analytical methods'
- The interpretability result is model-derived attention importance, not a direct causal or biochemical measure of metabolite–disease association

## Evidence

- [abstract] Attention-weight interpretability identifies key metabolites: "interpretability analysis of the attention mechanism identified key metabolites related to disease prediction"
- [abstract] Fine-tuning across 16 chronic diseases: "After fine-tuning with metabolomics data from individuals diagnosed with 16 common chronic diseases"
- [readme] 249 metabolite feature columns expected as input: [evidence span withheld — anchor status: external_doc]
- [discussion] NMR platform scope limitation: "it may overlook certain metabolites detectable by other analytical methods"
