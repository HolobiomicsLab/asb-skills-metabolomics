---
name: hierarchical-batch-merge-orchestration
description: Use when a large-scale metabolomics study spans multiple acquisition
  batches (e.g. 96-well plates across many days) with inter-batch replicate pairs
  embedded, and between-batch unwanted variation must be removed via RUV-III in a
  hierarchical merge design.
when_to_use_negative:
- Data are single-batch or lack inter-batch replicate pairs, since between-batch unwanted
  variation cannot be estimated with RUV-III replicate pairs
- Data contain missing values and no imputation step has been applied — hRUV expects
  complete data
- A within-batch signal-drift correction is still needed (this skill only handles
  the between-batch merge stage, not drift smoothing)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: hRUV
  role: R package implementing the hierarchical between-batch RUV-III correction and
    providing the replicate-design/batch-merge machinery and example datasets
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Execution environment for the hRUV package (tested with R 4.0.3); used to
    run and script the per-stage merge pipeline and CSV export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:6ee3ec460fe3fe8274c73ce7247b91088a9d194a7c9a6a2fa6443a8d75d3e57a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hierarchical-batch-merge-orchestration

## Summary

Orchestrates the inter-batch stage of hRUV by merging batches under either a balanced tree (log2(n) RUV-III adjustments) or a concatenating tree (n-1 adjustments), progressively removing unwanted variation between batches of large-scale metabolomics data.

## When to use

Use when a large-scale metabolomics study spans multiple acquisition batches (e.g. 96-well plates across many days) with inter-batch replicate pairs embedded, and between-batch unwanted variation must be removed via RUV-III in a hierarchical merge design.

## When NOT to use

- Data are single-batch or lack inter-batch replicate pairs, since between-batch unwanted variation cannot be estimated with RUV-III replicate pairs
- Data contain missing values and no imputation step has been applied — hRUV expects complete data
- A within-batch signal-drift correction is still needed (this skill only handles the between-batch merge stage, not drift smoothing)

## Inputs

- Normalized metabolomics data matrix with no missing values (samples x metabolites)
- Batch/replicate design annotations (pooled QC, short replicates, batch replicates) from the hRUV package example datasets
- Batch replicate pairs: replicate IDs linking consecutive batches

## Outputs

- hRUV-adjusted metabolite abundance matrix (balanced-tree design)
- hRUV-adjusted metabolite abundance matrix (concatenating-tree design)
- Per-stage intermediate adjustment outputs recorded for each hierarchical merge, exported to CSV

## How to apply

Load the normalized metabolomics data matrix (verified to contain no missing values) plus the replicate design annotations — pooled QC, short replicates, and batch replicates recorded as replicate-ID pairs across consecutive batches. Choose a tree structure: the balanced tree merges neighbouring batches in log2(n) hierarchical stages, while the concatenating tree sequentially combines the current merged data with the next batch, requiring n-1 total adjustments — more computation but easing future integration of new batches. At each merge, apply RUV-III to the replicate pairs linking the two child batches with negative-control columns set to all metabolites (the article found no significant difference between data-derived negative controls and all metabolites). Record the per-stage adjustment outputs and assemble the final adjusted metabolite abundance matrices for both the balanced and concatenating designs, saving full and per-stage intermediates to CSV.

## Related tools

- **hRUV** (R package implementing the hierarchical between-batch RUV-III correction and providing the replicate-design/batch-merge machinery and example datasets) — https://github.com/SydneyBioX/hRUV
- **R** (Execution environment for the hRUV package (tested with R 4.0.3); used to run and script the per-stage merge pipeline and CSV export)

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE); library(hRUV)
```

## Evaluation signals

- Adjusted output contains no missing values
- Output preserves the original number of samples and metabolites
- Variability among embedded replicate pairs is reduced relative to the input normalized data (manuscript criterion of low variability among replication)
- Balanced-tree design applies exactly log2(n) RUV adjustments for n batches; concatenating-tree design applies exactly n-1
- Signal between adjacent batches no longer shows shifts for selected metabolites after correction

## Limitations

- The hRUV algorithm expects data without missing values, often impossible in large-scale metabolomics, requiring an imputation workaround
- The concatenating tree costs more computation than the balanced tree (n-1 vs log2(n) adjustments), though it eases future integration of new batches
- By chance, replicate selection can draw samples clustered from only one half of a plate (e.g. between batches 6 and 7 and between batches 13 and 14), limiting capture of unwanted variability across the whole plate and leaving a slight signal shift between affected batches

## Evidence

- [abstract] hierarchical inter-batch correction via progressive merging: "We integrate these replicates in a novel workflow to remove unwanted variation in a hierarchical structure (hRUV) by progressively merging the adjustments in neighbouring batches"
- [intro] between-batch estimation with RUV-III: "estimation of unwanted variation within and between batches with RUV-III12"
- [discussion] tree-structure schematics referenced: "as illustrated in schematics in Figure 1d"
- [discussion] performance of hierarchical correction: "hRUV performs consistently better than them in retaining biological variation whilst at the same time removing unwanted variation within and across batches."
- [readme] hRUV package purpose per README: "hRUV` is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies."
