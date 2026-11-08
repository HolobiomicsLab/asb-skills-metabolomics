---
name: nearest-neighbor-outlier-alignment
description: Use when two adjacent spatial sections (e.g. 10X Visium spatial transcriptome
  and MALDI-MSI) have already undergone manual gross and ICP alignment, but residual
  spot-to-spot distances must be minimized before feature-based integration.
when_to_use_negative:
- When the sections have not yet undergone gross affine and ICP alignment — direct_alignment
  is the third stage of haCCA's modified morphological alignment pipeline, applied
  to (_a, _b_prime)
- When modalities other than spatial transcriptome and spatial metabolome data are
  involved — the article notes only these two have been used so far
tools:
- name: haCCA
  role: Provides the direct_alignment function that computes the nearest-neighbor
    translation matrix minimizing outlier-spot distances
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Reads the a.h5ad and b_prime.h5ad files supplying the spatial coordinates
    and feature matrices
- name: ICP
  role: Upstream icp_3d_alignment step that produces the _b_prime input to direct_alignment
  repo: github.com/LittleLittleCloud/haCCA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:72531d5022e6dc6323e239ba4c52b112a845f875b35463f16fc8ea450aa042a1
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
  - Scanpy
  - haCCA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nearest-neighbor-outlier-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

haCCA's direct_alignment step: after gross affine and ICP-based registration, it further aligns a MALDI-MSI section to a spatial transcriptome section by minimizing the distances between outlier spots lacking a MSI neighbor and their nearest-neighbor spots, yielding spot-to-spot aligned data (b_predict).

## When to use

Use when two adjacent spatial sections (e.g. 10X Visium spatial transcriptome and MALDI-MSI) have already undergone manual gross and ICP alignment, but residual spot-to-spot distances must be minimized before feature-based integration.

## When NOT to use

- When the sections have not yet undergone gross affine and ICP alignment — direct_alignment is the third stage of haCCA's modified morphological alignment pipeline, applied to (_a, _b_prime)
- When modalities other than spatial transcriptome and spatial metabolome data are involved — the article notes only these two have been used so far

## Inputs

- a: haCCA Data triplet (X: np.ndarray densified feature matrix from a_h5ad.X.toarray(), D: np.ndarray spatial coordinates from obsm['spatial'])
- _b_prime: ICP-refined haCCA Data triplet for the MSI section, output of hacca.icp_3d_alignment

## Outputs

- b_predict: haCCA Data object containing aligned features from a and samples from b_prime
- aligned coordinate matrices for _a, _b_prime and b_predict plus recovered translation matrices, persisted to a single npz archive

## How to apply

After loading both h5ad files with Scanpy and constructing haCCA Data triplets (X = feature matrix, D = obsm['spatial']), first run hacca.manual_gross_alignment(a, b_prime) to build the initial affine matrix, then hacca.icp_3d_alignment(a, _b_prime) for iterative closest-point refinement. Then call hacca.direct_alignment(_a, _b_prime), which identifies outlier spots in the spatial transcriptome data that lack a MALDI-MSI neighbor and computes a translation matrix by minimizing their distances to nearest-neighbor spots. The returned b_predict contains aligned features from a and samples from b_prime; persist the aligned coordinates (_a, _b_prime, b_predict) and translation matrices in a single npz archive. This nearest-neighbor-based refinement is the 'further alignment' component that lets haCCA outperform purely morphological alignment (STUtility) by 6-20% on label transfer accuracy and ARI.

## Related tools

- **haCCA** (Provides the direct_alignment function that computes the nearest-neighbor translation matrix minimizing outlier-spot distances) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Reads the a.h5ad and b_prime.h5ad files supplying the spatial coordinates and feature matrices)
- **ICP** (Upstream icp_3d_alignment step that produces the _b_prime input to direct_alignment) — github.com/LittleLittleCloud/haCCA

## Examples

```
b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Confirm the persisted npz archive contains coordinate arrays for both sections (_a, _b_prime, b_predict) and the translation matrices
- b_predict should contain aligned features from a and samples from b_prime
- Benchmark downstream label transfer accuracy and label transfer ARI against STUtility; the paper reports haCCA outperforming by 6-20% on pseudo-paired MSI-MALDI data
- Verify that outlier spots identified by the step now have minimized distances to their nearest-neighbor spots in the MSI section

## Limitations

- Behavior on higher resolution data has not been evaluated
- Integration limited to spatial transcriptome and spatial metabolome data; image-based sequencing, spatial proteomics, and histological images are not yet supported
- No software version specified for haCCA or the STUtility benchmark; no data accession identifiers given for the evaluation datasets

## Evidence
