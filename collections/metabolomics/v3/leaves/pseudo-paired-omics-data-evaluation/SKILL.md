---
name: pseudo-paired-omics-data-evaluation
description: Use when evaluating or benchmarking a spatial omics integration method
  (e.g. haCCA) that claims to outperform morphological alignment (STUtility, ICP)
  on data where no true shared spots exist, by building a pseudo-paired dataset with
  a known ground truth for label transfer.
when_to_use_negative:
- Data contains true shared spots or features between the spatial transcriptome and
  MALDI-MSI modalities, so a paired evaluation with real correspondence is possible
  instead of a pseudo-paired surrogate.
- The goal is to integrate real-world normal tissue or tumor data rather than to quantify
  benchmark performance against morphological alignment methods.
- Only a single spatial modality is available (no second spatial metabolome/MSI layer),
  so no cross-modality label transfer can be scored.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0154
tools:
- name: haCCA
  role: Provides the Data triplet (X, D, Label) structure, the manual_gross_alignment
    / icp_3d_alignment / direct_alignment pipeline, and label-transfer utilities used
    to generate the candidate alignment being benchmarked.
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Used to read the a.h5ad and b_prime.h5ad pseudo-paired AnnData files and extract
    X and obsm['spatial'] before constructing haCCA Data objects.
- name: STUtility
  role: Morphological-alignment comparator applied to the same pseudo-paired pairs
    to yield the baseline label-transfer AC/ARI that haCCA must exceed.
- name: ICP
  role: Standalone iterative closest point alignment run on the pseudo-paired pairs
    as a second comparator against haCCA's combined alignment.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:06a85cb212e9e3a6359254a4da3b2a39a5ac07e29b4d4d9ae9eb08f05ad1cb48
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pseudo-paired-omics-data-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Creates pseudo-paired spatial transcriptome/MALDI-MSI data with known ground-truth correspondence so that haCCA's label-transfer accuracy (AC) and ARI can be benchmarked against morphological-only alternatives such as STUtility and standalone ICP.

## When to use

Use when evaluating or benchmarking a spatial omics integration method (e.g. haCCA) that claims to outperform morphological alignment (STUtility, ICP) on data where no true shared spots exist, by building a pseudo-paired dataset with a known ground truth for label transfer.

## When NOT to use

- Data contains true shared spots or features between the spatial transcriptome and MALDI-MSI modalities, so a paired evaluation with real correspondence is possible instead of a pseudo-paired surrogate.
- The goal is to integrate real-world normal tissue or tumor data rather than to quantify benchmark performance against morphological alignment methods.
- Only a single spatial modality is available (no second spatial metabolome/MSI layer), so no cross-modality label transfer can be scored.

## Inputs

- Pseudo-paired spatial transcriptome .h5ad file (a.h5ad) with feature matrix X and spatial coordinates in obsm['spatial']
- Pseudo-paired MALDI-MSI .h5ad file (b_prime.h5ad) with feature matrix X and spatial coordinates in obsm['spatial']
- Cluster Label arrays attached to each Data object encoding ground-truth spot correspondence

## Outputs

- b_predict (aligned feature from a and samples from b_prime) for each of haCCA, STUtility and ICP alignments
- Label transfer accuracy (AC) per method
- Label transfer ARI per method
- benchmark_alignment_comparison.csv assembling AC and ARI for all four methods

## How to apply

Load the pseudo-paired spatial transcriptome and MALDI-MSI data (a.h5ad and b_prime.h5ad) with Scanpy and wrap them into haCCA Data objects, providing X (feature matrix), D (spatial coordinates from obsm['spatial']), and optional Label arrays carrying known cluster annotations so that spot correspondence is known by construction. Run the haCCA pipeline — manual_gross_alignment followed by icp_3d_alignment followed by direct_alignment — to infer b_predict, and in parallel run the comparator alignments: STUtility morphological alignment and a standalone ICP alignment on the same pseudo-paired pairs. Transfer cluster labels across aligned spots for each of the four methods, then compute label transfer accuracy (AC) and label transfer ARI per method. Judge success by whether the haCCA alignment shows the highest AC and ARI, reproducing the article's 6–20% AC gain over morphological alignment (represented by STUtility), and assemble the per-method metrics into a benchmark_alignment_comparison.csv.

## Related tools

- **haCCA** (Provides the Data triplet (X, D, Label) structure, the manual_gross_alignment / icp_3d_alignment / direct_alignment pipeline, and label-transfer utilities used to generate the candidate alignment being benchmarked.) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Used to read the a.h5ad and b_prime.h5ad pseudo-paired AnnData files and extract X and obsm['spatial'] before constructing haCCA Data objects.)
- **STUtility** (Morphological-alignment comparator applied to the same pseudo-paired pairs to yield the baseline label-transfer AC/ARI that haCCA must exceed.)
- **ICP** (Standalone iterative closest point alignment run on the pseudo-paired pairs as a second comparator against haCCA's combined alignment.)

## Examples

```
from hacca import *
import scanpy as sc
a_h5ad = sc.read_h5ad('/path/to/a.h5ad')
b_prime_h5ad = sc.read_h5ad('/path/to/b_prime.h5ad')
a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial'])
b_prime = Data(X=b_prime_h5ad.X.toarray(), D=b_prime_h5ad.obsm['spatial'])
_b = hacca.manual_gross_alignment(a, b_prime)
_a, _b = hacca.icp_3d_alignment(a, _b)
b_predict = hacca.direct_alignment(_a, _b)
```

## Evaluation signals

- On pseudo multi-module spatial data, haCCA should achieve the highest label transfer accuracy (AC) and label transfer ARI compared with direct alignment, manual alignment (STUtility) and ICP.
- haCCA's combined alignment should outperform the morphological-only alignment (STUtility) by 6–20% AC on the pseudo-paired dataset.
- benchmark_alignment_comparison.csv should contain one AC and one ARI value for each of the four methods (haCCA, direct alignment, STUtility manual alignment, ICP) and be internally consistent with the label-transfer step outputs.
- The pseudo-paired b_prime must preserve the ground-truth cluster Label arrays so label transfer can be scored; a missing or mismatched Label array invalidates the benchmark.

## Limitations

- No data accession or dataset identifier is given for the spatial transcriptome or pseudo-paired MSI-MALDI data, limiting reproducibility of the benchmark from public sources alone.
- No software version number is stated for STUtility (or ICP) used as the benchmark comparator, so exact comparator behavior may vary.
- The benchmark uses pseudo-paired data; behavior of haCCA on higher-resolution real data is not covered by this evaluation scheme.

## Evidence

- [results] haCCA best performance on pseudo multi-module data: [evidence span withheld — anchor status: not_found]
