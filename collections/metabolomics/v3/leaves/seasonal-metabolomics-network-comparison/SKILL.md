---
name: seasonal-metabolomics-network-comparison
description: Use when raw seasonal metabolite abundance profiles (replicates x metabolites
  per harvesting season) must be converted into correlation-based networks whose edges
  represent significant metabolite-metabolite associations.
when_to_use_negative:
- The input is already a correlation matrix or a pre-built network — this skill operates
  on raw abundance profiles and would re-derive correlations redundantly.
- The downstream goal is predicting metabolic pathways from an existing Metabolite
  CN — that is handled by the NetAurHPD hyperlink-prediction pipeline (network auralization
  + M5 classifier), not by CN construction.
- The abundance table has too few biological replicates per season for meaningful
  pairwise Pearson correlation estimates.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pcaMethods
  role: Provides the probabilistic PCA method used to impute missing values in the
    metabolite profiles before correlation computation
- name: NetAurHPD
  role: Downstream framework that consumes the constructed Metabolite correlation-based
    networks of tomato pericarp for hyperlink prediction
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:bb3427679d58479d4af867fa5df9f12f965965972040f3a8cb3af1b6a1ed9ac0
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - NetAurHPD
  - Rstudio
  - pcaMethods
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# seasonal-metabolomics-network-comparison

## Summary

Constructs per-season Metabolite correlation-based networks (Metabolite CNs) from tomato pericarp metabolomics profiles (harvesting seasons 2001, 2003, 2004) by normalizing sample- and metabolite-level data, imputing missing values with probabilistic PCA (pcaMethods in Rstudio), and thresholding pairwise Pearson correlations into significant links.

## When to use

Use when raw seasonal metabolite abundance profiles (replicates x metabolites per harvesting season) must be converted into correlation-based networks whose edges represent significant metabolite-metabolite associations.

## When NOT to use

- The input is already a correlation matrix or a pre-built network — this skill operates on raw abundance profiles and would re-derive correlations redundantly.
- The downstream goal is predicting metabolic pathways from an existing Metabolite CN — that is handled by the NetAurHPD hyperlink-prediction pipeline (network auralization + M5 classifier), not by CN construction.
- The abundance table has too few biological replicates per season for meaningful pairwise Pearson correlation estimates.

## Inputs

- Seasonal metabolite abundance profile matrices (biological replicates x metabolites) for tomato pericarp harvesting seasons 2001, 2003, and 2004
- Metabolite identifiers for node labeling

## Outputs

- Per-season nodes-and-edges correlation network table (e.g. metabolite_corr_network.csv)
- Node counts and edge counts per season for validation (expected: 2001 = 75 nodes/473 links; 2003 = 75 nodes/869 edges; 2004 = 78 nodes/338 edges)

## How to apply

Load the metabolic profiles for each harvesting season (2001, 2003, 2004) and normalize the data at both the sample and metabolite levels. Impute missing values using a probabilistic PCA method from the pcaMethods package in Rstudio. For each year independently, compute the Pearson Correlation Coefficient between every pair of metabolites across biological replicates. Threshold each correlation matrix at |r| >= 0.3 with p <= 0.01 so that only significant correlations become edges; this threshold choice was the one reported in the source article. Export each year's Metabolite CN as a nodes-and-edges table (e.g. metabolite_corr_network.csv) recording node counts and edge counts per season. Validate the resulting network sizes against the article's reference values: 2001 with 75 metabolites/473 links, 2003 with 75 nodes/869 edges, and 2004 with 78 nodes/338 edges.

## Related tools

- **pcaMethods** (Provides the probabilistic PCA method used to impute missing values in the metabolite profiles before correlation computation)
- **NetAurHPD** (Downstream framework that consumes the constructed Metabolite correlation-based networks of tomato pericarp for hyperlink prediction) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
library(pcaMethods); imputed <- pca(metab_profile, method = "ppca")
```

## Evaluation signals

- Each season's correlation matrix retains links only where |r| >= 0.3 and p <= 0.01 — inspect the edges table to confirm no weaker/insignificant edges survive.
- Network sizes match the reported reference values: 2001 with 75 metabolites/473 links, 2003 with 75 nodes/869 edges, 2004 with 78 nodes/338 edges.
- Edges table is a valid nodes-and-edges format recording node counts and edge counts per season.
- Missing values were imputed before correlation computation (probabilistic PCA), and normalization was applied at both the sample and metabolite levels.

## Limitations

- Results depend on the chosen correlation cutoff (|r| >= 0.3, p <= 0.01); different thresholds will change network size and downstream NetAurHPD performance.
- Imputation via probabilistic PCA introduces estimated values, so correlations involving heavily imputed metabolites may be less reliable.
- Networks must be built and compared per harvesting season (2001, 2003, 2004); pooling seasons would mix non-comparable profiles.

## Evidence

- [methods] we calculate the Pearson Correlation Coefficient between every pair of metabolites (Supplementary Fig 8.f): [evidence span withheld — anchor status: not_found]
- [methods] the selected correlation threshold was set to be |r|>= 0.3 and p<=0.01: [evidence span withheld — anchor status: not_found]
