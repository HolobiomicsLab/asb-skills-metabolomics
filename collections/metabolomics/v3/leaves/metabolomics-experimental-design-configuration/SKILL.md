---
name: metabolomics-experimental-design-configuration
description: Use when planning acquisition for a large-scale targeted metabolomics
  cohort (e.g. LC-MS/MS batches run over 96-well plates across days/weeks) and a replicate-embedded
  plate arrangement is needed for downstream batch correction.
when_to_use_negative:
- Data have already been acquired and no replicates were embedded — use hRUV normalization
  only where replicate structure exists, since RUV-III relies on inter-batch sample
  replicates for batch correction
- A single small batch with no plate-to-plate variation — inter-plate batch replicates
  and the hierarchical design target large-scale multi-batch studies
tools:
- name: hRUV (R package, plateDesign function)
  role: Programmatically generates the experimental design with appropriate numbers
    and assignment of inserted replicates given plate size and number of inter-plate
    replicates
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Interactive alternative for generating the same replicate arrangement design
- name: R
  role: Randomization via the sample function for selecting short-replicate and batch-replicate
    samples
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:822afd0a33a96fcdb4802123dc848051b8556576dc40b88648e5ae479e5ff3b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV (R package, plateDesign function)
  - hRUV Shiny application
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-experimental-design-configuration

## Summary

Generates a 96-well plate layout embedding pooled QC samples, short (within-row) replicates, and inter-plate batch replicates so that unwanted variation in large-scale metabolomics experiments can be estimated and removed downstream with hRUV/RUV-III.

## When to use

Use when planning acquisition for a large-scale targeted metabolomics cohort (e.g. LC-MS/MS batches run over 96-well plates across days/weeks) and a replicate-embedded plate arrangement is needed for downstream batch correction.

## When NOT to use

- Data have already been acquired and no replicates were embedded — use hRUV normalization only where replicate structure exists, since RUV-III relies on inter-batch sample replicates for batch correction
- A single small batch with no plate-to-plate variation — inter-plate batch replicates and the hierarchical design target large-scale multi-batch studies

## Inputs

- Desired plate size (e.g. 96 wells)
- Desired number of inter-plate (batch) replicates
- List of biological samples per plate/batch

## Outputs

- 96-well assignment table with well position, sample type, and replicate identifiers
- replicate_design.csv

## How to apply

Call the plateDesign function in the hRUV R package (or the hRUV Shiny application) with the desired plate size (96 wells) and desired number of inter-plate replicates. Populate the first three wells with pooled QC samples and repeat a single pooled QC after every 10 runs. In each row, randomly select one biological sample from the previous row using the sample function in R and insert it at a random position as a short replicate; after the first three pooled QC wells, embed a random selection of 5 biological samples from the previous plate as batch replicates. Assemble the full 96-well assignment table (well position, sample type, replicate identifiers) and write it to replicate_design.csv; systematic arrangements are preferred over fully random ones for compatibility with hRUV normalization.

## Related tools

- **hRUV (R package, plateDesign function)** (Programmatically generates the experimental design with appropriate numbers and assignment of inserted replicates given plate size and number of inter-plate replicates) — https://github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Interactive alternative for generating the same replicate arrangement design)
- **R** (Randomization via the sample function for selecting short-replicate and batch-replicate samples)

## Examples

```
library(hRUV); design <- plateDesign(plate.size = 96, num.replicates = 5); write.csv(design, "replicate_design.csv")
```

## Evaluation signals

- Output table contains exactly 96 well entries
- Exactly 3 pooled QC samples occupy the first wells of each plate, and one pooled QC appears after every 10 runs
- Each row contains exactly one short replicate (a biological sample drawn from the previous row) at a random position
- Exactly 5 batch replicates from the previous plate are embedded after the first three pooled QC wells
- Written replicate_design.csv parses with columns for well position, sample type, and replicate identifiers

## Limitations

- By chance, replicate selection can draw samples clustered from only one half of the previous plate, limiting the ability to capture unwanted variability across the whole plate (observed between batches 6/7 and 13/14)
- The design's assumption that pooled QC samples remain identical may not hold over extended periods spanning months or years
- hRUV expects data without missing values downstream, which is often not possible in large-scale metabolomics and requires imputation

## Evidence

- [discussion] Pooled QC layout on each 96-well plate (3 at start, then one every 10 runs): "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [discussion] Short replicate definition: "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [discussion] Batch replicate count and placement: "after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [discussion] plateDesign function availability: "The replicate design is available as a function plateDesign in the hRUV package"
- [discussion] Randomization tooling: "All randomization was performed using the sample function in R."
- [abstract] Design rationale for hRUV: "an arrangement to embed biological sample replicates to measure the variance within and between batches"
- [discussion] Systematic over random arrangement: "we believe that systematic arrangements perform better with hRUV normalization than fully random ones"
- [discussion] Plate-wide coverage limitation: "This limits our ability to capture the unwanted variability across the whole plate"
