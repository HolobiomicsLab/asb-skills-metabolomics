---
name: pseudo-maldi-msi-simulation
description: Use when benchmarking a spatial-transcriptome/MALDI-MSI integration method
  and no real paired metabolome data exist — split features of a 10X Visium mouse-brain
  section into two pseudo-sections and perturb coordinates with Gaussian translation
  to mimic neighboring-section morphology.
when_to_use_negative:
- When real paired MALDI-MSI and spatial transcriptome data from the same tissue section
  exist — use actual data rather than simulated pseudo-sections.
- When the integration method under test requires metabolite/peak features (m/z) rather
  than gene features — a pseudo-MSI section built from gene splits contains no metabolomic
  content.
- When evaluating performance on higher-resolution spatial data (the article notes
  this behavior 'should be further explored' and is not assessed by this benchmark).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: haCCA
  role: Benchmark target — the generated pseudo-pairs are fed into haCCA's manual_gross_alignment,
    icp_3d_alignment, and direct_alignment methods to measure label-transfer accuracy
    vs. STUtility morphological alignment
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Reads and processes the 10X Visium spatial transcriptome data into .h5ad AnnData
    objects that supply the feature matrix and spatial coordinates for feature splitting
    and Gaussian translation
- name: 10X Visium
  role: Source platform providing the public mouse-brain spatial transcriptome data
    from which pseudo-section pairs are derived
- name: STUtility
  role: Morphological-alignment baseline comparator; haCCA's 6-20% outperformance
    on pseudo-pairs is measured against it
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:397bc0a4dd0651a061a914d45e9ed1b684a215d83fd2b9b0e18909d5298dfac2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - 10X Visium
  - Python
  - STUtility
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# pseudo-maldi-msi-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates benchmark pairs of pseudo-spatial-transcriptome and pseudo-MALDI-MSI sections from public 10X Visium spatial transcriptome data by randomly splitting gene features and applying Gaussian translation to spatial coordinates, so haCCA's integration accuracy can be evaluated without real paired MSI data.

## When to use

Use when benchmarking a spatial-transcriptome/MALDI-MSI integration method and no real paired metabolome data exist — split features of a 10X Visium mouse-brain section into two pseudo-sections and perturb coordinates with Gaussian translation to mimic neighboring-section morphology.

## When NOT to use

- When real paired MALDI-MSI and spatial transcriptome data from the same tissue section exist — use actual data rather than simulated pseudo-sections.
- When the integration method under test requires metabolite/peak features (m/z) rather than gene features — a pseudo-MSI section built from gene splits contains no metabolomic content.
- When evaluating performance on higher-resolution spatial data (the article notes this behavior 'should be further explored' and is not assessed by this benchmark).

## Inputs

- 10X Visium spatial transcriptome AnnData (.h5ad) with obsm['spatial'] coordinates
- Gene/feature matrix X (e.g. scanpy-processed counts)
- Optional cluster/label array per spot for label-transfer evaluation

## Outputs

- Paired a.h5ad and b_prime.h5ad files (pseudo-spatial-transcriptome section + pseudo-MALDI-MSI section)
- Disjoint feature subsets (gene splits) for each pseudo section
- Gaussian-translated spatial coordinate matrices for the pseudo-MSI section

## How to apply

Start from public 10X Visium spatial transcriptome data (mouse brain cohort, per the haCCA article) processed with Scanpy into .h5ad AnnData objects. Randomly split the gene/feature set into two disjoint subsets: one becomes the pseudo-spatial-transcriptome section (a.h5ad), the other the pseudo-MALDI-MSI section (b_prime.h5ad). Apply Gaussian translation to the spatial coordinates (obsm['spatial']) of one section to mimic the morphological disagreement between neighboring tissue sections. Emit the paired a.h5ad and b_prime.h5ad files — the article's benchmark demonstrates haCCA outperforms STUtility morphological alignment by 6-20% on these pseudo pairs, evaluated by label transfer accuracy and label transfer ARI.

## Related tools

- **haCCA** (Benchmark target — the generated pseudo-pairs are fed into haCCA's manual_gross_alignment, icp_3d_alignment, and direct_alignment methods to measure label-transfer accuracy vs. STUtility morphological alignment) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Reads and processes the 10X Visium spatial transcriptome data into .h5ad AnnData objects that supply the feature matrix and spatial coordinates for feature splitting and Gaussian translation)
- **10X Visium** (Source platform providing the public mouse-brain spatial transcriptome data from which pseudo-section pairs are derived)
- **STUtility** (Morphological-alignment baseline comparator; haCCA's 6-20% outperformance on pseudo-pairs is measured against it)

## Examples

```
a_h5ad = sc.read_h5ad('/path/to/a.h5ad'); b_prime_h5ad = sc.read_h5ad('/path/to/b_prime.h5ad'); a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); b_prime = Data(X=b_prime_h5ad.X.toarray(), D=b_prime_h5ad.obsm['spatial'])
```

## Evaluation signals

- Each pseudo pair must have the same number of spots (spatial coordinates) in a.h5ad and b_prime.h5ad
- The two feature matrices must be disjoint — no gene appears in both a.X and b_prime.X after splitting
- Gaussian-translated coordinates must deviate from the original spatial coordinates by the Gaussian perturbation amount, not be identical
- Label transfer accuracy and label transfer ARI computed on the pseudo pairs must reproduce the reported 6-20% improvement of haCCA over STUtility morphological alignment

## Limitations

- The pseudo-MSI section is derived from gene features, not real metabolite/peak data, so it does not capture true m/z spectral complexity of MALDI-MSI.
- haCCA's behavior on higher-resolution data has not been evaluated, and this benchmark is generated from 10X Visium spot-level data.
- No data accession or dataset identifier is given in the article for the spatial transcriptome data used to generate the pseudo-pairs.

## Evidence

- [methods] Gaussian translation mimics neighboring-section morphology disagreement: [evidence span withheld — anchor status: not_found]
