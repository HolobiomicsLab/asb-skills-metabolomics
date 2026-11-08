---
name: spatial-anchor-spot-pair-identification
description: Use when integrating spatial transcriptome (e.g. 10X Visium .h5ad) and
  MALDI-MSI data that share no spots or features, and you need anchor spot pairs plus
  a CCA-derived fcca feature to guide spot-to-spot alignment.
when_to_use_negative:
- Input datasets already share spots or features, so no cross-modality anchor pairing
  or CCA variate is needed
- Data has not been through manual gross alignment / icp_3d_alignment, since anchor
  pairing is computed on the aligned b_prime object
- The modality pair is not spatial transcriptome + spatial metabolome (e.g. histological
  images or spatial proteomics), which haCCA does not yet support
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: haCCA
  role: Python package providing the alignment and CCA workflow used to identify anchor
    spot pairs and derive the fcca feature
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Standard processing of spatial data and extraction of the feature matrix from
    .h5ad inputs
- name: ICP
  role: Iterative Closest Point algorithm that performs final alignment using morphological
    data plus the fcca variate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:06bbaad0f17dd1d610629a92b4e084081f889385de72c2837d15ad7114fb9f2c
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
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# spatial-anchor-spot-pair-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify anchor spot pairs between a spatial transcriptome dataset and a MALDI-MSI dataset using spatial-distance and Simpson-diversity (<0.2) criteria, then compute Pearson correlations on their paired gene–m/z features and run CCA to derive a high-correlated variate (fcca) appended to both datasets before ICP alignment.

## When to use

Use when integrating spatial transcriptome (e.g. 10X Visium .h5ad) and MALDI-MSI data that share no spots or features, and you need anchor spot pairs plus a CCA-derived fcca feature to guide spot-to-spot alignment.

## When NOT to use

- Input datasets already share spots or features, so no cross-modality anchor pairing or CCA variate is needed
- Data has not been through manual gross alignment / icp_3d_alignment, since anchor pairing is computed on the aligned b_prime object
- The modality pair is not spatial transcriptome + spatial metabolome (e.g. histological images or spatial proteomics), which haCCA does not yet support

## Inputs

- Aligned spatial transcriptome Data object (a) built from a .h5ad file (X feature matrix, D spatial coordinates, optional cluster Label)
- Gross/ICP-aligned MALDI-MSI Data object (b_prime) built from a .h5ad file (X m/z peak/intensity matrix, D spatial coordinates)
- Candidate spot-pair table with spot pair identifiers, spatial distance, and neighbor label composition

## Outputs

- Anchor-pairs table containing spot pair identifiers, distance, Simpson diversity (<0.2), and Pearson correlation values
- fcca feature vectors (one extra feature dimension appended to the original feature matrices of both datasets)

## How to apply

Load the spatial transcriptome Data object (a) and the gross/ICP-aligned MALDI-MSI Data object (b_prime) as (X, D, Label) triplets from .h5ad files. Filter candidate spot pairs by spatial-distance and a Simpson-diversity threshold of <0.2 on neighbor labels to retain anchor spot pairs. On the retained anchors, compute Pearson correlations between gene features in a and metabolite m/z features in b_prime to select high-correlated gene–metabolite feature pairs. Convert those pairs into new high-correlated variables and run canonical correlation analysis (CCA) to obtain the fcca variate, appending it as one extra feature dimension to both feature matrices; alignment by the ICP algorithm then uses morphological data plus this variate.

## Related tools

- **haCCA** (Python package providing the alignment and CCA workflow used to identify anchor spot pairs and derive the fcca feature) — https://github.com/LittleLittleCloud/haCCA
- **Scanpy** (Standard processing of spatial data and extraction of the feature matrix from .h5ad inputs)
- **ICP** (Iterative Closest Point algorithm that performs final alignment using morphological data plus the fcca variate)

## Examples

```
a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); b_prime = Data(X=b_prime_h5ad.X.toarray(), D=b_prime_h5ad.obsm['spatial']); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime)
```

## Evaluation signals

- Anchor-pairs table exists and contains expected fields: spot pair identifiers, distance, Simpson diversity <0.2, and Pearson correlation values
- fcca feature vectors exist and the feature matrices gain exactly one extra feature dimension
- Spot pairs failing the spatial-distance criterion or the Simpson-diversity <0.2 threshold are excluded from the retained anchor set
- Selected gene–metabolite pairs show high Pearson correlation between gene features in a and m/z features in b_prime
- Downstream ICP alignment using morphological data plus fcca improves label transfer accuracy / ARI relative to morphological-only alignment (haCCA outperformed STUtility by 6-20%)

## Limitations

- haCCA's behavior on higher resolution data has not been evaluated
- Integration is limited to spatial transcriptome and spatial metabolome data; image-based sequencing, spatial proteomics, and histological images are not yet supported
- No data accession or dataset identifier is given for the pseudo-paired and real-world evaluation data

## Evidence

- [methods] Steps 4-5 of haCCA identify anchor spot pairs (close distance, low Simpson diversity <0.2 neighbors): [evidence span withheld — anchor status: not_found]
- [methods] apply CCA to generate a new high-correlated variate fcca that is added to both datasets as an individual feature: [evidence span withheld — anchor status: not_found]
