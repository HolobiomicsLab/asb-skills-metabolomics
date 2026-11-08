---
name: maldi-msi-spot-alignment
description: Use when integrating paired or pseudo-paired 10X Visium (.h5ad) and MALDI-MSI
  (.h5ad) spatial datasets that share no spots or features and need morphological/feature-based
  spatial alignment before cross-modality label transfer.
when_to_use_negative:
- The datasets already share spots or features between modalities, so no cross-modality
  alignment is needed
- The input modalities are not spatial transcriptome/spatial metabolome data (e.g.
  image-based sequencing, spatial proteomics, or histological images, which haCCA
  does not yet support)
- Higher-resolution spatial data where haCCA's behavior has not been evaluated
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3940
- http://edamontology.org/topic_3173
tools:
- name: haCCA
  role: Python package providing manual_gross_alignment, icp_3d_alignment, and direct_alignment
    functions for spatial and feature-level integration
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Reads the a.h5ad and b_prime.h5ad spatial data files and provides standard
    preprocessing before Data object construction
- name: ICP
  role: Iterative Closest Point algorithm used in icp_3d_alignment to refine spatial
    coordinates by morphological outlines
- name: STUtility
  role: Morphological-alignment benchmark whose cluster labels serve as ground truth
    for evaluating label transfer accuracy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:b5d99fa22ba9d41b45b537c476cb9072cca87bb2558efa32d613da9db2862da5
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
  - ICP
  - MALDI-MSI
  - Python
  - STUtility
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3940
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# maldi-msi-spot-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for spatially aligning MALDI-MSI metabolomics data to 10X Visium spatial transcriptomics data from the same or neighboring tissue section, using manual gross alignment, ICP 3D iterative-closest-point, and high-correlated gene-metabolite feature pairs so that spot-to-spot label transfer between the two modalities becomes possible.

## When to use

Use when integrating paired or pseudo-paired 10X Visium (.h5ad) and MALDI-MSI (.h5ad) spatial datasets that share no spots or features and need morphological/feature-based spatial alignment before cross-modality label transfer.

## When NOT to use

- The datasets already share spots or features between modalities, so no cross-modality alignment is needed
- The input modalities are not spatial transcriptome/spatial metabolome data (e.g. image-based sequencing, spatial proteomics, or histological images, which haCCA does not yet support)
- Higher-resolution spatial data where haCCA's behavior has not been evaluated

## Inputs

- a.h5ad: 10X Visium spatial transcriptome file (feature matrix X, spatial coordinates in obsm['spatial'])
- b_prime.h5ad: MALDI-MSI spatial metabolomics file (metabolite/peak intensity matrix X, pixel spatial coordinates in obsm['spatial'])
- Optional cluster label arrays for either modality

## Outputs

- b_predict: Data object containing features transferred from a and spots from b_prime
- Aligned spatial coordinate matrices for both modalities
- Label transfer accuracy and label transfer ARI metrics (e.g. label_transfer_accuracy_mouse_brain.csv)

## How to apply

Load the spatial transcriptome (a.h5ad) and MALDI-MSI (b_prime.h5ad) files with Scanpy and construct haCCA Data triplets (X feature matrix, D spatial matrix from obsm['spatial'], optional Label cluster array). First run manual_gross_alignment to obtain a coarse correspondence between the two sections, then refine coordinates with icp_3d_alignment so the morphological outlines of the two modalities match. Finally run direct_alignment on the ICP-aligned data, which integrates the modalities via high-correlated gene-metabolite feature pairs and infers b_predict containing features from a and spots from b_prime. Evaluate the result by label transfer accuracy and label transfer ARI against cluster labels from an independent morphological-alignment benchmark (e.g. STUtility), where haCCA should outperform it by 6-20%.

## Related tools

- **haCCA** (Python package providing manual_gross_alignment, icp_3d_alignment, and direct_alignment functions for spatial and feature-level integration) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Reads the a.h5ad and b_prime.h5ad spatial data files and provides standard preprocessing before Data object construction)
- **ICP** (Iterative Closest Point algorithm used in icp_3d_alignment to refine spatial coordinates by morphological outlines)
- **STUtility** (Morphological-alignment benchmark whose cluster labels serve as ground truth for evaluating label transfer accuracy)

## Examples

```
_b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Label transfer accuracies for the three mouse-brain sections match the reported Figure 3 values of 87.5%, 79.5%, and 87.6%
- Computed label transfer accuracy exceeds the STUtility morphological-alignment benchmark by 6-20%
- b_predict contains features from a and spots/spatial samples from b_prime, as documented in the README
- ICP-aligned spatial matrices show morphological outlines of the two modalities correspond before direct_alignment is run
- Label transfer ARI is computed alongside accuracy as a second agreement metric against STUtility-derived ground truth

## Limitations

- haCCA's behavior on higher-resolution spatial data has not been evaluated
- Integration is limited to spatial transcriptome and spatial metabolome data; image-based sequencing, spatial proteomics, and histological images are not yet supported
- No data accession or software version numbers were provided for the pseudo-paired and real-world evaluation datasets or the STUtility benchmark
- Alignment is challenged by having no shared spots or features between the spatial transcriptomics and MALDI-MSI data

## Evidence

- [results] achieving label transfer accuracies of 87.5%, 79.5% and 87.6% across the three mouse brains: [evidence span withheld — anchor status: not_found]
