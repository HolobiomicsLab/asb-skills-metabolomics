---
name: ssgsea-module-score-analysis
description: Use when a matched scRNA-seq/metabolomics dataset has ssGSEA module scores
  and metabolite abundances for knockdown vs control conditions, and the goal is to
  test whether substrate-level metabolic module activity tracks metabolite abundance
  changes.
when_to_use_negative:
- Input is raw scRNA-seq expression without precomputed ssGSEA module scores or module
  clusters — run ssGSEA/scFEA module scoring first
- The question is flux prediction itself rather than validating predicted variation
  against matched metabolomics — use the scFEA/FLUXestimator flux estimation pipeline
  instead
- Metabolomics data lack matched knockdown/control conditions, making fold-change
  computation and the correlation undefined
edam_operation: null
tools:
- name: scFEA
  role: Provides the underlying module framework and the validated Pa03c matched scRNA-seq/metabolomics
    analysis whose Figure S2 and Table S3 this correlation analysis reproduces
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web-server implementation of the scFEA pipeline used to generate the predicted
    flux and module metabolite-usage structure that the ssGSEA correlation validates
  repo: https://github.com/changwn/scFEA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:04ee84112b0540584e28acc337428dd8e7b37a2a4c6ab65b3b998ffbb6c7d777
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/changwn/scFEA
  repo_url: https://github.com/changwn/scFEA
  tools:
  - FLUXestimator
  - scFEA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ssgsea-module-score-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the Figure S2 analysis from the scFEA/Pa03c APEX1-knockdown study: correlating observed metabolomic fold changes of eight metabolites against knockdown-minus-control differences of averaged ssGSEA module scores for modules consuming those metabolites, linking predicted flux behavior to measured metabolite variation.

## When to use

Use when a matched scRNA-seq/metabolomics dataset has ssGSEA module scores and metabolite abundances for knockdown vs control conditions, and the goal is to test whether substrate-level metabolic module activity tracks metabolite abundance changes.

## When NOT to use

- Input is raw scRNA-seq expression without precomputed ssGSEA module scores or module clusters — run ssGSEA/scFEA module scoring first
- The question is flux prediction itself rather than validating predicted variation against matched metabolomics — use the scFEA/FLUXestimator flux estimation pipeline instead
- Metabolomics data lack matched knockdown/control conditions, making fold-change computation and the correlation undefined

## Inputs

- Supplementary Table S3: ssGSEA module scores, metabolomics measurements, and module clusters for the Pa03c APEX1-knockdown vs control comparison (CSV/table)
- List of the eight reported metabolites and their module substrate assignments
- Condition labels distinguishing APEX1-KD (siRef-1) cells from mock/SCR controls

## Outputs

- Pearson correlation statistic (estimate) between eight-metabolite fold changes and averaged module ssGSEA score differences
- Scatter plot reproducing Figure S2 (figure_s2_correlation.png) with annotated correlation
- Written correlation statistics summary

## How to apply

Load supplementary Table S3 (ssGSEA results, metabolomics data and clusters of metabolic modules derived in the Pa03c cell line data) into a pandas or R data frame. Compute observed fold changes (knockdown vs control) for the eight reported metabolites, then average ssGSEA module scores within each module cluster and subtract control from APEX1-KD to get per-cluster score differences. Compute the Pearson correlation between the eight-metabolite fold-change vector and the averaged module ssGSEA score-difference vector. Plot metabolite fold changes (y-axis) against module score differences (x-axis) as a scatter plot reproducing Figure S2, annotating the correlation estimate, and export statistics and the figure (figure_s2_correlation.png). The rationale is that scFEA's validation relies on consistency between predicted flux/imbalance and observed metabolite variation in matched metabolomics data.

## Related tools

- **scFEA** (Provides the underlying module framework and the validated Pa03c matched scRNA-seq/metabolomics analysis whose Figure S2 and Table S3 this correlation analysis reproduces) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web-server implementation of the scFEA pipeline used to generate the predicted flux and module metabolite-usage structure that the ssGSEA correlation validates) — https://github.com/changwn/scFEA

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(metabolite_fold_changes, module_ssGSEA_score_differences)
```

## Evaluation signals

- Fold-change vector has exactly eight values (one per reported metabolite) with correct knockdown/control orientation
- Per-cluster score differences are knockdown-minus-control averages of ssGSEA scores within each module cluster from Table S3
- Pearson correlation can be recomputed from the two eight-element vectors and matches the annotated statistic
- Scatter plot axes match Figure S2: x = difference of averaged ssGSEA score (APEX1-KD vs control), y = fold change of observed metabolomic profile
- Result is consistent with the scFEA finding that predicted flux/metabolic imbalance matches observed metabolite variation in matched metabolomics data

## Limitations

- The correlation is limited to the eight reported metabolites whose modules use them as substrates; broader metabolite coverage is not supported by Table S3
- Pearson correlation assumes approximately linear association between fold change and score difference; it does not establish causality of APEX1 knockdown effects
- scFEA/FLUXestimator does not yet support direct flux estimation from metabolomics data, so validation is indirect via metabolite variation rather than measured fluxes

## Evidence
