---
name: mouse-brain-spatial-data-processing
description: Use when integrating 10X Visium spatial transcriptome (.h5ad) with MALDI-MSI
  metabolome data from mouse-brain (or ICC tumor) sections that share no spots or
  features, and spot-to-spot cluster-label transfer accuracy must be quantified.
when_to_use_negative:
- Input modalities are image-based sequencing, spatial proteomics, or histological
  images — haCCA has only been applied to spatial transcriptome and spatial metabolome
  data
- The two datasets already share spots or features, so direct spot-to-spot mapping
  is possible without alignment-based integration
- Higher-resolution spatial data is involved — the authors state haCCA's behavior
  on higher resolution data has not been evaluated
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: haCCA
  role: Core workflow providing manual_gross_alignment, icp_3d_alignment, and direct_alignment
    for integrating the two spatial modalities
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Standard processing and reading of the .h5ad spatial transcriptome and MALDI-MSI
    files before Data object construction
- name: ICP
  role: Iterative closest point algorithm used for icp_3d_alignment of morphological
    outlines between modalities
- name: STUtility
  role: Morphological-alignment benchmark whose cluster labels serve as ground truth
    for label-transfer accuracy evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:45ffece4ec3f3c6b2c6cd9a18a1f6489b4893a98238673e6304f86019520e1c4
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
  - Python
  - STUtility
  - Scanpy
  - haCCA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mouse-brain-spatial-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A haCCA-based workflow for processing and integrating paired mouse-brain 10X Visium spatial transcriptome and MALDI-MSI metabolome sections by combining modified morphological alignment (manual gross alignment + ICP 3D) with high-correlated gene-metabolite feature pairs, enabling spot-to-spot label transfer despite no shared spots or features between modalities.

## When to use

Use when integrating 10X Visium spatial transcriptome (.h5ad) with MALDI-MSI metabolome data from mouse-brain (or ICC tumor) sections that share no spots or features, and spot-to-spot cluster-label transfer accuracy must be quantified.

## When NOT to use

- Input modalities are image-based sequencing, spatial proteomics, or histological images — haCCA has only been applied to spatial transcriptome and spatial metabolome data
- The two datasets already share spots or features, so direct spot-to-spot mapping is possible without alignment-based integration
- Higher-resolution spatial data is involved — the authors state haCCA's behavior on higher resolution data has not been evaluated

## Inputs

- 10X Visium spatial transcriptome file (a.h5ad) with feature matrix and obsm['spatial'] coordinates
- MALDI-MSI metabolome file (b_prime.h5ad) with peak/intensity feature matrix and spatial coordinates
- Optional cluster label array (Label) for supervised evaluation

## Outputs

- b_predict Data object containing aligned features from a and spots from b_prime
- Per-section label transfer accuracy values (e.g. 87.5%, 79.5%, 87.6% across three mouse brains)
- label_transfer_accuracy_mouse_brain.csv comparing haCCA vs STUtility benchmark

## How to apply

Load the paired spatial transcriptome (a.h5ad) and MALDI-MSI (b_prime.h5ad) files with Scanpy, then construct haCCA Data triplets of (X feature matrix, D spatial matrix from obsm['spatial'], optional Label cluster array). First apply manual_gross_alignment of the b_prime section onto the a section for coarse spatial correspondence, then refine coordinates with icp_3d_alignment (ICP iterative closest point on the morphological outlines), and finally run direct_alignment on the ICP-aligned pair to integrate the modalities via high-correlated gene-metabolite feature pairs, producing b_predict that carries features from a on spots from b_prime. For benchmarking, repeat the alignment/integration using STUtility's morphological alignment and treat its cluster labels as ground truth, then compute label transfer accuracy and label transfer ARI per brain section.

## Related tools

- **haCCA** (Core workflow providing manual_gross_alignment, icp_3d_alignment, and direct_alignment for integrating the two spatial modalities) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Standard processing and reading of the .h5ad spatial transcriptome and MALDI-MSI files before Data object construction)
- **ICP** (Iterative closest point algorithm used for icp_3d_alignment of morphological outlines between modalities)
- **STUtility** (Morphological-alignment benchmark whose cluster labels serve as ground truth for label-transfer accuracy evaluation)

## Examples

```
from hacca import *; a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); _b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Computed label transfer accuracies for the three mouse-brain sections match the reported Figure 3 values of 87.5%, 79.5%, and 87.6%
- haCCA accuracies exceed the STUtility morphological-alignment benchmark by 6–20%
- b_predict contains features from the spatial transcriptome (a) and spots from the MALDI-MSI data (b_prime), confirming correct spot-to-spot transfer structure
- Label transfer ARI is reported alongside accuracy for benchmark comparison
- Results are written to label_transfer_accuracy_mouse_brain.csv for per-brain comparison

## Limitations

- haCCA's behavior on higher resolution data has not been evaluated
- Only spatial transcriptome and spatial metabolome data have been used; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported
- No software version specified for haCCA (pip install hacca) or for STUtility used as the benchmark comparator
- No data accession or dataset identifier is given for the spatial transcriptome or MSI-MALDI data used in the pseudo-paired and real-world evaluations
- No changelog found for the haCCA repository

## Evidence
