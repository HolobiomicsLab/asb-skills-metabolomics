---
name: metabolite-pairwise-correlation-computation
description: Use when raw metabolite abundance profiles (e.g., tomato pericarp metabolic
  profiles across biological replicates) must be converted into a nodes-and-edges
  correlation network for downstream network analysis or pathway prediction.
when_to_use_negative:
- When the input is already a thresholded nodes-and-edges network rather than raw
  metabolite abundance profiles — reconstruction is unnecessary.
- When the analysis goal is pathway prediction itself (use NetAurHPD on an existing
  Metabolite CN) rather than building the correlation network from raw data.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Rstudio
  role: Environment in which normalization, imputation, and pairwise Pearson correlation
    are performed
- name: pcaMethods
  role: R package providing probabilistic PCA for missing value imputation prior to
    correlation computation
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:a5ad41f27cf1a168406241fb73215846538faede50ca5577ced130871e9ab6a1
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NetAurHPD
  - Rstudio
  - pcaMethods
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-pairwise-correlation-computation

## Summary

Constructs metabolite correlation-based networks (Metabolite CNs) by computing pairwise Pearson Correlation Coefficients across metabolite abundance profiles and thresholding the matrices at |r| >= 0.3 and p <= 0.01 to retain significant links.

## When to use

Use when raw metabolite abundance profiles (e.g., tomato pericarp metabolic profiles across biological replicates) must be converted into a nodes-and-edges correlation network for downstream network analysis or pathway prediction.

## When NOT to use

- When the input is already a thresholded nodes-and-edges network rather than raw metabolite abundance profiles — reconstruction is unnecessary.
- When the analysis goal is pathway prediction itself (use NetAurHPD on an existing Metabolite CN) rather than building the correlation network from raw data.

## Inputs

- tomato pericarp metabolomics dataset (2001, 2003, 2004) — metabolite abundance profiles with biological replicates
- sample-level and metabolite-level normalization parameters

## Outputs

- nodes-and-edges correlation network table per season (e.g., metabolite_corr_network.csv)
- per-year node counts and edge counts (2001: 75 nodes/473 links; 2003: 75 nodes/869 edges; 2004: 78 nodes/338 edges)

## How to apply

Load the metabolic profiles (tomato pericarp harvesting seasons 2001, 2003, and 2004) and normalize the data at both the sample and metabolite levels. Impute missing values with probabilistic PCA from the pcaMethods package in Rstudio. For each year, compute the Pearson Correlation Coefficient between every pair of metabolites across biological replicates. Threshold each correlation matrix at |r| >= 0.3 with p <= 0.01, retaining only significant links. Export each year's Metabolite CN as a nodes-and-edges table (e.g., metabolite_corr_network.csv) recording node counts and edge counts per season.

## Related tools

- **Rstudio** (Environment in which normalization, imputation, and pairwise Pearson correlation are performed)
- **pcaMethods** (R package providing probabilistic PCA for missing value imputation prior to correlation computation)

## Examples

```
R: library(pcaMethods); imputed <- pca(t(profiles), method="ppca", nPcs=2); R <- cor(t(imputed@completeObs)); sig <- which(abs(R) >= 0.3 & pmat <= 0.01, arr.ind=TRUE)
```

## Evaluation signals

- Verify resulting network sizes against reported reference values: 2001: 75 metabolites/473 links; 2003: 75 nodes/869 edges; 2004: 78 nodes/338 edges.
- Confirm all retained edges satisfy both threshold conditions: |r| >= 0.3 and p-value <= 0.01.
- Confirm network structure is symmetric (undirected pairwise correlations) with no self-loop edges.
- Check the exported CSV records node counts and edge counts for each harvesting season separately.

## Limitations

- Threshold values (|r| >= 0.3, p <= 0.01) were tuned on tomato pericarp data; other metabolomics datasets may require re-tuning.
- Correlation-based links do not imply direct metabolic reactions — NetAurHPD's low explainability means network links alone cannot confirm pathway membership.

## Evidence

- [methods] Construction of metabolite correlation network (Metabolite CN) using Pearson Correlation Coefficient: [evidence span withheld — anchor status: not_found]
- [methods] we calculate the Pearson Correlation Coefficient between every pair of metabolites: [evidence span withheld — anchor status: not_found]
- [methods] threshold |r| >= 0.3 and p <= 0.01: [evidence span withheld — anchor status: not_found]
