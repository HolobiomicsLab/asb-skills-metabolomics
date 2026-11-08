---
name: metabolite-importance-ranking
description: Use when you have fine-tuned MetaboLM disease model checkpoints over
  249 NMR metabolite columns and need to identify which metabolites drive predictions,
  especially multi-disease biomarker candidates across the 16 diseases.
when_to_use_negative:
- Your data are not NMR-based plasma metabolomics aligned to the 249 metabolite columns
  — the article notes NMR-based metabolomics may overlook certain metabolites detectable
  by other analytical methods
- You need disease risk stratification scores rather than feature importance — derive
  MetaboRS from prediction probabilities instead
- You only have a single generic (non-fine-tuned) checkpoint; interpretability of
  pre-trained normative patterns differs from the disease-specific analysis
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Load the 16 fine-tuned model checkpoints, run inference, and register hooks
    on the last transformer layer to capture attention weight matrices
  repo: github.com/Qiu-Shizheng/MetaboLM
- name: pandas
  role: Aggregate and save the per-disease metabolite importance rankings as metabolite_importance_rankings.csv
- name: Metabolome-Transformer
  role: Latest MetaboLM implementation providing the fine-tuned classifiers (classifier.pt)
    whose attention layers are analyzed
  repo: Qiu-Shizheng/Metabolome-Transformer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:aaef6fb7f7f015bead660e0d807ca3c2208c500a35a81e15fab200056b5ebe43
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - Metabolome-Transformer
  - PyTorch
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolite-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks NMR metabolite features by their importance for disease prediction using attention-weight interpretability of fine-tuned MetaboLM transformer models, to nominate metabolites that recur as key predictors across the 16 chronic diseases.

## When to use

Use when you have fine-tuned MetaboLM disease model checkpoints over 249 NMR metabolite columns and need to identify which metabolites drive predictions, especially multi-disease biomarker candidates across the 16 diseases.

## When NOT to use

- Your data are not NMR-based plasma metabolomics aligned to the 249 metabolite columns — the article notes NMR-based metabolomics may overlook certain metabolites detectable by other analytical methods
- You need disease risk stratification scores rather than feature importance — derive MetaboRS from prediction probabilities instead
- You only have a single generic (non-fine-tuned) checkpoint; interpretability of pre-trained normative patterns differs from the disease-specific analysis

## Inputs

- 16 fine-tuned MetaboLM disease model checkpoints (PyTorch .pt classifiers, e.g. results/current_task/classifier.pt)
- Metabolomics input tables with one sample_id column and the 249 metabolite columns listed in assets/pretrained/metabolite_columns.txt
- Metabolite column manifest (assets/pretrained/metabolite_columns.txt)

## Outputs

- metabolite_importance_rankings.csv with per-disease metabolite importance scores and top-20 rankings
- Top metabolite-metabolite attention interaction scores per disease (e.g., dementia model)

## How to apply

Load each of the 16 fine-tuned disease model checkpoints from the MetaboLM repository with PyTorch and run inference on the metabolomics input samples while registering hooks on the last transformer layer to capture attention weight matrices. Aggregate attention mass per metabolite feature (column- and row-wise) to compute an importance score for each of the 249 NMR metabolite columns. Rank metabolites by aggregated importance within each disease model and record the top-20 per disease; optionally extract pairwise attention interaction scores (e.g., for the dementia model). Compare rankings across the 16 diseases to the reported key-metabolite findings (acetate and leucine in 15/16 diseases, phenylalanine in 13/16) and write the per-disease table to metabolite_importance_rankings.csv with pandas. Note that the MetaboLM README points to Qiu-Shizheng/Metabolome-Transformer as the latest version, so model-loading code may live in that repo.

## Related tools

- **PyTorch** (Load the 16 fine-tuned model checkpoints, run inference, and register hooks on the last transformer layer to capture attention weight matrices) — github.com/Qiu-Shizheng/MetaboLM
- **pandas** (Aggregate and save the per-disease metabolite importance rankings as metabolite_importance_rankings.csv)
- **Metabolome-Transformer** (Latest MetaboLM implementation providing the fine-tuned classifiers (classifier.pt) whose attention layers are analyzed) — Qiu-Shizheng/Metabolome-Transformer

## Examples

```
metabolome-transformer predict-binary --input test.csv --classifier results/current_task/classifier.pt --output results/predictions.csv --id-column sample_id
```

## Evaluation signals

- The ranking reproduces the paper's headline result: acetate and leucine appear as key predictors in ~15 of 16 diseases and phenylalanine in ~13 of 16
- metabolite_importance_rankings.csv contains one importance score per metabolite for each of the 16 disease models (no missing columns relative to the 249-feature manifest)
- Captured attention matrices from the last transformer layer have shape [num_heads, num_features, num_features] matching the 249 metabolite columns
- Top-20 metabolite lists overlap substantially across the 16 diseases, consistent with multi-disease biomarker candidates
- Inference is run under torch.no_grad()/eval mode so attention weights reflect model parameters, not training updates

## Limitations

- Cohort is predominantly European descent, healthier, and of higher socioeconomic status, limiting generalizability of key-metabolite findings
- NMR-based metabolomics may miss metabolites detectable by other analytical platforms, so importance rankings are bounded by the 249 measured features
- MetaboLM README redirects to Qiu-Shizheng/Metabolome-Transformer as the latest version, so checkpoint interfaces may have changed; input tables must contain exactly the metabolite columns in assets/pretrained/metabolite_columns.txt

## Evidence

- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [readme] readme-predict: [evidence span withheld — anchor status: not_found]
