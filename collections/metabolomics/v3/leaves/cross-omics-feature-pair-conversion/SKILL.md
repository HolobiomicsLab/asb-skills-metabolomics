---
name: cross-omics-feature-pair-conversion
description: Use when integrating two spatial omics modalities (e.g. 10X Visium spatial
  transcriptome and MALDI-MSI metabolome) that share no features, and you need a cross-omics
  correlated feature to guide spot-to-spot alignment.
when_to_use_negative:
- When the spatial transcriptome and MALDI-MSI data already share a common feature
  space (shared genes/metabolites), the cross-omics feature-pair conversion step is
  unnecessary and simpler direct alignment suffices.
- When the goal is purely morphological (image-based) alignment of spatial coordinates
  without leveraging cross-omics co-variation, use the manual_gross_alignment or icp_3d_alignment
  path alone instead.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: haCCA
  role: Python package providing the anchor-pair identification, high-correlated feature
    pair selection, and CCA conversion routines that produce the fcca feature.
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Python
  role: Runtime environment for executing haCCA's cross-omics feature-pair conversion
    and CCA computations.
- name: Scanpy
  role: Standard preprocessing of the spatial transcriptome .h5ad data before constructing
    the Data triplets used in feature-pair identification.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:61e36b2654dabbe2deb365da2885023e6fb09102e34672344e1a3ac603a334b8
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-omics-feature-pair-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify gene–metabolite (m/z) feature pairs with high Pearson correlation across paired spatial transcriptome and MALDI-MSI spot maps, then convert those pairs into a single canonical-correlation variate (fcca) that is appended as an extra feature dimension to both datasets before alignment. This enables haCCA to anchor spot-to-spot integration on cross-omics co-variation rather than shared features alone.

## When to use

Use when integrating two spatial omics modalities (e.g. 10X Visium spatial transcriptome and MALDI-MSI metabolome) that share no features, and you need a cross-omics correlated feature to guide spot-to-spot alignment.

## When NOT to use

- When the spatial transcriptome and MALDI-MSI data already share a common feature space (shared genes/metabolites), the cross-omics feature-pair conversion step is unnecessary and simpler direct alignment suffices.
- When the goal is purely morphological (image-based) alignment of spatial coordinates without leveraging cross-omics co-variation, use the manual_gross_alignment or icp_3d_alignment path alone instead.

## Inputs

- aligned spatial transcriptome Data object (a): X feature matrix of gene expression, D spatial coordinates, optional Label cluster information
- gross/ICP-aligned MALDI-MSI Data object (b_prime): X feature matrix of metabolite m/z intensities, D spatial coordinates, optional Label
- anchor spot pairs table (spot pair identifiers, spatial distance, Simpson diversity <0.2, Pearson correlation values)

## Outputs

- anchor-pairs table with fields: spot pair identifiers, distance, Simpson diversity (<0.2), Pearson correlation values
- fcca feature vectors: the new canonical-correlation variate appended as an extra feature dimension to both original X matrices

## How to apply

Start from the gross/ICP-aligned MALDI-MSI Data object (b_prime) and the aligned spatial transcriptome Data object (a), each a triplet of (X feature matrix, D spatial matrix, optional Label). First identify anchor spot pairs by filtering candidate spot pairs using spatial-distance criteria and a Simpson diversity threshold of <0.2 on neighboring spots. On the retained anchor spot pairs, compute Pearson correlation between gene features in a and metabolite m/z features in b_prime to derive high-correlated gene-metabolite feature pairs. Convert those selected high-correlated feature pairs into new high-correlated variables and run canonical correlation analysis (CCA) to obtain the fcca feature, appending it as an extra feature dimension to both X matrices. The rationale is that these variates carry cross-modal biological signal that the subsequent ICP alignment can exploit in addition to morphological data.

## Related tools

- **haCCA** (Python package providing the anchor-pair identification, high-correlated feature pair selection, and CCA conversion routines that produce the fcca feature.) — https://github.com/LittleLittleCloud/haCCA
- **Python** (Runtime environment for executing haCCA's cross-omics feature-pair conversion and CCA computations.)
- **Scanpy** (Standard preprocessing of the spatial transcriptome .h5ad data before constructing the Data triplets used in feature-pair identification.)

## Examples

```
from hacca import *; _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime)
```

## Evaluation signals

- Verify the anchor-pairs table exists and contains the expected fields: spot pair identifiers, distance, Simpson diversity (<0.2), and Pearson correlation values.
- Confirm the fcca feature vectors exist with one extra feature dimension added to the original feature matrices (dimension check on X).
- Check that the selected gene–metabolite pairs show high Pearson correlation on the anchor spot pairs before CCA conversion.
- Confirm that the subsequent ICP alignment uses both morphological data and the appended fcca variate, not morphology alone.

## Limitations

- No changelog found for the haCCA repository.
- No software version number stated for STUtility used as the benchmark comparator.
- Behavior of haCCA on higher resolution data has not been evaluated.
- No data accession or dataset identifier given for the spatial transcriptome or MSI-MALDI data used in the pseudo-paired and real-world evaluations.

## Evidence

- [methods] Steps 4-5 of haCCA identify anchor spot pairs (close distance, low Simpson diversity <0.2 neighbors): [evidence span withheld — anchor status: not_found]
- [methods] CCA generates new high-correlated variate fcca added as extra feature dimension: [evidence span withheld — anchor status: not_found]
