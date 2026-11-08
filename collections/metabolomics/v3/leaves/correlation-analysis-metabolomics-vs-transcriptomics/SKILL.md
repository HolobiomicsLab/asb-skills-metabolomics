---
name: correlation-analysis-metabolomics-vs-transcriptomics
description: Use when a matched scRNA-seq/metabolomics dataset with perturbation (e.g.,
  APEX1-KD vs control) is available and the goal is to verify that transcriptome-derived
  module score shifts align with observed metabolite fold changes (Figure S2-style
  reproduction).
when_to_use_negative:
- When the transcriptomics and metabolomics data are not matched to the same perturbation
  experiment (e.g., only unmatched cancer-vs-stroma scRNA-seq data are available)
- When the goal is flux prediction itself rather than cross-omics validation — use
  scFEA/FLUXestimator's flux-estimation workflow instead
- When module-to-metabolite substrate assignments are unavailable, so the eight-metabolite
  / module-cluster pairing cannot be constructed
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scFEA
  role: Standalone tool whose validation this correlation reproduces; scFEA provides
    the metabolic module framework and predicted flux/imbalance that the ssGSEA score
    differences are compared against
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web server implementation of the scFEA pipeline that can generate the flux/metabolic
    imbalance outputs underlying the module-level comparisons
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:b6069f400716e62aaefc77435a3df107ba07891fa26697a8a57282a792933e16
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
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# correlation-analysis-metabolomics-vs-transcriptomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-omics consistency check that correlates observed metabolomic fold changes (e.g., APEX1-knockdown vs control in the Pa03c cell line) with differences in averaged ssGSEA scores of the metabolic modules consuming those metabolites, as in Figure S2 of the scFEA paper. It validates that scFEA-predicted flux/metabolic imbalance varies consistently with measured metabolite abundance.

## When to use

Use when a matched scRNA-seq/metabolomics dataset with perturbation (e.g., APEX1-KD vs control) is available and the goal is to verify that transcriptome-derived module score shifts align with observed metabolite fold changes (Figure S2-style reproduction).

## When NOT to use

- When the transcriptomics and metabolomics data are not matched to the same perturbation experiment (e.g., only unmatched cancer-vs-stroma scRNA-seq data are available)
- When the goal is flux prediction itself rather than cross-omics validation — use scFEA/FLUXestimator's flux-estimation workflow instead
- When module-to-metabolite substrate assignments are unavailable, so the eight-metabolite / module-cluster pairing cannot be constructed

## Inputs

- Supplementary Table S3 (ssGSEA module scores, metabolomics measurements and module clusters for the Pa03c APEX1-knockdown vs control comparison)
- List of eight reported metabolites and the module clusters using them as substrates

## Outputs

- Pearson correlation estimate and statistics for the eight-metabolite fold-change vector vs averaged module ssGSEA score-difference vector
- Scatter plot reproducing Figure S2 (saved as figure_s2_correlation.png)

## How to apply

Load the deposited supplementary Table S3 (ssGSEA results, metabolomics data and module clusters for the Pa03c Pa03c APEX1-knockdown vs control comparison) into a data frame with pandas or R. Compute metabolomic fold changes (knockdown vs control) for the eight reported metabolites. Average ssGSEA module scores within each module cluster and compute the knockdown-minus-control score difference per cluster, yielding one eight-element vector per omics layer. Compute the Pearson correlation between the two eight-element vectors, and plot the averaged ssGSEA score differences on the x-axis against metabolite fold changes on the y-axis, annotating the correlation estimate to reproduce Figure S2. Rationale: scFEA's validation rests on the consistency between predicted flux/metabolic imbalance and observed metabolite variation, so a positive correlation supports the flux-estimation approach.

## Related tools

- **scFEA** (Standalone tool whose validation this correlation reproduces; scFEA provides the metabolic module framework and predicted flux/imbalance that the ssGSEA score differences are compared against) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web server implementation of the scFEA pipeline that can generate the flux/metabolic imbalance outputs underlying the module-level comparisons)

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; df = pd.read_csv('TableS3.csv'); r, p = pearsonr(df['ssGSEA_score_diff'], df['metabolite_fold_change']); print(r, p)
```

## Evaluation signals

- The two correlated vectors each contain exactly eight elements (eight metabolites / eight module clusters)
- Pearson correlation is computed on fold-change (KD vs control) and averaged ssGSEA score difference (KD minus control), not on raw abundances
- Scatter plot axes match Figure S2: x = difference of averaged ssGSEA score (APEX1-KD vs control), y = fold change of observed metabolomic profile
- A positive correlation is expected if scFEA-predicted metabolic imbalance is consistent with observed metabolite variation
- Output figure is saved as figure_s2_correlation.png with correlation statistics reported

## Limitations

- The validation rests on only eight metabolites and their module clusters, so the correlation estimate has limited statistical power
- Flux estimation for direct metabolomics/proteomics inputs is not yet available; the metabolite layer here must come from matched bulk metabolomics, not single-cell metabolomics
- scFEA requires sampling/fitting for datasets larger than 10,000 cells without GPU acceleration, and results depend on imputation settings (Magic imputation recommended for 10x data)

## Evidence

- [readme] Table S3. ssGSEA results, metabolomics data and clusters of metabolic modules derived in the Pa03c cell line data.: "Table S3. ssGSEA results, metabolomics data and clusters of metabolic modules derived in the Pa03c cell line data."
- [readme] Figure S2 axis definition: "The x-axis is the difference of averaged ssGSEA score in the APEX1-KD cells vs control and the y-axis is the fold change of observed metabolomic profile."
- [abstract] Validation rationale: "Application of scFEA on this dataset demonstrated the consistency between predicted flux and metabolic imbalance with the observed variation of metabolites in the matched metabolomics data."
