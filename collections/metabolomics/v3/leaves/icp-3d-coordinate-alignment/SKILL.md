---
name: icp-3d-coordinate-alignment
description: Use when two spatially resolved datasets (e.g., a .h5ad Visium section
  and a MALDI-MSI section) lack shared spots/features and their spatial matrices must
  be geometrically co-registered prior to spot-to-spot integration.
when_to_use_negative:
- The two datasets already share spots or features, so no geometric co-registration
  is needed before integration.
- Only gross/coarse correspondence is sufficient — manual_gross_alignment alone may
  be the appropriate step, and skipping to direct_alignment on unrefined coordinates
  will degrade label transfer accuracy.
- Data are not spatial (no D spatial matrix / obsm['spatial'] coordinates), since
  ICP operates on point clouds of spatial coordinates.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_0092
tools:
- name: haCCA
  role: Python package providing manual_gross_alignment and icp_3d_alignment functions
    for coordinate co-registration of spatial transcriptome and MALDI-MSI Data objects
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Used to read a.h5ad / b_prime.h5ad files and extract the feature matrix and
    obsm['spatial'] coordinates used to construct haCCA Data objects
- name: ICP
  role: The iterative closest point algorithm that refines spatial coordinates by
    matching morphological outlines between the two modalities
- name: STUtility
  role: Morphological-alignment benchmark against which the ICP-based alignment is
    compared (label transfer accuracy)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:916603f9ef45c020283e20c331076d070c4074062e34ad8954d8df63c66ceaf0
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# icp-3d-coordinate-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Iterative Closest Point (ICP) 3D alignment of spatial coordinates from paired spatial-omics modalities (e.g., 10X Visium spots and MALDI-MSI pixels) so that morphological outlines of two sections match before feature-pair integration in haCCA.

## When to use

Use when two spatially resolved datasets (e.g., a .h5ad Visium section and a MALDI-MSI section) lack shared spots/features and their spatial matrices must be geometrically co-registered prior to spot-to-spot integration.

## When NOT to use

- The two datasets already share spots or features, so no geometric co-registration is needed before integration.
- Only gross/coarse correspondence is sufficient — manual_gross_alignment alone may be the appropriate step, and skipping to direct_alignment on unrefined coordinates will degrade label transfer accuracy.
- Data are not spatial (no D spatial matrix / obsm['spatial'] coordinates), since ICP operates on point clouds of spatial coordinates.

## Inputs

- a: haCCA Data object (X feature matrix, D spatial matrix, optional Label) built from a.h5ad (e.g., 10X Visium spatial transcriptome)
- b_prime: haCCA Data object built from b_prime.h5ad (e.g., MALDI-MSI peak/intensity matrix with obsm['spatial'] coordinates)
- coarse-aligned b_prime output from hacca.manual_gross_alignment

## Outputs

- _a: a copy of the first Data object with refined spatial coordinates
- _b_prime: the second Data object with ICP-refined spatial coordinates, ready for hacca.direct_alignment

## How to apply

First run hacca.manual_gross_alignment(a, b_prime) to obtain a coarse correspondence between the spatial matrices (D arrays) of the two Data objects. Then apply hacca.icp_3d_alignment(a, _b_prime), which iteratively minimizes the distance between the point clouds of spot/pixel coordinates to align the morphological outlines of the two modalities. The function returns refined copies of both inputs (_a, _b_prime) with updated spatial coordinates. Rationale: alignment using morphological data and high-correlated variates between MALDI-MSI and spatial transcriptome is carried out by the ICP algorithm, and the modified morphological alignment alone outperforms STUtility-style benchmarks, with alignment quality directly determining downstream direct_alignment label-transfer accuracy.

## Related tools

- **haCCA** (Python package providing manual_gross_alignment and icp_3d_alignment functions for coordinate co-registration of spatial transcriptome and MALDI-MSI Data objects) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Used to read a.h5ad / b_prime.h5ad files and extract the feature matrix and obsm['spatial'] coordinates used to construct haCCA Data objects)
- **ICP** (The iterative closest point algorithm that refines spatial coordinates by matching morphological outlines between the two modalities)
- **STUtility** (Morphological-alignment benchmark against which the ICP-based alignment is compared (label transfer accuracy))

## Examples

```
from hacca import *; _b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- The icp_3d_alignment call returns two Data objects (_a, _b_prime) whose D spatial matrices have been updated relative to the inputs.
- Downstream direct_alignment on the ICP-aligned pair should yield label transfer accuracies exceeding the STUtility morphological-alignment benchmark by 6-20%.
- On the mouse-brain benchmark (STUtility labels as ground truth), per-brain label transfer accuracies should reproduce the reported 87.5%, 79.5%, and 87.6% values.
- Visually, the morphological outlines of the two point clouds should be superposed after alignment; gross misregistration indicates the manual_gross_alignment step was skipped or misapplied.
- Label transfer ARI computed on the aligned pair should be higher than that of the morphology-only STUtility comparison.

## Limitations

- No software version is specified for hacca ('pip install hacca' with no version pin), so reproducibility of alignment behavior across versions is unverified.
- The modified morphological alignment of haCCA itself can outperform STUtility, but the reported gains are established on pseudo-paired MSI-MALDI mouse-brain data with no dataset accession provided.
- haCCA's evaluation is limited to spatial transcriptome and spatial metabolome data; behavior on higher-resolution data has not been evaluated.
- Alignment quality depends on a reasonable initial gross alignment; ICP is applied after manual_gross_alignment in the documented workflow.

## Evidence
