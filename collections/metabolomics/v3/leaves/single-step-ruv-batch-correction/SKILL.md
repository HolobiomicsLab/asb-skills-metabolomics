---
name: single-step-ruv-batch-correction
description: Use when a multi-batch metabolomics abundance matrix (e.g. BioHEART-CT
  targeted metabolomics) contains embedded intra-batch and inter-batch replicates
  and you need a baseline batch-correction with RUV-III, or to benchmark against hierarchical
  normalisation.
when_to_use_negative:
- When the goal is the paper's improved batch correction itself — hRUV hierarchically
  merges adjustments of neighbouring batches and achieves lower replicate SD than
  single-step RUV (Fig. 3b)
- When data contain missing values that cannot be imputed — the algorithm expects
  complete data, which is often not possible in large-scale metabolomics
- When replicates were placed fully at random rather than in the systematic embedding
  arrangement — the authors state systematic arrangements perform better with hRUV/RUV
  normalisation than fully random ones
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: R package providing RUV-III-based normalisation and the plateDesign function
    for generating the replicate structure used as input to single-step correction
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV (R package, plateDesign function)
  role: Generates the replicate design (pooled QC, short replicates, batch replicates)
    required to estimate unwanted variation with RUV-III
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Runtime environment; the hRUV package was tested with R version 4.0.3 on macOS
    Big Sur 11.1 and Linux Debian 10
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:0e623d64297f672fe4db55a9d728b80c58826e9f1f98edafe8becc9a921e55ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV (R package, plateDesign function)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-step-ruv-batch-correction

## Summary

Applies single-step RUV-III normalisation to large-scale metabolomics data using embedded sample replicates (pooled QC, short replicates, batch replicates) to remove unwanted variation across all batches in one pass, serving as the non-hierarchical baseline for comparison against hRUV.

## When to use

Use when a multi-batch metabolomics abundance matrix (e.g. BioHEART-CT targeted metabolomics) contains embedded intra-batch and inter-batch replicates and you need a baseline batch-correction with RUV-III, or to benchmark against hierarchical normalisation.

## When NOT to use

- When the goal is the paper's improved batch correction itself — hRUV hierarchically merges adjustments of neighbouring batches and achieves lower replicate SD than single-step RUV (Fig. 3b)
- When data contain missing values that cannot be imputed — the algorithm expects complete data, which is often not possible in large-scale metabolomics
- When replicates were placed fully at random rather than in the systematic embedding arrangement — the authors state systematic arrangements perform better with hRUV/RUV normalisation than fully random ones

## Inputs

- Filtered metabolite abundance matrix from BioHEART-CT targeted metabolomics (metabolites present in >= 50% of samples, missing values imputed)
- Replicate design information: pooled QC positions (3 at plate start, then one every 10 runs), short within-row replicates, and 5 batch replicates from the previous plate per plate
- Negative-control metabolite specification (all metabolites, or non-differentially expressed metabolites ranked by p-values per batch)

## Outputs

- Single-step RUV-III normalised abundance matrix
- Per-metabolite, per-replicate-set standard deviation table (e.g. results_table.csv) for comparison against the hRUV-normalised matrix

## How to apply

Load the filtered metabolite abundance matrix (metabolites present in at least 50% of samples, missing values imputed since RUV expects complete data) together with the replicate design (pooled QC at 3 per plate start plus one every 10 runs, short within-row replicates, and 5 batch replicates from the previous plate per plate) generated via the plateDesign function in the hRUV package. Apply single-step RUV-III to the whole experiment at once — i.e. without the progressive merging of adjustments in neighbouring batches that characterises hRUV — using either all metabolites or data-driven negative controls (the article reports no significant difference between these two negative-control choices, while inter-batch sample replicates were more effective for batch correction). Then, for each replicate set (pooled QC, short replicates, batch replicates), compute the standard deviation of log-abundance measurements across replicate members for every metabolite. These per-replicate-set SDs are the metric by which the single-step result is compared with hRUV, which the paper's Figure 3b shows achieves lower replicate SD.

## Related tools

- **hRUV** (R package providing RUV-III-based normalisation and the plateDesign function for generating the replicate structure used as input to single-step correction) — https://github.com/SydneyBioX/hRUV
- **hRUV (R package, plateDesign function)** (Generates the replicate design (pooled QC, short replicates, batch replicates) required to estimate unwanted variation with RUV-III) — https://github.com/SydneyBioX/hRUV
- **R** (Runtime environment; the hRUV package was tested with R version 4.0.3 on macOS Big Sur 11.1 and Linux Debian 10)

## Examples

```
library(devtools); devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Replicate SDs for the single-step normalised matrix should be higher than those from the hierarchical hRUV matrix (orange/red vs blue/green in Fig. 3b), confirming hRUV's more effective between-batch correction
- The per-replicate-set SD table should cover all three replicate types: pooled QC, short (within-row) replicates, and batch replicates
- No missing values should remain in the normalised matrix after the imputation workaround, since the RUV algorithm expects complete data
- Comparing negative-control choices (data-driven ranked metabolites vs all metabolites) should show no significant difference in removal of unwanted variation
- Biological signal should be retained — hRUV-family normalisation is judged on retention of biological signal, low variability among replicates, and reproducibility

## Limitations

- hRUV/RUV-III expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround
- Random batch-replicate selection can draw samples clustered from only one half of the previous plate (observed between batches 6 and 7, and 13 and 14), limiting the ability to capture unwanted variability across the whole plate and causing a slight residual shift in signal for selected metabolites (Supplementary Fig 5c)
- Single-step application does not progressively merge adjustments of neighbouring batches, and thus removes between-batch unwanted variation less effectively than the hierarchical strategy

## Evidence

- [readme] The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III.: "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III."
- [intro] RUV-III replicate estimation: "estimation of unwanted variation within and between batches with RUV-III12"
- [discussion] Negative-control equivalence and inter-batch replicate utility: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] plateDesign function: "The replicate design is available as a function plateDesign in the hRUV package"
- [discussion] Missing-value requirement: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [intro] Assessment criteria: "we utilise this to assess the normalization on a number of criteria including retention of biological signal, low variability among replication, and reproducibility"
