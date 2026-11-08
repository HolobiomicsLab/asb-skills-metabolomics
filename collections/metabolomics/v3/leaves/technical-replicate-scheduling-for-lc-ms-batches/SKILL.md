---
name: technical-replicate-scheduling-for-lc-ms-batches
description: Use when planning acquisition of a large-scale targeted metabolomics
  batch series (e.g. >1,000 plasma samples over 44 days) and you need a concrete 96-well
  plate layout embedding pooled QCs, short replicates, and batch replicates before
  running samples.
when_to_use_negative:
- The samples have already been run — replicate embedding must be designed before
  acquisition and cannot be retrofitted to existing LC-MS/MS sequence files
- Single-plate or small-cohort experiments with no batch series, where inter-plate
  batch replicates and RUV-III batch correction are not needed
- The goal is drift correction or negative-control selection itself (later hRUV stages),
  not the experimental plate arrangement
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: hRUV (R package, plateDesign function)
  role: Generates the replicate design / 96-well assignment table from plate size
    and number of inter-plate replicates
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Interactive web interface to produce the embedded replicate arrangement for
    a given plate size and replicate count
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Used via the sample function to randomize short-replicate selection within
    rows and batch-replicate selection across plates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:ae042df634f9ca4de6e639523b7c6e9bfe77ec48593d103f9f01ee293d343aab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV (R package, plateDesign function)
  - hRUV Shiny application
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# technical-replicate-scheduling-for-lc-ms-batches

## Summary

Generates the embedded replicate arrangement for 96-well LC-MS/MS metabolomics plates: pooled QCs, short (within-row) replicates, and inter-plate batch replicates, so that within- and between-batch unwanted variation can later be estimated with RUV-III in the hRUV workflow.

## When to use

Use when planning acquisition of a large-scale targeted metabolomics batch series (e.g. >1,000 plasma samples over 44 days) and you need a concrete 96-well plate layout embedding pooled QCs, short replicates, and batch replicates before running samples.

## When NOT to use

- The samples have already been run — replicate embedding must be designed before acquisition and cannot be retrofitted to existing LC-MS/MS sequence files
- Single-plate or small-cohort experiments with no batch series, where inter-plate batch replicates and RUV-III batch correction are not needed
- The goal is drift correction or negative-control selection itself (later hRUV stages), not the experimental plate arrangement

## Inputs

- Desired plate size (e.g. single 96-well plate)
- Desired number of inter-plate (batch) replicates (5 per plate in the published layout)
- Ordered list of biological samples per plate and per row
- Pooled QC sample material

## Outputs

- 96-well plate assignment table with well position, sample type, and replicate identifiers
- replicate_design.csv

## How to apply

Call the plateDesign function in the hRUV R package (or the hRUV Shiny application), providing the desired plate size (96 wells) and the desired number of inter-plate replicates. Populate the first three wells of each plate with three pooled QC samples, then repeat a single pooled QC after every 10 runs. Randomly select one biological sample from the previous row (using the sample function in R) and place it at a random position within the current row as a short replicate. After the first three pooled QC wells, embed a random selection of 5 biological samples from the previous plate at the start of the current plate as batch replicates. Assemble the full 96-well assignment table (well position, sample type, replicate identifiers) and write it to replicate_design.csv; systematic arrangements were preferred over fully random ones for hRUV normalization.

## Related tools

- **hRUV (R package, plateDesign function)** (Generates the replicate design / 96-well assignment table from plate size and number of inter-plate replicates) — https://github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Interactive web interface to produce the embedded replicate arrangement for a given plate size and replicate count) — https://github.com/SydneyBioX/hRUV
- **R** (Used via the sample function to randomize short-replicate selection within rows and batch-replicate selection across plates)

## Examples

```
library(hRUV); replicate_design <- plateDesign(plateSize = 96, nInterPlateReplicates = 5); write.csv(replicate_design, "replicate_design.csv", row.names = FALSE)
```

## Evaluation signals

- Output table contains exactly 96 well entries
- First three wells are populated with three pooled QC samples
- A single pooled QC sample appears after every 10 runs
- Exactly one short replicate (a biological sample from the previous row) appears at a random position in each row
- Exactly 5 batch replicates drawn from the previous plate are placed after the first three pooled QC wells at the start of the plate

## Limitations

- By chance, random replicate selection can draw samples from only the first or second half of the previous plate; this unintended clustering was observed between batches 6 and 7 and between batches 13 and 14, limiting the ability to capture unwanted variability across the whole plate and producing a slight residual signal shift for selected metabolites
- The authors note that systematic arrangements were believed to perform better with hRUV than fully random ones, so fully random layouts may be suboptimal
- Downstream, hRUV expects data without missing values, which is often not possible in large-scale metabolomics and requires an imputation workaround

## Evidence

- [discussion] Pooled QC layout convention: "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [discussion] Short replicate convention: "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [discussion] Batch replicate convention: "after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [discussion] plateDesign function availability: "The replicate design is available as a function plateDesign in the hRUV package"
- [discussion] Randomization tooling: "All randomization was performed using the sample function in R."
- [readme] Shiny app provenance of designs: "We have designed the replicate embedding arrangements within and between batches from http://shiny.maths.usyd.edu.au/hRUV/."
