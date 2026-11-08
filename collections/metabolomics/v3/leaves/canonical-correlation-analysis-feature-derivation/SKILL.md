---
name: canonical-correlation-analysis-feature-derivation
description: Use when integrating spatial transcriptome (a.h5ad) and MALDI-MSI (b_prime.h5ad)
  data that share no spots or features, after gross/ICP alignment and anchor spot-pair
  selection (Simpson diversity < 0.2), to create a CCA-derived variate that improves
  alignment accuracy.
when_to_use_negative:
- When spatial transcriptome and MALDI-MSI data already share spots or features —
  no CCA variate is needed
- When the data have not yet been gross/ICP aligned, since anchor spot pairs and Pearson
  correlations are computed on aligned coordinates
- When the goal is higher-resolution integration than spot level, since the article
  notes haCCA behavior on higher-resolution data is not evaluated
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: haCCA
  role: Provides the CCA feature-derivation step that turns high-correlated gene-metabolite
    pairs into the fcca variate appended to both data objects
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Python
  role: Language used to implement the haCCA pipeline and to run Scanpy and haCCA
    functions
- name: Scanpy
  role: Reads and processes the spatial transcriptome and MALDI-MSI .h5ad files whose
    feature matrices feed the Pearson correlation and CCA computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:55c29d7da68b13c8a49ef6d8c317a77ab4dab7c26866c05e5290ea2571a80972
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
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# canonical-correlation-analysis-feature-derivation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derive a shared high-correlated variate (fcca) from gene-metabolite feature pairs across anchor spots and append it as an extra feature dimension to both the spatial transcriptome and MALDI-MSI data before spot-to-spot alignment in haCCA.

## When to use

Use when integrating spatial transcriptome (a.h5ad) and MALDI-MSI (b_prime.h5ad) data that share no spots or features, after gross/ICP alignment and anchor spot-pair selection (Simpson diversity < 0.2), to create a CCA-derived variate that improves alignment accuracy.

## When NOT to use

- When spatial transcriptome and MALDI-MSI data already share spots or features — no CCA variate is needed
- When the data have not yet been gross/ICP aligned, since anchor spot pairs and Pearson correlations are computed on aligned coordinates
- When the goal is higher-resolution integration than spot level, since the article notes haCCA behavior on higher-resolution data is not evaluated

## Inputs

- Spatial transcriptome Scanpy AnnData (.h5ad) — converted into a Data triplet (X feature matrix, D spatial matrix, optional Label cluster information)
- MALDI-MSI Scanpy AnnData (.h5ad, b_prime) — converted into a Data triplet after manual_gross_alignment and icp_3d_alignment
- Anchor spot-pair table (spot pair identifiers, distance, Simpson diversity <0.2, Pearson correlation values)
- Gene features and metabolite m/z feature matrices from the paired anchor spots

## Outputs

- fcca feature vectors — one extra high-correlated variate appended to both the spatial transcriptome and MALDI-MSI feature matrices
- Updated Data objects (a and b_prime) carrying the extra fcca feature dimension for downstream direct_alignment
- Anchor-pairs table with expected fields (spot pair identifiers, distance, Simpson diversity, Pearson correlation values)

## How to apply

Load the gross-aligned spatial transcriptome Data object (a) and the ICP-aligned MALDI-MSI Data object (b_prime) as (X, D, Label) triplets, where X is the feature matrix and D is the spatial matrix from the .h5ad files. Identify anchor spot pairs by filtering candidate spot pairs using spatial distance and the Simpson-diversity threshold (<0.2 on neighbors). For the retained anchor spot pairs, compute Pearson correlations between gene features in a and metabolite m/z features in b_prime to select high-correlated gene-metabolite pairs, then run canonical correlation analysis (CCA) on these pairs to obtain a new high-correlated variate fcca. Append fcca as an individual extra feature dimension to both datasets before the ICP algorithm performs the final alignment using morphological data plus this variate.

## Related tools

- **haCCA** (Provides the CCA feature-derivation step that turns high-correlated gene-metabolite pairs into the fcca variate appended to both data objects) — https://github.com/LittleLittleCloud/haCCA
- **Python** (Language used to implement the haCCA pipeline and to run Scanpy and haCCA functions)
- **Scanpy** (Reads and processes the spatial transcriptome and MALDI-MSI .h5ad files whose feature matrices feed the Pearson correlation and CCA computation)

## Examples

```
pip install hacca; from hacca import *; a_h5ad = sc.read_h5ad('/path/to/a.h5ad'); b_prime_h5ad = sc.read_h5ad('/path/to/b_prime.h5ad')
```

## Evaluation signals

- The anchor-pairs table exists and contains spot pair identifiers, spatial distance, Simpson diversity values all below 0.2, and Pearson correlation values
- The fcca feature vectors exist with exactly one extra feature dimension added to the original feature matrices of both a and b_prime
- Downstream icp_3d_alignment and direct_alignment runs succeed using the morphological data plus the appended fcca variate
- The Pearson-correlation step on anchor spot pairs yields identifiable high-correlated gene-metabolite pairs; no anchor pairs or no correlated pairs indicate a failure of the filtering/CCA step

## Limitations

- haCCA has not been evaluated on higher-resolution (single-cell / sub-spot) data
- Integration is limited to spatial transcriptome and spatial metabolome data; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported
- No software version is specified for haCCA in the README, and STUtility used as benchmark comparator lacks a stated version
- The article does not provide dataset accession identifiers for the mouse brain or ICC datasets used in the pseudo-paired and real-world evaluations

## Evidence

- [methods] Anchor spot-pair identification: [evidence span withheld — anchor status: not_found]
- [methods] Pearson correlation on paired features: [evidence span withheld — anchor status: not_found]
- [methods] CCA to fcca feature: [evidence span withheld — anchor status: not_found]
- [intro] Alignment use of fcca: "Alignment using morphological data and high-correlated variate between MALDI-MSI and spatial transcriptome is carried out by ICP(lterative Closest Point) algorithm"
- [abstract] No shared features rationale: "integrating them with high accuracy is challenge due to no shared spots or features"
