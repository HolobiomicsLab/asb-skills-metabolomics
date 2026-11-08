---
name: plate-layout-replicate-design
description: Use when planning acquisition order for a large-scale targeted LC-MS/MS
  metabolomics run where a 96-well plate batch structure is used and downstream normalization
  needs embedded technical replicates to quantify within- and between-batch variance
  (e.g., hRUV/RUV-III batch correction).
when_to_use_negative:
- Studies without a multi-batch / multi-plate acquisition structure where inter-batch
  variance estimation is unnecessary (e.g., a single batch or a design where all samples
  are randomized into one run) — embedding replicates would waste sample volume without
  downstream hRUV benefit.
- Analyses where signal drift correction alone suffices and inter-plate replicates
  are unavailable or impractical to acquire (e.g., scarce patient samples) — the design
  presumes the ability to re-run selected biological samples from the previous plate.
- Downstream pipelines that do not support RUV-III-style normalization using replicate
  information; the replicate embedding is specifically designed to feed hRUV hierarchical
  unwanted-variation removal and may not benefit other correction methods.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: hRUV
  role: R package providing the plateDesign function to generate the replicate arrangement
    for a given plate size and number of inter-plate replicates; also the downstream
    normalization that consumes the design.
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV shiny application
  role: Interactive web interface (http://shiny.maths.usyd.edu.au/hRUV) for generating
    the replicate embedding arrangement for large-scale metabolomics batches.
  repo: http://shiny.maths.usyd.edu.au/hRUV
- name: R
  role: Language used to call the plateDesign function and to perform randomization
    with the sample function during replicate selection.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:ba228e877b37e30afc3b1e052c33d3923faa514f14f8adc8b6495452e0205445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV shiny application
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# plate-layout-replicate-design

## Summary

Designs a 96-well plate layout that systematically embeds pooled QC samples, short (within-row) replicates, and inter-plate batch replicates, so that later hRUV normalization can estimate unwanted variation within and between batches in large-scale metabolomics studies.

## When to use

Use when planning acquisition order for a large-scale targeted LC-MS/MS metabolomics run where a 96-well plate batch structure is used and downstream normalization needs embedded technical replicates to quantify within- and between-batch variance (e.g., hRUV/RUV-III batch correction).

## When NOT to use

- Studies without a multi-batch / multi-plate acquisition structure where inter-batch variance estimation is unnecessary (e.g., a single batch or a design where all samples are randomized into one run) — embedding replicates would waste sample volume without downstream hRUV benefit.
- Analyses where signal drift correction alone suffices and inter-plate replicates are unavailable or impractical to acquire (e.g., scarce patient samples) — the design presumes the ability to re-run selected biological samples from the previous plate.
- Downstream pipelines that do not support RUV-III-style normalization using replicate information; the replicate embedding is specifically designed to feed hRUV hierarchical unwanted-variation removal and may not benefit other correction methods.

## Inputs

- desired plate size (e.g., 96-well plate dimensions)
- desired number of inter-plate (batch) replicates per plate (e.g., 5)
- ordered list of biological samples per plate

## Outputs

- 96-well plate assignment table mapping well position to sample type (pooled QC, biological sample, short replicate, batch replicate)
- replicate design with embedded replicate identifiers for downstream hRUV normalization

## How to apply

Populate the first three wells of each 96-well plate with three pooled QC samples, then place a single pooled QC sample after every 10 runs; per the article, randomization is performed with the sample function in R. For each row, randomly select one biological sample from the previous row and replicate it at a random position in that row (a short replicate). After the first three pooled QC wells, embed a random selection of 5 biological samples drawn from the previous plate at the start of the current plate (batch replicates). The replicate positions can be generated programmatically via the plateDesign function in the hRUV R package or via the hRUV shiny application (http://shiny.maths.usyd.edu.au/hRUV). The rationale is that these systematic replicate arrangements let hRUV estimate intra-batch (short replicates, pooled QC) and inter-batch (batch replicates) variance and progressively merge adjustments between neighbouring batches, and the authors believe systematic arrangements perform better with hRUV than fully random ones.

## Related tools

- **hRUV** (R package providing the plateDesign function to generate the replicate arrangement for a given plate size and number of inter-plate replicates; also the downstream normalization that consumes the design.) — https://github.com/SydneyBioX/hRUV
- **hRUV shiny application** (Interactive web interface (http://shiny.maths.usyd.edu.au/hRUV) for generating the replicate embedding arrangement for large-scale metabolomics batches.) — http://shiny.maths.usyd.edu.au/hRUV
- **R** (Language used to call the plateDesign function and to perform randomization with the sample function during replicate selection.)

## Examples

```
library(hRUV); design <- plateDesign(plate_size = 96, n_inter_plate_replicates = 5)
```

## Evaluation signals

- Verify the design contains the expected replicate classes in the expected positions: the first three wells of each plate are pooled QC samples, a single pooled QC follows every 10 runs, each row contains one short replicate drawn from the previous row, and ~5 batch replicates from the previous plate appear near the start of the plate (after the first three QC wells).
- Check the design is usable downstream: the embedded replicates should be sufficient to support RUV-III-based intra- and inter-batch variance estimation and hierarchical merging across neighbouring batches in hRUV.
- If an even positional spread across the previous plate is required, inspect the distribution of batch replicate positions — the design can by chance draw samples clustered from only one half of the previous plate (observed between batches 6 and 7, and 13 and 14), which limits capture of unwanted variability across the whole plate.
- Cross-check a programmatic plateDesign output against the interactive shiny application at http://shiny.maths.usyd.edu.au/hRUV for the same plate size and replicate count to confirm consistency of the arrangement.
- Confirm the short replicates were drawn using the sample function in R (random selection) rather than a fixed positional pattern, since the article states randomization was performed with sample in R.

## Limitations

- By chance, replicate selection can draw samples clustered from only one half of the previous plate, limiting the ability to capture unwanted variability across the whole plate (observed between batches 6 and 7, and 13 and 14).
- The design's utility depends on the downstream normalization being hRUV/RUV-III, which expects data without missing values; large-scale metabolomics data usually require an imputation workaround.
- Selection of appropriate negative-control metabolites for batch correction remains a challenge because metabolite signals depend on individual chemical properties.

## Evidence

- [methods] Pooled QC layout on each 96-well plate (3 at start, then one every 10 runs): "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [methods] Short replicates drawn from previous row: "we randomly selected a single biological sample from the previous row to be replicated at random position in a row (short replicate)"
- [methods] Batch replicates embedded per plate: "after the first three pooled QC samples, a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [discussion] Randomization performed with R sample function: "All randomization was performed using the sample function in R."
- [discussion] hRUV replicate design function and shiny app: [evidence span withheld — anchor status: not_found]
- [discussion] Systematic arrangements believed better than fully random: "we believe that systematic arrangements perform better with hRUV normalization than fully random ones"
- [discussion] Limitation: half-plate replicate clustering: "it is possible by chance to select samples whose positions are from only the first half or only the second half of the previous plate"
- [abstract] Purpose of replicate embedding for variance estimation: "an arrangement to embed biological sample replicates to measure the variance within and between batches"
- [abstract] hierarchical merging of batch adjustments: "remove unwanted variation in a hierarchical structure (hRUV) by progressively merging the adjustments in neighbouring batches"
- [readme] README overview of hRUV replicate types: "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III."
- [readme] README software requirements for reproducibility: [evidence span withheld — anchor status: not_found]
