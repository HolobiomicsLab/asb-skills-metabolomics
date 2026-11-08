---
name: cosine-similarity-confidence-interval-estimation
description: Use when comparing spectral prediction checkpoints on a validation split
  and you need median cosine similarity plus uncertainty (95% CI) to select the configuration
  rather than trusting point estimates.
when_to_use_negative:
- When ranking candidate compounds by spectral similarity for identification rather
  than comparing trained models during selection
- When test-set benchmarking against ICEBERG/CFM-ID is the goal — model selection
  must use the validation split only, never the CASMI 2016/2022 test sets
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3473
tools:
- name: FIORA
  role: Graph neural network whose checkpoints are trained (fiora-train, 200 epochs,
    ADAM, weighted MSE, plateau LR scheduler) and evaluated (fiora-eval) to obtain
    spectral_sqrt_cosine scores for the selection curves
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:884859b78284f2ebb438f0508d618fdd368256b7ddca38295df06daa0be85364
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cosine similarity confidence interval estimation

## Summary

Aggregates per-spectrum spectral_sqrt_cosine scores from FIORA model evaluations into median cosine similarities with 95% confidence intervals (e.g., bootstrapping compounds) so that architecture/depth model-selection decisions are statistically grounded.

## When to use

Use when comparing spectral prediction checkpoints on a validation split and you need median cosine similarity plus uncertainty (95% CI) to select the best configuration rather than trusting point estimates.

## When NOT to use

- When ranking candidate compounds by spectral similarity for identification rather than comparing trained models during selection
- When test-set benchmarking against ICEBERG/CFM-ID is the goal — model selection must use the validation split only, never the CASMI 2016/2022 test sets

## Inputs

- validation_eval.csv per trained checkpoint (fiora-eval output with spectral_sqrt_cosine scores per spectrum)
- Trained FIORA checkpoints (.pt) from fiora-train runs per architecture/depth configuration
- Validation split of the merged NIST'17 + MS-DIAL library

## Outputs

- Median spectral_sqrt_cosine per architecture/depth configuration
- 95% confidence intervals on the median (bootstrap over compounds)
- Architecture/depth selection curves (median cosine similarity vs. graph depth per architecture)

## How to apply

Evaluate each trained FIORA checkpoint on the validation split with fiora-eval, which reports spectral_sqrt_cosine summary scores and writes a validation_eval.csv per run. For each architecture (GCN, RGCN, GAT, Graph Transformer) and graph depth 0-7, compute the median spectral_sqrt_cosine over validation spectra. Compute 95% confidence intervals by bootstrapping compounds (resample compounds, recompute the median, take percentile intervals), since multiple spectra per compound would otherwise overstate the effective sample size. Select the configuration whose median with CI is highest; in the source study this identified the 6-layer RGCN as the best architecture/depth for subsequent benchmarking.

## Related tools

- **FIORA** (Graph neural network whose checkpoints are trained (fiora-train, 200 epochs, ADAM, weighted MSE, plateau LR scheduler) and evaluated (fiora-eval) to obtain spectral_sqrt_cosine scores for the selection curves) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-eval -m checkpoints/fiora.pt --device cuda:0 --output-dir eval/rgcn_d6
```

## Evaluation signals

- Every configuration has a validation_eval.csv with per-spectrum spectral_sqrt_cosine values from fiora-eval
- Median cosine similarity and a 95% CI are reported for each of the four architectures at each depth 0-7
- Bootstrap resampling is done at the compound level, not the spectrum level, so CIs reflect compound-level variability
- The selected best configuration (6-layer RGCN) reproduces the paper's validation selection curve ranking

## Limitations

- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limiting generalization of the selected model
- Exact model specification is only in the Supplementary Information, so depth sweep implementations may diverge from the original
- NIST'17 is commercial and its homogeneity may limit generalization of the selected configuration to more diverse experimental setups

## Evidence

- [abstract] FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality: "FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality"
- [discussion] Train/val/test split of molecular structures: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [readme] Persisting training history for later aggregation: "To persist per-epoch training history, add `--history-out` (supports `.` or `.csv`)"
