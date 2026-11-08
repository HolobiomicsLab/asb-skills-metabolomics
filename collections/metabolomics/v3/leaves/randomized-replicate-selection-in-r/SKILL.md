---
name: randomized-replicate-selection-in-r
description: Use when planning multi-plate/multi-batch metabolomics acquisition and
  you need an embedded replicate layout (pooled QC, short within-row replicates, batch
  replicates) before running samples.
when_to_use_negative:
- 'You need post-acquisition normalization: plateDesign only generates the experimental
  layout; normalization is performed later by the hRUV workflow on acquired data.'
- 'Pooled QC-only designs suffice (no inter-batch biological replicates needed): the
  hRUV discussion notes inter-batch sample replicates were the more effective batch-correction
  mechanism, but simpler designs may not require randomized replicate insertion.'
- 'Your plate format differs fundamentally (e.g. 384-well): the published convention
  — 3 QC wells first, QC every 10 runs, one short replicate per row, 5 batch replicates
  per 96-well plate — was validated on 96-well plates.'
tools:
- name: hRUV (R package, plateDesign function)
  role: Generates the replicate design by taking the plate size and desired number
    of inter-plate replicates
  repo: github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Interactive alternative to the plateDesign R function for generating the same
    experimental design
- name: R
  role: All replicate randomization (short and batch replicate selection) is performed
    with the sample function
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:67865a058f8b42688dd8f3f079a08e4cbaf4698774c570a0b57f8dc5d539ed34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV (R package, plateDesign function)
  - hRUV Shiny application
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# randomized-replicate-selection-in-r

## Summary

Generates a 96-well plate replicate arrangement for large-scale metabolomics by calling hRUV's plateDesign (or its shiny app) and randomizing short and batch replicates with R's sample function, so unwanted variation within and between batches can be estimated with RUV-III.

## When to use

Use when planning multi-plate/multi-batch metabolomics acquisition and you need an embedded replicate layout (pooled QC, short within-row replicates, batch replicates) before running samples.

## When NOT to use

- You need post-acquisition normalization: plateDesign only generates the experimental layout; normalization is performed later by the hRUV workflow on acquired data.
- Pooled QC-only designs suffice (no inter-batch biological replicates needed): the hRUV discussion notes inter-batch sample replicates were the more effective batch-correction mechanism, but simpler designs may not require randomized replicate insertion.
- Your plate format differs fundamentally (e.g. 384-well): the published convention — 3 QC wells first, QC every 10 runs, one short replicate per row, 5 batch replicates per 96-well plate — was validated on 96-well plates.

## Inputs

- Desired plate size (96 wells)
- Desired number of inter-plate replicates (5 per plate)
- Sample list / plate assignment table for the previous plate (source of batch replicates)
- Row-wise sample ordering of the previous row (source of short replicates)

## Outputs

- 96-well assignment table mapping well position to sample type and replicate identifiers
- replicate_design.csv file for downstream acquisition

## How to apply

Call hRUV's plateDesign function (or the equivalent expDesign functionality in the shiny app) supplying the desired plate size (96 wells) and the desired number of inter-plate replicates. On each plate, populate the first three wells with pooled QC samples and insert one pooled QC after every 10 runs for signal-drift monitoring. Using R's sample function, randomly select a single biological sample from the previous row to repeat at a random position within each row (short replicate), and after the first three pooled QC wells embed 5 biological samples randomly drawn from the previous plate at the start of the current plate (batch replicates). The randomization is intentional — replicate placement must not be fully random across the plate, since systematic arrangements were judged to perform better with hRUV normalization than fully random ones. Write the resulting table (well position, sample type, replicate identifiers) to replicate_design.csv.

## Related tools

- **hRUV (R package, plateDesign function)** (Generates the replicate design by taking the plate size and desired number of inter-plate replicates) — github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Interactive alternative to the plateDesign R function for generating the same experimental design)
- **R** (All replicate randomization (short and batch replicate selection) is performed with the sample function)

## Examples

```
batch_reps <- sample(prev_plate_ids, 5); short_rep <- sample(prev_row_ids, 1)  # per published convention; full layout via hRUV::plateDesign(96, 5)
```

## Evaluation signals

- Output table contains exactly 96 well entries for a single 96-well plate.
- The first three wells hold pooled QC samples, and one pooled QC appears after every 10 runs thereafter.
- Each row contains exactly one short replicate: a biological sample drawn from the previous row, placed at a random position within that row.
- After the first three pooled QC wells there are exactly 5 batch replicates, each drawn from the previous plate and placed at the start of the current plate.
- The table is exported as replicate_design.csv with well position, sample type, and replicate identifier columns.

## Limitations

- By chance, replicate selection can draw samples clustered from only the first or only the second half of the previous plate, limiting the design's ability to capture unwanted variability across the whole plate (observed between batches 6/7 and 13/14, producing a slight residual signal shift for selected metabolites).
- This limitation surfaced as unintended clustering of selected batch replicates documented only in Supplementary Table 1, which is not included in the article text.
- hRUV downstream expects data without missing values, so the replicate design feeds into an analysis requiring an imputation workaround in large-scale metabolomics.

## Evidence

- [discussion] the samples to be replicated were selected randomly from the previous plate and the corresponding repeats (batch replicates) were placed at the start of the current plate: "the samples to be replicated were selected randomly from the previous plate and the corresponding repeats (batch replicates) were placed at the start of the current plate"
- [discussion] we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate): "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [discussion] The replicate design is available as a function plateDesign in the hRUV package: "The replicate design is available as a function plateDesign in the hRUV package"
- [readme] harnessing information from sample replicates embedded in the seequence of experimental runs/batches: "harnessing information from sample replicates embedded in the seequence of experimental runs/batches"
