---
name: replicate-design-construction-for-metabolomics
description: Use when planning a multi-plate, multi-day targeted metabolomics run
  (e.g., >1,000 plasma samples over 44 days) where signal drift and inter-plate batch
  effects must be estimable from embedded replicates before normalisation.
when_to_use_negative:
- Data have already been acquired without embedded sample replicates — the design
  must be specified before the run, since RUV-III estimation depends on the physical
  replicate positions.
- Small single-batch/single-plate studies where signal drift over months and inter-plate
  variation are not the dominant unwanted variation.
- The user only needs downstream normalisation of an existing matrix and already possesses
  a replicate design table (use hRUV normalisation directly instead).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: hRUV
  role: R package providing the plateDesign function to generate the replicate embedding
    arrangements used within and between batches
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Web interface for designing replicate embedding arrangements within and between
    batches
  repo: http://shiny.maths.usyd.edu.au/hRUV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:569222afe7c35d1676d32e0b90d3ed8fed67e9edc4cc5a30a48cb3d83b005c37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV Shiny application
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# replicate-design-construction-for-metabolomics

## Summary

Constructs an experimental plate/run layout that embeds biological and technical sample replicates (pooled QCs, short within-row replicates, and inter-plate batch replicates) throughout a large-scale metabolomics acquisition so that within- and between-batch unwanted variation can later be estimated with RUV-III/hRUV.

## When to use

Use when planning a multi-plate, multi-day targeted metabolomics run (e.g., >1,000 plasma samples over 44 days) where signal drift and inter-plate batch effects must be estimable from embedded replicates before normalisation.

## When NOT to use

- Data have already been acquired without embedded sample replicates — the design must be specified before the run, since RUV-III estimation depends on the physical replicate positions.
- Small single-batch/single-plate studies where signal drift over months and inter-plate variation are not the dominant unwanted variation.
- The user only needs downstream normalisation of an existing matrix and already possesses a replicate design table (use hRUV normalisation directly instead).

## Inputs

- Number of samples, plates (batches), and rows in the acquisition (e.g., BioHEART-CT plasma cohort run over 44 days)
- Plate geometry (96-well plate, run order within each plate)
- List of biological samples available for re-selection as batch/short replicates from the previous plate/row

## Outputs

- Replicate design table mapping each replicate type (pooled QC, short replicate, batch replicate) to well position and run order per plate (cf. Supplementary Table 1)
- Replicate-position metadata usable by the hRUV normalisation workflow to group replicate sets

## How to apply

Lay out each 96-well plate so that the first three wells are pooled QC samples, followed by a single pooled QC after every 10 runs. After the three initial pooled QCs, place 5 batch replicates: biological samples selected from the previous plate and re-run at the start of the current plate to link batches. Within each row, randomly select one biological sample from the previous row and replicate it at a random position in the row (short replicate) to capture within-plate drift. The authors recommend systematic rather than fully random replicate arrangements, and the hRUV package's plateDesign function (plus the shiny app) generates these embedding arrangements; the design should be fixed before acquisition because RUV-III depends on replicate positions. Note that random batch-replicate selection can by chance draw only from one half of the previous plate, which limits plate-wide variance capture (observed between batches 6–7 and 13–14).

## Related tools

- **hRUV** (R package providing the plateDesign function to generate the replicate embedding arrangements used within and between batches) — https://github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Web interface for designing replicate embedding arrangements within and between batches) — http://shiny.maths.usyd.edu.au/hRUV

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Each 96-well plate contains exactly 3 pooled QC samples in the first three wells, then one pooled QC after every 10 runs.
- Each plate contains exactly 5 batch replicates drawn from the previous plate, placed immediately after the three starting pooled QC samples.
- Each row contains exactly 1 short replicate: a biological sample from the previous row placed at a random position in the row.
- Batch-replicate source positions span both halves of the previous plate — clustering of replicates from only one half (as seen between batches 6–7 and 13–14) flags a degraded design.
- Every biological sample assigned to a replicate set appears in the design table with consistent positions, matching the replicate structure later consumed by hRUV/RUV-III.

## Limitations

- Random batch-replicate selection can by chance draw samples only from the first or second half of the previous plate, limiting ability to capture unwanted variability across the whole plate; this occurred between batches 6–7 and 13–14 in the BioHEART-CT study, leaving a slight residual signal shift for selected metabolites.
- The authors state systematic arrangements perform better with hRUV normalisation than fully random ones, so a purely random placement is a known weaker design.
- Full replicate positions were published only as Supplementary Table 1, which is not included in the article text.

## Evidence

- [discussion] Pooled QC plate layout: "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [discussion] Batch replicate rule: "after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [discussion] Short replicate rule: "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [abstract] Purpose of embedding replicates: "an arrangement to embed biological sample replicates to measure the variance within and between batches"
- [discussion] Systematic vs random design: "we believe that systematic arrangements perform better with hRUV normalization than fully random ones"
- [discussion] plateDesign tool support: "The replicate design is available as a function plateDesign in the hRUV package"
- [readme] Replicate types used by hRUV: "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III."
- [discussion] Batch replicate placement: "the samples to be replicated were selected randomly from the previous plate and the corresponding repeats (batch replicates) were placed at the start of the current plate"
