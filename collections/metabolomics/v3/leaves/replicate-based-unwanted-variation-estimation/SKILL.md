---
name: replicate-based-unwanted-variation-estimation
description: Use when large-scale metabolomics batches contain embedded sample replicates
  (pooled QC, short replicates, batch replicates) and unwanted variation within and
  between batches must be estimated with RUV-III.
when_to_use_negative:
- Data contain missing values that cannot be imputed — hRUV expects data without missing
  values, which is often not possible in large-scale metabolomics data
- No inter-batch sample replicates are embedded, so between-batch variation cannot
  be estimated from replicate pairs
- Input is raw/unfiltered data — metabolites not present in at least 50% of samples
  must first be filtered out
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: hRUV
  role: R package implementing the hierarchical RUV-III adjustment using intra- and
    inter-batch replicates
  repo: github.com/SydneyBioX/hRUV
- name: hRUV plateDesign
  role: Function to generate the replicate design (replicate positions) for each plate
  repo: github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Web interface for designing replicate embedding arrangements within and between
    batches
- name: R
  role: Runtime environment; all randomization of replicate selection performed with
    the sample function in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:7475d0e81f63c39026e5973eac81bda915c1afbc0a0d6a1285a446ee5d87bf1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  - hRUV Shiny application
  - hRUV plateDesign
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# replicate-based-unwanted-variation-estimation

## Summary

Estimates and removes unwanted variation in large-scale metabolomics data by embedding biological sample replicates (pooled QC, short within-row replicates, inter-batch replicates) and applying RUV-III hierarchically across batches (hRUV).

## When to use

Use when large-scale metabolomics batches contain embedded sample replicates (pooled QC, short replicates, batch replicates) and unwanted variation within and between batches must be estimated with RUV-III.

## When NOT to use

- Data contain missing values that cannot be imputed — hRUV expects data without missing values, which is often not possible in large-scale metabolomics data
- No inter-batch sample replicates are embedded, so between-batch variation cannot be estimated from replicate pairs
- Input is raw/unfiltered data — metabolites not present in at least 50% of samples must first be filtered out

## Inputs

- Normalized metabolomics data matrix (samples x metabolites) with no missing values
- Batch/replicate design annotations: pooled QC positions, short (within-row) replicate pairs, inter-batch replicate pairs across consecutive batches

## Outputs

- hRUV-adjusted metabolite abundance matrix (same samples and metabolites as input)
- Replicate design object from plateDesign for each plate

## How to apply

Embed replicates in each 96-well plate per the systematic layout: three pooled QC samples in the first wells, one pooled QC after every 10 runs, one short (within-row) replicate of a biological sample from the previous row, and 5 batch replicates selected randomly from the previous plate and placed at its start. Obtain the replicate design via the plateDesign function in the hRUV R package, confirm the data contain no missing values, and correct signal drift with robust linear or non-linear smoothers. Then apply RUV-III using replicate pairs linking consecutive batches, setting negative-control columns to all metabolites since data-driven negative controls showed no significant advantage over all metabolites in the authors' assessment. Merge batches hierarchically — either a balanced tree (log2(n) adjustments) or a concatenating tree (n-1 adjustments) — to produce the final adjusted abundance matrix while preserving biological signal.

## Related tools

- **hRUV** (R package implementing the hierarchical RUV-III adjustment using intra- and inter-batch replicates) — github.com/SydneyBioX/hRUV
- **hRUV plateDesign** (Function to generate the replicate design (replicate positions) for each plate) — github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Web interface for designing replicate embedding arrangements within and between batches)
- **R** (Runtime environment; all randomization of replicate selection performed with the sample function in R)

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Adjusted output contains no missing values
- Adjusted matrix preserves the original number of samples and metabolites
- Variability among embedded replicate pairs (short, batch, pooled QC) is reduced relative to the input normalized data
- Normalization judged on retention of biological signal, low variability among replication, and reproducibility against 13 publicly available metabolomics normalization methods
- Inter-batch replicates used for batch correction show effective removal of unwanted variation per the manuscript's Supplementary Fig. 7 comparison

## Limitations

- By chance, replicate selection can draw batch-replicate samples from only one half of the previous plate, limiting capture of unwanted variability across the whole plate (observed between batches 6-7 and 13-14)
- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on individual chemical properties
- Systematic replicate arrangements perform better with hRUV normalization than fully random ones; random embedding risks unintended clustering
- Requires R version 4.0.3-era tooling (tested on macOS Big Sur 11.1 and Linux Debian 10) with CRAN and Bioconductor dependencies (SummarizedExperiment, impute)

## Evidence

- [abstract] Replicate embedding rationale: "an arrangement to embed biological sample replicates to measure the variance within and between batches"
- [intro] RUV-III estimation step: "estimation of unwanted variation within and between batches with RUV-III12"
- [discussion] Negative-control comparison: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] plateDesign function: "The replicate design is available as a function plateDesign in the hRUV package"
- [readme] Two replicate types: "utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III"
