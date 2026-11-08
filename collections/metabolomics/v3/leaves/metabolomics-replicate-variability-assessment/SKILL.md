---
name: metabolomics-replicate-variability-assessment
description: Use when comparing normalisation strategies for multi-batch, large-scale
  metabolomics matrices (e.g. BioHEART-CT plasma data) by measuring replicate SD per
  metabolite across pooled QC, short replicate, and batch replicate sets.
when_to_use_negative:
- The goal is assessing retention of biological signal or reproducibility of differential
  results rather than low variability among replicates — use a biological-signal or
  reproducibility criterion instead
- Data are not arranged with embedded replicates (pooled QC, short replicates, batch
  replicates); replicate-SD assessment requires the systematic plate embedding design
  the paper describes
- Input is a single small batch with no inter-plate structure, where between-batch
  unwanted variation is not the quantity of interest
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: Provides the hierarchical RUV-III normalisation whose replicate variability
    is being assessed, and the plateDesign function that defines the replicate sets
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Environment in which the replicate SD computation and hRUV vs single-step
    RUV comparison are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:ba9263d40ff6e5e02b7618d8d2037990e75e4141bd620658b3de3caf0241e95f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metabolomics-replicate-variability-assessment

## Summary

Quantifies standard deviation of log-abundance measurements across embedded sample replicates (pooled QC, short within-row replicates, and cross-plate batch replicates) to judge whether a normalisation — e.g. hierarchical hRUV versus single-step RUV — effectively removes unwanted variation in large-scale metabolomics data.

## When to use

Use when comparing normalisation strategies for multi-batch, large-scale metabolomics matrices (e.g. BioHEART-CT plasma data) by measuring replicate SD per metabolite across pooled QC, short replicate, and batch replicate sets.

## When NOT to use

- The goal is assessing retention of biological signal or reproducibility of differential results rather than low variability among replicates — use a biological-signal or reproducibility criterion instead
- Data are not arranged with embedded replicates (pooled QC, short replicates, batch replicates); replicate-SD assessment requires the systematic plate embedding design the paper describes
- Input is a single small batch with no inter-plate structure, where between-batch unwanted variation is not the quantity of interest

## Inputs

- Filtered metabolite abundance matrix (metabolites present in at least 50% of samples) from SydneyBioX/BioHEART_metabolomics BioHEART-CT targeted metabolomics data
- Replicate design information: pooled QC, short replicate, and batch replicate positions (Supplementary Table 1 / plateDesign function output)
- Normalised matrices from each method being compared (hRUV hierarchical and single-step RUV-III)

## Outputs

- Per-metabolite, per-replicate-set standard deviation table for each normalisation method (results_table.csv)
- Aggregated comparison of replicate SDs: hRUV versus single-step RUV

## How to apply

Load the filtered metabolite abundance matrix plus the replicate design (generated with the plateDesign function in the hRUV package, which encodes 3 pooled QC wells at each plate start followed by one every 10 runs, 1 short replicate per row, and 5 batch replicates from the previous plate). Normalise the matrix under each candidate method — here hRUV (progressively merging RUV-III adjustments of neighbouring batches, optionally after intra-batch RUV and signal drift correction with robust smoothers) and single-step RUV using the same replicate structure. For each replicate set, compute the standard deviation of log-abundance measurements across replicate members for every metabolite under each normalised matrix. Aggregate the per-metabolite SDs per method and compare: hierarchical normalisation should achieve consistently lower replicate variability than single-step RUV, consistent with the paper's Fig. 3b where hRUV variants (orange/red) show lower between-replicate SD than single-step methods (blue/green). Write the per-metabolite, per-replicate-set SD table for both methods to results_table.csv.

## Related tools

- **hRUV** (Provides the hierarchical RUV-III normalisation whose replicate variability is being assessed, and the plateDesign function that defines the replicate sets) — https://github.com/SydneyBioX/hRUV
- **R** (Environment in which the replicate SD computation and hRUV vs single-step RUV comparison are executed)

## Examples

```
library(devtools); devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Computed replicate standard deviations for hRUV are consistently lower than those for single-step RUV across all three replicate sets (pooled QC, short replicates, batch replicates)
- SDs are computed on log-abundance values, per metabolite, with replicate membership determined by the plateDesign-defined positions
- Intra-batch RUV applied before hierarchical merging further lowers replicate variation relative to hierarchical-only adjustment
- results_table.csv contains both per-metabolite and per-replicate-set SD entries for every method compared
- Input matrix already passed the paper's filter: metabolites not present in at least 50% of samples removed, and outlying measurements from the three consecutive QC samples at each plate start removed

## Limitations

- By chance, batch-replicate selection can draw samples clustered from only one half of the previous plate (observed between batches 6 and 7, and between batches 13 and 14), limiting the ability to capture unwanted variability across the whole plate and leaving slight residual signal shifts for selected metabolites
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround before variability assessment
- Selection of negative-control metabolites for batch correction is challenging because metabolite signals depend on individual chemical properties; the paper found no significant difference between data-driven negative controls and using all metabolites

## Evidence

- [intro] low variability among replicates as an assessment criterion for normalisation: "we utilise this to assess the normalization on a number of criteria including retention of biological signal, low variability among replication, and reproducibility"
- [readme] hRUV uses intra- and inter-batch replicates with RUV-III (README): "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III"
- [abstract] hierarchical merging of neighbouring-batch adjustments: "a novel workflow to remove unwanted variation in a hierarchical structure (hRUV)"
- [discussion] plateDesign function defines replicate positions: "The replicate design is available as a function plateDesign in the hRUV package"
- [discussion] 50% presence filter applied before assessment: "metabolites that were not present in at least 50% of the samples were filtered out"
