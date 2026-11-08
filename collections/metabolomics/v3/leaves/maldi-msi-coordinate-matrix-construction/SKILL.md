---
name: maldi-msi-coordinate-matrix-construction
description: Use when two spatial datasets (e.g. 10X Visium/BMKMANU S1000 transcriptome
  and MALDI-MSI) are stored as .h5ad files with feature matrices in .X and spot coordinates
  in obsm['spatial'] and must be packaged for haCCA alignment.
when_to_use_negative:
- The section files do not contain spatial coordinates in obsm['spatial'] or a feature
  matrix in .X (e.g. image-based sequencing or histological-image-only data, which
  haCCA does not yet support)
- The data are already haCCA Data objects or the morphological alignment has already
  been performed and only feature-based integration remains
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: haCCA
  role: Provides the Data triplet format and the alignment functions that consume
    the constructed coordinate matrices
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Reads the .h5ad sections and exposes .X (densified via toarray) and obsm['spatial']
    coordinate matrices
- name: Python
  role: Runtime for the hacca pip package and NumPy array construction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:cf6193136bf69a0285346e23152634f51151112229ab8431c4e8c487173051ef
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  repo_url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - ICP
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# maldi-msi-coordinate-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs haCCA Data triplets (feature matrix X, spatial coordinate matrix D, optional Label) from .h5ad spatial transcriptome and MALDI-MSI sections so the modified morphological alignment module can register the two sections at spot resolution.

## When to use

Use when two spatial datasets (e.g. 10X Visium/BMKMANU S1000 transcriptome and MALDI-MSI) are stored as .h5ad files with feature matrices in .X and spot coordinates in obsm['spatial'] and must be packaged for haCCA alignment.

## When NOT to use

- The section files do not contain spatial coordinates in obsm['spatial'] or a feature matrix in .X (e.g. image-based sequencing or histological-image-only data, which haCCA does not yet support)
- The data are already haCCA Data objects or the morphological alignment has already been performed and only feature-based integration remains

## Inputs

- a.h5ad spatial transcriptome section (e.g. 10X Visium) with .X feature matrix and obsm['spatial'] coordinates
- b_prime.h5ad MALDI-MSI section with m/z feature matrix in .X and spot coordinates in obsm['spatial']
- Optional cluster label array (Label) per section

## Outputs

- haCCA Data triplet (X: np.ndarray densified feature matrix, D: np.ndarray spatial matrix, Label: Optional[np.ndarray]) for each section
- npz archive containing coordinate matrices from _a, _b_prime, b_predict and the recovered affine translation matrices

## How to apply

Read each section with Scanpy via sc.read_h5ad (a.h5ad for the transcriptome, b_prime.h5ad for the MSI data). Densify the sparse feature matrices with a_h5ad.X.toarray() and b_prime_h5ad.X.toarray() because haCCA expects np.ndarray features, and extract spot locations from obsm['spatial'] as D. Build Data objects as the triplet (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray]), where Label is an optional cluster-information array that can be attached when available. These triplets are the required input format for the subsequent manual_gross_alignment, icp_3d_alignment, and direct_alignment calls; persist the resulting coordinate matrices and affine translation matrices into a single npz archive for downstream integration.

## Related tools

- **haCCA** (Provides the Data triplet format and the alignment functions that consume the constructed coordinate matrices) — https://github.com/LittleLittleCloud/haCCA
- **Scanpy** (Reads the .h5ad sections and exposes .X (densified via toarray) and obsm['spatial'] coordinate matrices)
- **Python** (Runtime for the hacca pip package and NumPy array construction)

## Examples

```
from hacca import *; a_h5ad = sc.read_h5ad('/path/to/a.h5ad'); a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial'])
```

## Evaluation signals

- The densified X arrays equal a_h5ad.X.toarray() and b_prime_h5ad.X.toarray() in shape (n_spots x n_features) and are dense np.ndarray, not scipy sparse
- The D matrices have exactly as many rows as there are spots per section and match the obsm['spatial'] coordinates
- The saved npz contains coordinate arrays for both sections plus the affine translation matrices recovered from the alignment steps
- The Data triplets are accepted without error by hacca.manual_gross_alignment / icp_3d_alignment / direct_alignment, yielding b_predict with aligned features from a and samples from b_prime

## Limitations

- No shared spots or features exist between spatial transcriptomics and MALDI-MSI data, so the coordinate construction is only a precursor — alignment quality must be judged by label transfer accuracy and label transfer ARI
- haCCA has only been used on spatial transcriptome and spatial metabolome data; image-based sequencing, spatial proteomics, and histological images are not yet supported
- Behavior on higher-resolution data has not been evaluated; no software version is specified for the hacca package

## Evidence
