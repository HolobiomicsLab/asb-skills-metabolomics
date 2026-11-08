---
name: spatial-transcriptome-msi-alignment-benchmarking
description: Use when two neighboring tissue sections have been profiled with spatial
  transcriptomics (e.g. 10X Visium .h5ad) and MALDI-MSI, you lack shared spots/features,
  and you must decide which spot-to-spot alignment method maximizes label transfer
  fidelity.
when_to_use_negative:
- Your two modalities share actual spots or features, so alignment-free direct matching
  suffices and pseudo-pairing benchmarks are unnecessary
- Your goal is real-world integration and metabolic-state discovery (e.g. NETs-induced
  Scd1 fatty-acid changes in ICC) rather than quantifying alignment method differences
- Your data is not spot-resolved spatial transcriptome/spatial metabolome — higher-resolution
  data or other modalities (image-based sequencing, spatial proteomics, histological
  images) have not been evaluated with these alignment baselines
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3173
tools:
- name: haCCA
  role: Provides the cascade of manual_gross_alignment, icp_3d_alignment, and direct_alignment
    on Data triplets, and the label-transfer evaluation target being benchmarked
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Standard preprocessing of the spatial data and extraction of the feature matrix
    and spatial coordinates via sc.read_h5ad before Data-object construction
- name: STUtility
  role: Morphological-alignment comparator (manual alignment baseline) run on the
    same pseudo-paired pairs; represents the morphological-only alignment that haCCA
    outperforms by 6–20%
- name: ICP
  role: Standalone Iterative Closest Point alignment comparator; haCCA also uses icp_3d_alignment
    internally as a middle stage of its own cascade
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:800a9f5682a34c152cadff14eed18f87dd483f9086bbfd77918c968350afbe28
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-transcriptome-msi-alignment-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks label-transfer performance of spatial coordinate alignment strategies (haCCA's combined manual_gross_alignment → icp_3d_alignment → direct_alignment pipeline vs STUtility morphological alignment and standalone ICP) on pseudo-paired spatial transcriptome / MALDI-MSI data, scoring each method by label transfer accuracy (AC) and ARI.

## When to use

Use when two neighboring tissue sections have been profiled with spatial transcriptomics (e.g. 10X Visium .h5ad) and MALDI-MSI, you lack shared spots/features, and you must decide which spot-to-spot alignment method maximizes label transfer fidelity.

## When NOT to use

- Your two modalities share actual spots or features, so alignment-free direct matching suffices and pseudo-pairing benchmarks are unnecessary
- Your goal is real-world integration and metabolic-state discovery (e.g. NETs-induced Scd1 fatty-acid changes in ICC) rather than quantifying alignment method differences
- Your data is not spot-resolved spatial transcriptome/spatial metabolome — higher-resolution data or other modalities (image-based sequencing, spatial proteomics, histological images) have not been evaluated with these alignment baselines

## Inputs

- a.h5ad — spatial transcriptome section (10X Visium) read with Scanpy, with .X feature matrix and .obsm['spatial'] coordinates
- b_prime.h5ad — pseudo-paired MALDI-MSI section (peak/intensity feature matrix plus spatial coordinates)
- Optional cluster Label arrays attached to each Data object as label-transfer ground truth
- Pseudo multi-module spatial data constructed from the mouse brain 10X Visium + MALDI-MSI cohort

## Outputs

- b_predict — Data object containing aligned features from a and samples from b_prime
- Per-method label transfer accuracy (AC) and label transfer ARI values
- benchmark_alignment_comparison.csv assembling AC/ARI for haCCA, direct alignment, STUtility manual alignment, and ICP
- Reproduction of the Figure 2 alignment-benchmark comparison

## How to apply

Load the paired sections (a.h5ad, b_prime.h5ad) with Scanpy and wrap each as a haCCA Data triplet of (X feature matrix, D spatial matrix from obsm['spatial'], optional Label cluster array). Run the haCCA cascade — manual_gross_alignment, then icp_3d_alignment, then direct_alignment — to infer b_predict, and run the comparators (STUtility morphological alignment and standalone ICP) on the same pseudo-paired sections. For each of the four methods, transfer cluster labels across aligned spots, compute label transfer accuracy (AC) and label transfer ARI, and assemble the results into benchmark_alignment_comparison.csv. Judgment criterion: haCCA's combined alignment should beat the morphological-only baseline (represented by STUtility) by 6–20% AC, and the modified morphological alignment component of haCCA alone should already outperform STUtility.

## Related tools

- **haCCA** (Provides the cascade of manual_gross_alignment, icp_3d_alignment, and direct_alignment on Data triplets, and the label-transfer evaluation target being benchmarked) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Standard preprocessing of the spatial data and extraction of the feature matrix and spatial coordinates via sc.read_h5ad before Data-object construction)
- **STUtility** (Morphological-alignment comparator (manual alignment baseline) run on the same pseudo-paired pairs; represents the morphological-only alignment that haCCA outperforms by 6–20%)
- **ICP** (Standalone Iterative Closest Point alignment comparator; haCCA also uses icp_3d_alignment internally as a middle stage of its own cascade)

## Examples

```
from hacca import *; a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); b_prime = Data(X=b_prime_h5ad.X.toarray(), D=b_prime_h5ad.obsm['spatial']); b_predict = hacca.direct_alignment(*hacca.icp_3d_alignment(a, hacca.manual_gross_alignment(a, b_prime)))
```

## Evaluation signals

- haCCA's combined alignment achieves the highest label transfer AC and ARI among the four methods (haCCA, direct, STUtility manual, ICP) on pseudo multi-module spatial data
- Combined haCCA alignment outperforms the STUtility morphological-only alignment by 6–20% AC
- The modified morphological alignment component of haCCA alone outperforms STUtility, and combining alignments further increases accuracy
- benchmark_alignment_comparison.csv contains AC and ARI rows for all four compared methods on the same pseudo-paired pairs
- All methods are evaluated on identical pseudo-paired section pairs so AC/ARI differences are attributable to alignment strategy

## Limitations

- No software version number stated for STUtility used as the benchmark comparator, and no software version specified for hacca (pip install hacca)
- No data accession or dataset identifier given for the spatial transcriptome or MSI-MALDI data used in the pseudo-paired evaluation
- Behavior of haCCA on higher-resolution data has not been evaluated
- Evaluation is limited to spatial transcriptome and spatial metabolome data; other modalities not yet supported

## Evidence

- [results] benchmark finding on pseudo multi-module data: [evidence span withheld — anchor status: not_found]
