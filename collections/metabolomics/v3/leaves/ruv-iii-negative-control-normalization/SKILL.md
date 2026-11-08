---
name: ruv-iii-negative-control-normalization
description: Use when a multi-batch metabolomics abundance matrix contains embedded
  replicate pairs (pooled QC, short replicates, batch replicates) and inter-batch
  unwanted variation must be removed with RUV-III before downstream cohort analysis.
when_to_use_negative:
- The dataset contains missing values and imputation has not yet been performed —
  hRUV/RUV-III expects data without missing values
- The study is a single batch with no inter-batch replicate pairs, so the between-batch
  RUV-III correction has no replicate pairs to estimate unwanted variation
- The goal is only signal-drift (within-run) correction with robust linear or non-linear
  smoothers rather than inter-batch removal of unwanted variation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: hRUV
  role: R package wrapping the hierarchical RUV-III adjustment for intra- and inter-batch
    replicates; also exposed as a Shiny application
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Runtime language for the hRUV package (tested on R 4.0.3) in which RUV-III
    adjustment is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:dab64032285c2617bcde3916773b5a35525cec4a60240652fa3492a2e2a10381
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ruv-iii-negative-control-normalization

## Summary

Applies RUV-III, using embedded biological sample replicate pairs as controls and metabolite negative-control columns, to remove unwanted variation within and between metabolomics batches, as implemented in the hRUV hierarchical workflow.

## When to use

Use when a multi-batch metabolomics abundance matrix contains embedded replicate pairs (pooled QC, short replicates, batch replicates) and inter-batch unwanted variation must be removed with RUV-III before downstream cohort analysis.

## When NOT to use

- The dataset contains missing values and imputation has not yet been performed — hRUV/RUV-III expects data without missing values
- The study is a single batch with no inter-batch replicate pairs, so the between-batch RUV-III correction has no replicate pairs to estimate unwanted variation
- The goal is only signal-drift (within-run) correction with robust linear or non-linear smoothers rather than inter-batch removal of unwanted variation

## Inputs

- Normalized metabolomics data matrix (no missing values) with samples as rows and metabolite abundance columns
- Batch/replicate design annotations: pooled QC, short (within-row) replicate, and batch replicate pair IDs across consecutive batches

## Outputs

- RUV-III-adjusted metabolite abundance matrix (same samples × metabolites as input)
- Per-stage adjustment intermediates when run hierarchically (balanced or concatenating tree)

## How to apply

Load the normalized metabolomics matrix together with the replicate design annotations (pooled QC, short/within-row replicates, and batch replicates placed from the previous plate onto the start of the current plate), and confirm the data contain no missing values since RUV-III expects complete data. Supply the replicate pairs linking batches and set negative-control columns to all metabolites — the article found no significant difference between data-driven negative-control selection and using all metabolites, while inter-batch sample replicates were the more effective driver of batch correction. Apply RUV-III either within a single batch adjustment or hierarchically inside hRUV, merging neighbouring batches in log2(n) stages (balanced tree) or sequentially in n-1 stages (concatenating tree). Record per-stage adjusted matrices and validate that sample and metabolite counts are unchanged and replicate-pair variability is reduced relative to the input normalized data.

## Related tools

- **hRUV** (R package wrapping the hierarchical RUV-III adjustment for intra- and inter-batch replicates; also exposed as a Shiny application) — https://github.com/SydneyBioX/hRUV
- **R** (Runtime language for the hRUV package (tested on R 4.0.3) in which RUV-III adjustment is executed)

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Adjusted output contains no missing values (matches hRUV's complete-data requirement)
- Adjusted matrix preserves the original number of samples and metabolites
- Variability among embedded replicate pairs is reduced relative to the input normalized data (manuscript criterion of low variability among replication)
- Biological signal is retained (hRUV outperformed 13 compared normalization methods on retention of biological variation while removing unwanted variation)
- Batch-replicate clustering gaps (e.g. between batches 6 and 7, and 13 and 14) show no residual signal shift for selected metabolites

## Limitations

- hRUV expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround
- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on individual chemical properties
- Random batch-replicate selection can cluster replicates from only one half of the previous plate (seen between batches 6 and 7, and 13 and 14), limiting capture of unwanted variability across the whole plate and leaving slight residual signal shifts

## Evidence

- [intro] estimation of unwanted variation within and between batches with RUV-III12: "estimation of unwanted variation within and between batches with RUV-III12"
- [discussion] no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III: "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III."
