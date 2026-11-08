---
name: spatial-transcriptome-peak-integration
description: Use when you hold paired spatial transcriptome and MALDI-MSI data from
  the same or neighboring tissue section (no shared spots or features) and need spot-to-spot
  integration to map gene expression onto biomolecule profiles.
when_to_use_negative:
- Input data already share spots or features (the challenge haCCA targets is no shared
  spots or features between modalities).
- Data are not spatially referenced (no spatial coordinate matrix) — morphological
  alignment and ICP 3D cannot be applied.
- Data come from higher resolution than spot-level spatial transcriptomics; the article
  notes haCCA's behavior on higher-resolution data has not been evaluated.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: haCCA
  role: Python package providing manual_gross_alignment, icp_3d_alignment and direct_alignment
    to integrate spatial transcriptome and MALDI-MSI data
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: standard processing and reading of the .h5ad inputs before building haCCA
    Data objects
- name: ICP
  role: iterative closest point algorithm used for 3D morphological alignment of spatial
    coordinates
- name: STUtility
  role: morphological-alignment benchmark used as the ground-truth comparator for
    label transfer accuracy
- name: 10X Visium
  role: spatial transcriptome platform supplying the gene-expression modality
- name: MALDI-MSI
  role: matrix-assisted laser desorption ionization mass spectrometry imaging platform
    supplying the metabolite/biomolecule modality
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:bbe86950dc1bda80c9090cb4ed7a16accc44b1c5a7c19ff0ba78703c46a963d1
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
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 21
---

# spatial-transcriptome-peak-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrate spatial transcriptome (e.g. 10X Visium .h5ad) and MALDI-MSI peak/intensity matrices at spot resolution by first morphologically aligning the two spatial coordinate systems (manual gross alignment + ICP 3D), then fusing features through high-correlated gene-metabolite pairs via haCCA's direct_alignment.

## When to use

Use when you hold paired spatial transcriptome and MALDI-MSI data from the same or neighboring tissue section (no shared spots or features) and need spot-to-spot integration to map gene expression onto biomolecule profiles.

## When NOT to use

- Input data already share spots or features (the challenge haCCA targets is no shared spots or features between modalities).
- Data are not spatially referenced (no spatial coordinate matrix) — morphological alignment and ICP 3D cannot be applied.
- Data come from higher resolution than spot-level spatial transcriptomics; the article notes haCCA's behavior on higher-resolution data has not been evaluated.

## Inputs

- .h5ad spatial transcriptome file (a.h5ad) with X expression matrix and obsm['spatial'] coordinates
- .h5ad MALDI-MSI file (b_prime.h5ad) with peak/intensity matrix and obsm['spatial'] coordinates
- optional cluster Label array per spot

## Outputs

- b_predict — integrated Data object carrying aligned gene-expression features from the transcriptome and samples (spots) from the MALDI-MSI section
- label_transfer_accuracy_mouse_brain.csv (per-brain accuracy values)
- spot-to-spot gene expression and biomolecule profiling on the MSI tissue

## How to apply

Load each modality with Scanpy from its .h5ad file and build haCCA Data objects from (X feature matrix, D spatial matrix from obsm['spatial'], optional cluster Label). First run hacca.manual_gross_alignment(a, b_prime) for a coarse correspondence, then refine coordinates with hacca.icp_3d_alignment using the ICP iterative-closest-point algorithm on the morphological outlines. Finally call hacca.direct_alignment(_a, _b_prime), which identifies high-correlated gene-metabolite feature pairs, converts them into new high-correlated variables, and infers b_predict containing aligned features from a and samples from b_prime. The rationale is that morphology-only alignment (e.g. STUtility) under-performs, so haCCA adds the correlated-pair feature dimension to gain 6–20% label transfer accuracy. Validate by comparing per-brain label transfer accuracy against STUtility ground-truth labels (reported 87.5%, 79.5%, 87.6% on three mouse-brain sections).

## Related tools

- **haCCA** (Python package providing manual_gross_alignment, icp_3d_alignment and direct_alignment to integrate spatial transcriptome and MALDI-MSI data) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (standard processing and reading of the .h5ad inputs before building haCCA Data objects)
- **ICP** (iterative closest point algorithm used for 3D morphological alignment of spatial coordinates)
- **STUtility** (morphological-alignment benchmark used as the ground-truth comparator for label transfer accuracy)
- **10X Visium** (spatial transcriptome platform supplying the gene-expression modality)
- **MALDI-MSI** (matrix-assisted laser desorption ionization mass spectrometry imaging platform supplying the metabolite/biomolecule modality)

## Examples

```
from hacca import *; _b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Label transfer accuracy against STUtility-derived ground-truth cluster labels should reach 87.5%, 79.5% and 87.6% for the three mouse-brain sections.
- haCCA accuracy should exceed the STUtility morphological-alignment benchmark by 6–20%.
- b_predict must contain features from a (transcriptome) and samples from b_prime (MSI) — verify shape consistency of the aligned matrices.
- Label transfer ARI should additionally be computed and compared against the benchmark.
- Check that both Data objects were constructed with X from .X.toarray() and D from obsm['spatial'] before running alignment.

## Limitations

- Only spatial transcriptome and spatial metabolome data have been used; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported.
- How haCCA behaves on higher-resolution data has not been explored.
- No software version number is stated for haCCA or STUtility, and no data accession is given for the pseudo-paired and real-world evaluation datasets.
- No changelog is available in the haCCA repository.

## Evidence
