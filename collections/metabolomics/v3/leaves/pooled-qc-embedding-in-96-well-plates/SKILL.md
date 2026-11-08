---
name: pooled-qc-embedding-in-96-well-plates
description: Use when designing the run order / plate layout for a targeted metabolomics
  batch that will later be normalized with replicate-based methods (e.g. hRUV/RUV-III)
  and you need pooled QC, short-replicate and batch-replicate positions for 96-well
  plates.
when_to_use_negative:
- Data acquisition is already complete — this skill designs the experimental run order
  before acquisition and cannot retrofit replicates onto existing runs
- The downstream normalization does not rely on embedded sample replicates (e.g. methods
  that assume pooled QCs are identical over extended periods without inter-batch replicates)
- Plate format or run length differs substantially from 96-well/10-run spacing without
  re-deriving QC spacing and replicate counts
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: hRUV (R package, plateDesign function)
  role: Generates the replicate/plate design programmatically given plate size and
    number of inter-plate replicates
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Web interface alternative to the plateDesign function for producing the experimental
    replicate design
- name: R
  role: Used for randomization of replicate positions via the sample function
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:d21fadf83340d6bba40012bd66bccc8504a7955c48f5877f6904fb538687fcd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV (R package, plateDesign function)
  - hRUV Shiny application
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pooled-qc-embedding-in-96-well-plates

## Summary

Generates the hRUV replicate arrangement for a 96-well plate batch layout: three pooled QC samples in the first wells, one pooled QC after every 10 runs, one short replicate per row, and 5 batch replicates from the previous plate — enabling within- and between-batch unwanted variation estimation for large-scale metabolomics.

## When to use

Use when designing the run order / plate layout for a targeted metabolomics batch that will later be normalized with replicate-based methods (e.g. hRUV/RUV-III) and you need pooled QC, short-replicate and batch-replicate positions for 96-well plates.

## When NOT to use

- Data acquisition is already complete — this skill designs the experimental run order before acquisition and cannot retrofit replicates onto existing runs
- The downstream normalization does not rely on embedded sample replicates (e.g. methods that assume pooled QCs are identical over extended periods without inter-batch replicates)
- Plate format or run length differs substantially from 96-well/10-run spacing without re-deriving QC spacing and replicate counts

## Inputs

- Desired plate size (96-well)
- Desired number of inter-plate (batch) replicates (5 per plate in the published design)
- Biological sample list ordered by previous rows/plates

## Outputs

- replicate_design.csv — 96-well assignment table with well position, sample type (pooled QC / biological sample / short replicate / batch replicate), and replicate identifiers

## How to apply

Load the hRUV package in R (or use the hRUV Shiny application) and call the plateDesign function with the desired plate size and number of inter-plate replicates, or build the layout manually following the published convention. Populate the first three wells with pooled QC samples, then insert a single pooled QC after every 10 runs. In each row, use the R sample function to randomly select one biological sample from the previous row and place it at a random position within the current row (short replicate). After the first three pooled QC wells, embed a random selection of 5 biological samples drawn from the previous plate (batch replicates) at the start of the current plate. Assemble the full 96-well assignment table (well position, sample type, replicate identifiers) and write it to replicate_design.csv, validating that it contains 96 entries with exactly 3 pooled QCs in the first wells, one pooled QC per 10 runs, one short replicate per row, and 5 batch replicates.

## Related tools

- **hRUV (R package, plateDesign function)** (Generates the replicate/plate design programmatically given plate size and number of inter-plate replicates) — https://github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Web interface alternative to the plateDesign function for producing the experimental replicate design)
- **R** (Used for randomization of replicate positions via the sample function)

## Examples

```
library(hRUV); design <- plateDesign(plate_size = 96, n_interplate_replicates = 5); write.csv(design, "replicate_design.csv", row.names = FALSE)
```

## Evaluation signals

- Output table contains exactly 96 well entries
- Exactly 3 pooled QC samples occupy the first wells of the plate
- A single pooled QC appears after every 10 runs
- Each row contains exactly one short replicate drawn from the previous row
- Exactly 5 batch replicates from the previous plate appear after the first three pooled QC wells

## Limitations

- By chance, random replicate selection can draw samples clustered from only one half of the previous plate, preventing even plate-wide positional coverage (observed between batches 6–7 and 13–14)
- Such clustering limits the ability to capture unwanted variability across the whole plate and produced a slight residual shift in signal between affected batches
- The authors note systematic replicate arrangements generally perform better with hRUV normalization than fully random ones, so purely random placement is not guaranteed optimal

## Evidence

- [discussion] the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs: "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [discussion] we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate): "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [discussion] after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate): "after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [discussion] All randomization was performed using the sample function in R.: "All randomization was performed using the sample function in R."
- [discussion] The replicate design is available as a function plateDesign in the hRUV package: "The replicate design is available as a function plateDesign in the hRUV package"
