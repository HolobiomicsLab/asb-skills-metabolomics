---
name: gene-metabolite-pearson-correlation
description: Use when integrating spatial transcriptome and MALDI-MSI data at spot
  resolution and you need to select gene–metabolite feature pairs whose Pearson correlation
  on anchor spot pairs will drive CCA-based feature enrichment before ICP alignment.
when_to_use_negative:
- Input data have already been aligned and a shared fcca variate exists — rerunning
  Pearson correlation on aligned features would double-count information already captured
  by the high-correlated variables
- The spatial transcriptome and MALDI-MSI data are from non-adjacent tissue sections
  with no plausible co-localization, so anchor spot pairs cannot be meaningfully identified
- The task is general-purpose correlation analysis between two unrelated feature matrices
  (e.g. bulk omics or image features) with no spatial anchor-pair or Simpson-diversity
  context
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3520
tools:
- name: haCCA
  role: Provides the icp_3d_alignment and direct_alignment routines and the Data object
    structure in which the anchor pairs, Pearson correlations, and fcca variate are
    computed and stored
  repo: github.com/LittleLittleCloud/haCCA
- name: Python
  role: Language in which the haCCA package is published and the correlation/CCA pipeline
    is executed (pip install hacca)
- name: Scanpy
  role: Used for standard preprocessing of spatial data and reading .h5ad files before
    Data objects are constructed (sc.read_h5ad)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:7de6b1200e96520506e78a9bf93be180608f81a7b9b8e3574b6bfbc063284376
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-metabolite-pearson-correlation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In the haCCA workflow, this skill identifies high-correlated gene–metabolite feature pairs by computing Pearson correlations between gene features from a spatial transcriptome Data object (a) and metabolite m/z features from an ICP-aligned MALDI-MSI Data object (b_prime) on the anchor spot pairs retained by spatial-distance and Simpson-diversity (<0.2) filters.

## When to use

Use when integrating spatial transcriptome and MALDI-MSI data at spot resolution and you need to select gene–metabolite feature pairs whose Pearson correlation on anchor spot pairs will drive CCA-based feature enrichment before ICP alignment.

## When NOT to use

- Input data have already been aligned and a shared fcca variate exists — rerunning Pearson correlation on aligned features would double-count information already captured by the high-correlated variables
- The spatial transcriptome and MALDI-MSI data are from non-adjacent tissue sections with no plausible co-localization, so anchor spot pairs cannot be meaningfully identified
- The task is general-purpose correlation analysis between two unrelated feature matrices (e.g. bulk omics or image features) with no spatial anchor-pair or Simpson-diversity context

## Inputs

- Spatial transcriptome Data object a (X feature matrix, D spatial matrix, optional Label), loaded from an .h5ad file (e.g. a_h5ad = sc.read_h5ad('/path/to/a.h5ad'))
- MALDI-MSI Data object b_prime (X m/z feature matrix, D spatial matrix, optional Label), loaded from an .h5ad file (e.g. b_prime_h5ad = sc.read_h5ad('/path/to/b_prime.h5ad'))
- Anchor spot-pairs table with fields for spot pair identifiers, distance, Simpson diversity (<0.2), and Pearson correlation values

## Outputs

- High-correlated gene–metabolite feature pairs table (with Pearson correlation values)
- fcca feature vectors (the new high-correlated variate) appended as an extra feature dimension to the original feature matrices
- Updated Data objects _a and _b_prime carrying the fcca feature dimension

## How to apply

After gross (manual) and ICP-3D alignment have produced _a and _b_prime Data objects — each a triplet of (X: np.ndarray feature matrix, D: np.ndarray spatial matrix, optional Label cluster array) — filter candidate spot pairs to anchor pairs using the spatial-distance criterion and the Simpson-diversity threshold of <0.2 on neighbors. For the retained anchor spot pairs, compute Pearson correlation between gene features in _a and metabolite m/z features in _b_prime to derive high-correlated gene–metabolite pairs. The rationale is that these high-correlated pairs, though derived from non-shared spots, carry enough co-localization signal to be converted into new high-correlated variables via CCA, yielding the fcca variate that is appended as an extra feature dimension to both datasets so that the subsequent ICP integration uses morphology plus this variate for improved accuracy.

## Related tools

- **haCCA** (Provides the icp_3d_alignment and direct_alignment routines and the Data object structure in which the anchor pairs, Pearson correlations, and fcca variate are computed and stored) — github.com/LittleLittleCloud/haCCA
- **Python** (Language in which the haCCA package is published and the correlation/CCA pipeline is executed (pip install hacca))
- **Scanpy** (Used for standard preprocessing of spatial data and reading .h5ad files before Data objects are constructed (sc.read_h5ad))

## Examples

```
from hacca import *; _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); # then compute Pearson correlations on anchor pairs and run hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- The anchor-pairs table exists and contains the expected fields: spot pair identifiers, distance, Simpson diversity <0.2, and Pearson correlation values
- The fcca feature vectors exist with exactly one extra feature dimension added to the original feature matrices of both _a and _b_prime
- Only features from anchor spot pairs passing the Simpson-diversity <0.2 filter contribute to the high-correlated pair list (features from excluded pairs must not appear)
- Pearson correlations of the retained pairs are of sufficient magnitude to be treated as 'high correlated' — pairs with weak correlations should not survive to the CCA step
- Downstream label transfer accuracy/ARI on pseudo-paired MSI-MALDI data improves when the fcca variate is included, consistent with the reported 6–20% gain over STUtility-style morphological alignment

## Limitations

- haCCA has only been evaluated on spatial transcriptome and MALDI-MSI data; integrating image-based sequencing, spatial proteomics, or histological images is not yet supported
- Behavior of haCCA on higher-resolution data has not been evaluated
- The Simpson-diversity threshold (<0.2) and the anchor-pair spatial-distance criterion are workflow-specific cutoffs not validated outside the mouse brain and ICC-model datasets
- The haCCA repository provides no changelog and no stated software version in the pip install instructions (pip install hacca)

## Evidence

- [methods] Pearson correlation on anchor spot pairs to select high-correlated gene-metabolite pairs: [evidence span withheld — anchor status: not_found]
- [methods] Simpson diversity threshold: [evidence span withheld — anchor status: not_found]
- [methods] anchor spot pair identification: [evidence span withheld — anchor status: not_found]
