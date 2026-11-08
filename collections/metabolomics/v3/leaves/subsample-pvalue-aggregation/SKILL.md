---
name: subsample-pvalue-aggregation
description: Use when comparing statistical testing methods whose p-values are indistinguishable
  on the full cohort (here, >1,000 complete cases), and differences in power need
  to be exposed by evaluating averaged p-values over random subsamples of controlled
  smaller sizes.
when_to_use_negative:
- When the full-cohort analysis already separates the methods — the article notes
  p-values were indistinguishable only 'for both data sets with over 1,000 observations';
  with <1,000 observations the raw p-values may differ without subsampling.
- When the goal is a single definitive association test on the complete cohort rather
  than a power comparison between competing test variants.
- When samples are too costly or too few to allow 100 repeated draws of sizes up to
  500 without heavy overlap / duplication.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Computing environment in which the score tests and p-value averaging over
    100 random subsets are performed; all simulations and analyses were run in R.
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Supplies pathway graph structures used to construct the normalized Laplacian
    and PaIRKAT-regularized kernels evaluated on each subsample.
- name: KEGGREST
  role: Fetches KEGG pathway memberships (keggLink) that define the 28 pathways tested
    under each method on the subsamples.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:9237b6d486bfe8e697e5674d5124b12ab9a0cc1e1fd50b4922b56b862d39b595
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  repo_url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - KEGGREST
  - R
  - igraph
  - impute (R package)
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# subsample-pvalue-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A power-revealing p-value averaging scheme that draws 100 random subject subsets of sizes 100–500 from each phenotype dataset, runs the comparison methods (PaIRKAT regularized Laplacian, unregularized normalized Laplacian, no-topography) on every subset, and averages p-values per method so that differences between kernel machine variants become visible even when full-cohort p-values are identical.

## When to use

Use when comparing statistical testing methods whose p-values are indistinguishable on the full cohort (here, >1,000 complete cases), and differences in power need to be exposed by evaluating averaged p-values over random subsamples of controlled smaller sizes.

## When NOT to use

- When the full-cohort analysis already separates the methods — the article notes p-values were indistinguishable only 'for both data sets with over 1,000 observations'; with <1,000 observations the raw p-values may differ without subsampling.
- When the goal is a single definitive association test on the complete cohort rather than a power comparison between competing test variants.
- When samples are too costly or too few to allow 100 repeated draws of sizes up to 500 without heavy overlap / duplication.

## Inputs

- Phenotype-split complete-case datasets (e.g. 1,113 log FEV1/FVC cases, 1,065 percent emphysema cases) with preprocessed metabolomics measurements
- Pathway graph / kernel definitions for each of the 28 KEGG pathways under comparison (no topography, normalized Laplacian, PaIRKAT regularized normalized Laplacian with τ = 1)

## Outputs

- Per-pathway, per-method averaged p-values at each subset size (100, 200, 300, 400, 500)
- Power-comparison table/plot (Figures 3, S1, S2 style) showing which method yields smaller averaged p-values per pathway

## How to apply

Take 100 random subsets of subjects at each of five sizes — 100, 200, 300, 400, and 500 — drawn separately from each phenotype dataset (here the 1,113 log FEV1/FVC cases and 1,065 percent emphysema cases). On every subset, compute p-values from the kernel machine score test under the conditions being compared: ignoring pathway topography, using the unregularized normalized Laplacian, and using PaIRKAT's regularized normalized Laplacian with regularization parameter τ = 1. Then average the p-values across the 100 subsets for each method, pathway, and subset size. The rationale, from the article, is that with large n the methods converge and 'P-values from each method were indistinguishable from one another for both data sets with over 1,000 observations', so the averaged small-sample p-values expose which method is more powerful (e.g. PaIRKAT improved power for 18/28 KEGG pathways against log FEV1/FVC). Plot or tabulate the averaged p-values per method to reproduce the comparison corresponding to Figures 3, S1, and S2.

## Related tools

- **R** (Computing environment in which the score tests and p-value averaging over 100 random subsets are performed; all simulations and analyses were run in R.) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Supplies pathway graph structures used to construct the normalized Laplacian and PaIRKAT-regularized kernels evaluated on each subsample.)
- **KEGGREST** (Fetches KEGG pathway memberships (keggLink) that define the 28 pathways tested under each method on the subsamples.)

## Examples

```
set.seed(1); for (m in 1:100) p[m] <- pairkat_test(subsample(113, 500), tau = 1); mean(p)
```

## Evaluation signals

- Confirm 100 subsets were drawn at each of sizes 100, 200, 300, 400, and 500 from each of the two phenotype datasets.
- Verify three methods (no topography, normalized Laplacian, PaIRKAT τ = 1) were scored on every subset so per-method averaged p-values are comparable.
- Check that with >1,000 observations (full cohort) the three methods' p-values are essentially identical, while averaged small-n p-values separate — PaIRKAT should show improved power for 18/28 pathways (FEV1/FVC) and 17/28 (emphysema).
- Reproduced output should be tabulable/plottable as Figures 3, S1, and S2 comparisons.

## Limitations

- Method differences are demonstrated only on downsampled subsets (100–500 subjects), not on the full cohort.
- P-values from each method were indistinguishable for datasets with over 1,000 observations.
- Results depend on the chosen regularization parameter; τ was fixed to 1 with no sensitivity analysis reported.
- Graph structure must be examined before applying regularization; sparse or disconnected metabolomic pathways can greatly hinder performance.

## Evidence
