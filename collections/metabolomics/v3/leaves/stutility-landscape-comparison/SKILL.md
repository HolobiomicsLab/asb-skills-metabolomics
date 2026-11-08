---
name: stutility-landscape-comparison
description: Use when benchmarking a spatial multi-omics integration workflow (e.g.,
  spatial transcriptome + MALDI-MSI) against a morphology-only alignment baseline
  such as STUtility on same-section or neighboring-section data.
when_to_use_negative:
- When no morphological/histological image information is available for the sections,
  since STUtility's alignment relies on morphology
- When the goal is integration itself rather than benchmarking — use haCCA's direct_alignment
  on high-correlated gene-metabolite feature pairs instead
- When data are higher-resolution than spot-level spatial transcriptomics; haCCA behavior
  on higher resolution data has not been evaluated
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: STUtility
  role: Morphological-alignment benchmark method whose spatial-coordinate alignment
    outcome serves as the comparator (and ground-truth label source) for evaluating
    haCCA label transfer accuracy/ARI
- name: haCCA
  role: Integration workflow under test, combining manual_gross_alignment, icp_3d_alignment,
    and direct_alignment via high-correlated gene-metabolite feature pairs
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Standard processing and reading of the a.h5ad / b_prime.h5ad spatial data
    files before constructing haCCA Data objects
- name: ICP
  role: Iterative Closest Point algorithm (icp_3d_alignment) refining spatial coordinates
    after manual gross alignment, part of the haCCA alignment being benchmarked
- name: 10X Visium
  role: Source platform of the spatial transcriptome data in the mouse-brain benchmark
    cohort
- name: MALDI-MSI
  role: Source platform of the metabolome (m/z spectra) data in the mouse-brain benchmark
    cohort
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:46b9b36932e8a5e832d30403642e8222be85bd240de32adb2f522454fc1b8e1d
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# stutility-landscape-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses STUtility's morphological-image-based alignment of spatial coordinates as a benchmark landscape-alignment method against which haCCA's high-correlated-feature-pair integration of 10X Visium spatial transcriptomes and MALDI-MSI metabolomes is compared via label transfer accuracy and ARI.

## When to use

Use when benchmarking a spatial multi-omics integration workflow (e.g., spatial transcriptome + MALDI-MSI) against a morphology-only alignment baseline such as STUtility on same-section or neighboring-section data.

## When NOT to use

- When no morphological/histological image information is available for the sections, since STUtility's alignment relies on morphology
- When the goal is integration itself rather than benchmarking — use haCCA's direct_alignment on high-correlated gene-metabolite feature pairs instead
- When data are higher-resolution than spot-level spatial transcriptomics; haCCA behavior on higher resolution data has not been evaluated

## Inputs

- Paired spatial transcriptome file (a.h5ad, 10X Visium, read with Scanpy)
- Paired MALDI-MSI file (b_prime.h5ad, m/z peak/intensity matrix plus obsm['spatial'] coordinates)
- Optional cluster-label array (Label) carried on the Data triplet (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray])

## Outputs

- STUtility morphological-alignment benchmark prediction (b_predict analog containing features from a and spots from b_prime)
- Per-brain label transfer accuracy values comparable to the reported 87.5%, 79.5%, 87.6% (e.g., label_transfer_accuracy_mouse_brain.csv)
- Label transfer ARI values for haCCA vs STUtility comparison

## How to apply

On the same paired inputs (a.h5ad spatial transcriptome, b_prime.h5ad MALDI-MSI with feature matrix X and spatial matrix D from obsm['spatial']), run the STUtility morphological-alignment pipeline as the benchmark instead of (or alongside) the haCCA chain of manual_gross_alignment, icp_3d_alignment, and direct_alignment. STUtility aligns the spatial coordinates of the two modalities using morphological/histological image information, whereas haCCA additionally identifies high-correlated gene-metabolite feature pairs and converts them into new high-correlated variables for integration. After producing the STUtility-aligned prediction, evaluate label transfer accuracy and label transfer ARI against the ground-truth cluster labels defined by the STUtility morphological-alignment outcome. In the mouse-brain evaluation, haCCA exceeded the STUtility benchmark by 6-20%, with reported haCCA accuracies of 87.5%, 79.5% and 87.6% across the three mouse-brain sections (Fig 3 A,C,E). Record per-brain accuracies (e.g., to label_transfer_accuracy_mouse_brain.csv) so the comparison against reported values is reproducible.

## Related tools

- **STUtility** (Morphological-alignment benchmark method whose spatial-coordinate alignment outcome serves as the comparator (and ground-truth label source) for evaluating haCCA label transfer accuracy/ARI)
- **haCCA** (Integration workflow under test, combining manual_gross_alignment, icp_3d_alignment, and direct_alignment via high-correlated gene-metabolite feature pairs) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Standard processing and reading of the a.h5ad / b_prime.h5ad spatial data files before constructing haCCA Data objects)
- **ICP** (Iterative Closest Point algorithm (icp_3d_alignment) refining spatial coordinates after manual gross alignment, part of the haCCA alignment being benchmarked)
- **10X Visium** (Source platform of the spatial transcriptome data in the mouse-brain benchmark cohort)
- **MALDI-MSI** (Source platform of the metabolome (m/z spectra) data in the mouse-brain benchmark cohort)

## Examples

```
from hacca import *; _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Computed label transfer accuracies for the three mouse-brain sections should match the reported Figure 3 values of 87.5%, 79.5%, and 87.6%
- haCCA accuracy should exceed the STUtility morphological-alignment benchmark by 6-20%
- Label transfer ARI should also be reported to corroborate the accuracy comparison
- b_predict must contain features from modality a and spots from b_prime, confirming correct orientation of the label transfer
- Same inputs (a.h5ad, b_prime.h5ad) must be used for both haCCA and the STUtility benchmark so the comparison is paired

## Limitations

- No software version number stated for STUtility used as the benchmark comparator
- haCCA's integration is limited to spatial transcriptome and spatial metabolome data; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported
- Behavior of haCCA on higher resolution data has not been evaluated

## Evidence

- [other] Ground truth definition: [evidence span withheld — anchor status: not_found]
- [other] Accuracy values: [evidence span withheld — anchor status: not_found]
