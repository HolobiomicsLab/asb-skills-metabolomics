---
name: spatial-coordinate-alignment
description: Use when integrating spatial transcriptome and spatial metabolome (MALDI-MSI)
  data from adjacent tissue sections that share no spots or features, and spot-to-spot
  coordinate registration must precede feature-based integration.
when_to_use_negative:
- When the modalities already share spots or features — the paper notes the challenge
  exists because there are 'no shared spots or features'; direct feature matching
  suffices otherwise.
- When the goal is morphological-image-based registration rather than coordinate-based
  alignment of transcriptome/metabolome modalities — haCCA's modified morphological
  alignment uses spatial coordinates, not histology, and is benchmarked against STUtility's
  morphology-only approach.
- When data are single-cell or sub-spot resolution — the paper states behavior on
  higher-resolution data has not been evaluated.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: haCCA
  role: Python package providing manual_gross_alignment, icp_3d_alignment, and direct_alignment
    for the coordinate alignment pipeline
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Reads the .h5ad sections via sc.read_h5ad and supplies the feature matrix
    and obsm['spatial'] coordinates used to build haCCA Data triplets
- name: ICP
  role: Iterative Closest Point algorithm underlying the icp_3d_alignment refinement
    of spot coordinates
- name: STUtility
  role: Morphological-alignment comparator benchmark (manual alignment) against which
    the coordinate alignment is evaluated
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:33b2b69c82caea55ecd0ada9170d6e8b6d773d41f1844d48ae2aea2164bcbb24
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - ICP
  - Python
  - STUtility
  - Scanpy
  - haCCA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# spatial-coordinate-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

haCCA's modified morphological alignment that registers spot coordinates from two neighboring tissue sections (e.g., spatial transcriptome .h5ad vs MALDI-MSI) via a manual gross affine alignment, ICP 3D refinement, and direct outlier-minimizing alignment before feature integration.

## When to use

Use when integrating spatial transcriptome and spatial metabolome (MALDI-MSI) data from adjacent tissue sections that share no spots or features, and spot-to-spot coordinate registration must precede feature-based integration.

## When NOT to use

- When the modalities already share spots or features — the paper notes the challenge exists because there are 'no shared spots or features'; direct feature matching suffices otherwise.
- When the goal is morphological-image-based registration rather than coordinate-based alignment of transcriptome/metabolome modalities — haCCA's modified morphological alignment uses spatial coordinates, not histology, and is benchmarked against STUtility's morphology-only approach.
- When data are single-cell or sub-spot resolution — the paper states behavior on higher-resolution data has not been evaluated.

## Inputs

- a.h5ad — spatial transcriptome section (Scanpy AnnData with .X feature matrix and .obsm['spatial'] coordinates)
- b_prime.h5ad — MALDI-MSI / spatial metabolome section (AnnData with intensity/peak matrix and spatial coordinates)
- Optional Label arrays with cluster information per spot for label-transfer evaluation

## Outputs

- b_predict — haCCA Data object containing features aligned from a onto samples of b_prime
- Transformed coordinate matrices _a and _b_prime from the ICP step
- Affine/translation matrices recovered from gross, ICP, and direct alignment steps (persisted as npz)

## How to apply

Read each section's .h5ad with Scanpy and build haCCA Data triplets where X is the densified feature matrix (a_h5ad.X.toarray()) and D is obsm['spatial']. First run hacca.manual_gross_alignment(a, b_prime) to build a gross affine matrix from manually selected points; this returns a translated b_prime. Then refine registration with hacca.icp_3d_alignment(a, _b_prime), the Iterative Closest Point step that returns the transformed pair (_a, _b_prime). Finally run hacca.direct_alignment(_a, _b_prime), which finds outlier spots lacking a MALDI-MSI neighbor and computes a translation matrix minimizing their distances to nearest neighbor spots, yielding b_predict with aligned features from a and samples from b_prime. Persist the aligned coordinate matrices and the recovered translation matrices (e.g., a single npz archive) so they can feed downstream feature-pair integration.

## Related tools

- **haCCA** (Python package providing manual_gross_alignment, icp_3d_alignment, and direct_alignment for the coordinate alignment pipeline) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Reads the .h5ad sections via sc.read_h5ad and supplies the feature matrix and obsm['spatial'] coordinates used to build haCCA Data triplets)
- **ICP** (Iterative Closest Point algorithm underlying the icp_3d_alignment refinement of spot coordinates)
- **STUtility** (Morphological-alignment comparator benchmark (manual alignment) against which the coordinate alignment is evaluated)

## Examples

```
from hacca import *; _b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- b_predict contains aligned features from a and samples from b_prime — check the feature/sample provenance of the returned object
- The persisted npz must contain coordinate arrays for both sections plus the affine translation matrices
- Label transfer accuracy (AC) and label transfer ARI on pseudo-paired data should exceed STUtility morphological alignment by 6-20%
- The modified morphological alignment alone should outperform STUtility, and combining it with feature-pair alignment should further increase accuracy
- Manual gross alignment must precede ICP and direct alignment — verify the sequential call order manual_gross_alignment → icp_3d_alignment → direct_alignment

## Limitations

- haCCA behavior on higher-resolution spatial data has not been evaluated
- Only spatial transcriptome and spatial metabolome (MALDI-MSI) modalities have been used; image-based sequencing, spatial proteomics, and histological images are not yet supported
- Integration is challenged because there are no shared spots or features between the two modalities
- No software version number stated for STUtility used as the benchmark comparator, and no data accession for the pseudo-paired evaluation data

## Evidence
