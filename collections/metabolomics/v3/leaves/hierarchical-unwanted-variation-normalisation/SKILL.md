---
name: hierarchical-unwanted-variation-normalisation
description: Use when a large-scale metabolomics matrix spans multiple 96-well plates/batches
  with embedded replicate structure (pooled QC, short replicates, batch replicates)
  and you must correct signal drift and inter-batch unwanted variation while retaining
  biological signal.
when_to_use_negative:
- When there is only a single batch and no embedded intra- or inter-batch replicates
  — hierarchical merging of neighbouring batches and RUV-III batch correction have
  no replicate structure to estimate unwanted variation from.
- When the input is a missing-value-laden matrix without an imputation step — hRUV
  expects data without missing values, so imputation (e.g. knn/impute) is required
  first.
- When the goal is drift correction only on pooled-QC-identical assumptions for a
  small study — existing single-step normalisation methods may suffice and hierarchical
  RUV is unnecessary.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: hRUV
  role: R package implementing the hierarchical RUV-III normalisation workflow that
    merges adjustments across neighbouring batches using embedded replicate information.
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV (R package, plateDesign function)
  role: Generates/replicates the plate layout that embeds pooled QC, short and batch
    replicates needed to estimate within- and between-batch unwanted variation.
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Language in which the hRUV package and downstream replicate-SD computations
    are run.
  repo: https://github.com/SydneyBioX/hRUV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:5fbf12a356c8b51cd2f67cc169c8f67d2eada044c7d064d2387882d865ff0c13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV (R package, plateDesign function)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hierarchical-unwanted-variation-normalisation

## Summary

Normalises multi-batch large-scale metabolomics data (e.g. BioHEART-CT plasma LC-MS/MS abundances) by progressively merging RUV-III adjustments across neighbouring batches using embedded intra-batch (pooled QC, short) and inter-batch sample replicates, as implemented in the hRUV R package.

## When to use

Use when a large-scale metabolomics matrix spans multiple 96-well plates/batches with embedded replicate structure (pooled QC, short replicates, batch replicates) and you must correct signal drift and inter-batch unwanted variation while retaining biological signal.

## When NOT to use

- When there is only a single batch and no embedded intra- or inter-batch replicates — hierarchical merging of neighbouring batches and RUV-III batch correction have no replicate structure to estimate unwanted variation from.
- When the input is a missing-value-laden matrix without an imputation step — hRUV expects data without missing values, so imputation (e.g. knn/impute) is required first.
- When the goal is drift correction only on pooled-QC-identical assumptions for a small study — existing single-step normalisation methods may suffice and hierarchical RUV is unnecessary.

## Inputs

- Abundance matrix of targeted metabolomics peak areas (from MultiQuant (SCIEX) integration of LC-MS/MS runs on an Agilent 1260 Infinity HPLC System with Qtrap5500) for >1,000 human plasma samples
- Replicate design information assigning pooled QC, short (within-row), and batch (between-plate) replicates to each sample position across batches

## Outputs

- hRUV-normalised metabolite abundance matrix with unwanted variation removed within and between batches
- Per-replicate-set standard deviation table (e.g. results_table.csv) of log-abundance across replicate members for hRUV vs single-step RUV

## How to apply

Filter the abundance matrix so metabolites present in at least 50% of samples are kept, and examine the three consecutive pooled QC samples at the start of each plate to remove outlying measurements. Identify replicate members per set: pooled QC (3 at start, then one every 10 runs), short replicates (one biological sample from the previous row replicated at a random position in each row), and 5 batch replicates drawn from the previous plate and placed after the three pooled QC wells (use the plateDesign function from the hRUV R package). Apply hRUV so it progressively merges adjustments of neighbouring batches via RUV-III with embedded sample replicates, optionally preceded by signal drift correction with robust linear or non-linear smoothers. For comparison, also apply single-step RUV using the same replicate structure. Judge correctness by whether the hRUV-normalised matrix yields lower standard deviation of log-abundance across replicate members than single-step RUV, while retaining biological variation (e.g. hypertension-associated metabolite signals).

## Related tools

- **hRUV** (R package implementing the hierarchical RUV-III normalisation workflow that merges adjustments across neighbouring batches using embedded replicate information.) — https://github.com/SydneyBioX/hRUV
- **hRUV (R package, plateDesign function)** (Generates/replicates the plate layout that embeds pooled QC, short and batch replicates needed to estimate within- and between-batch unwanted variation.) — https://github.com/SydneyBioX/hRUV
- **R** (Language in which the hRUV package and downstream replicate-SD computations are run.) — https://github.com/SydneyBioX/hRUV

## Examples

```
library(hRUV); norm <- hRUV::runHRUV(abund_matrix, replicate_design, negativeControls = NULL);
```

## Evaluation signals

- Per-replicate-set standard deviation of log-abundance across replicate members is lower for hRUV than for single-step RUV (as in Fig. 3b).
- Biological signal (e.g. hypertension-associated metabolite differences) is retained after normalisation, not attenuated alongside unwanted variation.
- All replicate sets (pooled QC, short replicates, batch replicates) can be unambiguously identified from the replicate design before normalisation.
- No missing values remain in the matrix passed to hRUV (imputation must precede normalisation).
- Results are reproducible across repeated runs given the same replicate design and negative-control metabolite set.

## Limitations

- hRUV expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround.
- By chance, replicate selection can draw batch replicates clustered from only one half of the previous plate, limiting the ability to capture unwanted variability across the whole plate (observed between batches 6 and 7, and 13 and 14).
- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on individual chemical properties; data-driven negative controls and all metabolites gave no significant difference in the paper.
- Residual slight shift in signal between affected batches may remain for selected metabolites (documented only in Supplementary Fig 5c).

## Evidence

- [abstract] We integrate these replicates in a novel workflow to remove unwanted variation in a hierarchical structure (hRUV) by progressively merging the adjustments in neighbouring batches: "We integrate these replicates in a novel workflow to remove unwanted variation in a hierarchical structure (hRUV) by progressively merging the adjustments in neighbouring batches"
- [discussion] metabolites that were not present in at least 50% of the samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs: "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [results] Figure 3b shows lower standard deviation between replicate sets for hierarchical normalization methods (orange and red) than for single-step ones (blue and green), indicating more effective batch-correction: [evidence span withheld — anchor status: not_found]
- [discussion] While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] `hRUV` is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies: "`hRUV` is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies"
- [discussion] The replicate design is available as a function plateDesign in the hRUV package: "The replicate design is available as a function plateDesign in the hRUV package"
