---
name: icp-point-cloud-registration
description: Use when two spatial omics datasets (e.g. spatial transcriptome .h5ad
  and MALDI-MSI peak/intensity matrices) from neighbor tissue sections lack shared
  spots or features and their spot coordinate point sets must be geometrically aligned
  for spot-to-spot integration.
when_to_use_negative:
- The two datasets already share spots or features, so spot-to-spot integration does
  not require geometric point-cloud registration.
- The goal is solely to detect high correlated gene-metabolite feature pairs without
  aligning spatial coordinates.
- Input data are higher-resolution than spot-level (e.g. subcellular) and the behavior
  of the haCCA alignment pipeline has not been evaluated there.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: haCCA
  role: provides icp_3d_alignment function implementing Iterative Closest Point registration
    of spatial coordinate matrices
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: reads the two h5ad spatial datasets from which X feature matrix and D spatial
    coordinates are extracted
- name: Python
  role: language in which haCCA is published as an easy-to-use package installed via
    pip install hacca
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:d68ea3edd1268efc881822d84afaba6c205d77cdaefc49fa8baa7d21580e557a
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# icp-point-cloud-registration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Registers spatial coordinate sets from two tissue sections by iteratively closest-point aligning points, as implemented in the haCCA icp_3d_alignment step that refines the initial gross alignment of spatial transcriptome and MALDI-MSI spot coordinates before further direct alignment.

## When to use

Use when two spatial omics datasets (e.g. spatial transcriptome .h5ad and MALDI-MSI peak/intensity matrices) from neighbor tissue sections lack shared spots or features and their spot coordinate point sets must be geometrically aligned for spot-to-spot integration.

## When NOT to use

- The two datasets already share spots or features, so spot-to-spot integration does not require geometric point-cloud registration.
- The goal is solely to detect high correlated gene-metabolite feature pairs without aligning spatial coordinates.
- Input data are higher-resolution than spot-level (e.g. subcellular) and the behavior of the haCCA alignment pipeline has not been evaluated there.

## Inputs

- a_h5ad: spatial transcriptome AnnData file with feature matrix and obsm['spatial'] coordinates
- b_prime_h5ad: MALDI-MSI or second spatial dataset AnnData file with feature matrix and obsm['spatial'] coordinates
- Data triplet (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray]) for each section
- grossly aligned coordinate set _b_prime produced by manual_gross_alignment

## Outputs

- _a: transformed/refined spatial transcriptome coordinate matrix
- _b_prime: ICP-aligned coordinate matrix of the second dataset
- affine/translation matrices recovered from the alignment step, persisted alongside coordinate arrays in an npz archive

## How to apply

First read the two h5ad files with Scanpy and build haCCA Data triplets with X as the densified feature matrix (a_h5ad.X.toarray()) and D as obsm['spatial'] coordinates. Then run the initial gross affine alignment with hacca.manual_gross_alignment(a, b_prime) so the two coordinate point clouds are roughly overlaid. Next apply hacca.icp_3d_alignment(a, _b_prime), which uses the Iterative Closest Point algorithm to repeatedly match each point to its nearest neighbor in the other set and estimate a rigid transform that minimizes spot-spot distances; it returns the refined pair (_a, _b_prime). The rationale, per the article, is that morphological-based alignment via ICP alone is less accurate than haCCA's full workflow (STUtility trails by 6-20% on label transfer accuracy and label transfer ARI), so ICP serves as the intermediate refinement between gross alignment and the subsequent direct_alignment that minimizes outlier-to-nearest-neighbor distances.

## Related tools

- **haCCA** (provides icp_3d_alignment function implementing Iterative Closest Point registration of spatial coordinate matrices) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (reads the two h5ad spatial datasets from which X feature matrix and D spatial coordinates are extracted)
- **Python** (language in which haCCA is published as an easy-to-use package installed via pip install hacca)

## Examples

```
_a, _b_prime = hacca.icp_3d_alignment(a, _b_prime)
```

## Evaluation signals

- Confirm the returned pair (_a, _b_prime) has coordinate matrices with the same shape as the original inputs, indicating both point clouds were transformed.
- Validate that per-point nearest-neighbor distances between _a and _b_prime are reduced relative to the gross-aligned input.
- Check that the downstream direct_alignment(_a, _b_prime) call completes and returns a b_predict object, indicating ICP output is geometrically consistent enough to serve as its input.
- Assess final integration quality by label transfer accuracy and label transfer ARI, where haCCA was shown to outperform STUtility by 6-20% on pseudo-paired MSI-MALDI data.
- Persist translation matrices recovered from the alignment steps in an npz archive and verify they can reproduce the coordinate transforms.

## Limitations

- No software version for the hacca package is specified beyond the pip install command, and no changelog exists for the repository.
- ICP is only the middle stage; the modified morphological alignment component alone outperforms STUtility but the full haCCA accuracy gain comes from combining alignment with high-correlated-variable integration.
- Behavior on higher-resolution data has not been evaluated.
- Only spatial transcriptome and spatial metabolome data are supported; other modalities such as image-based sequencing or spatial proteomics are not yet integrated.

## Evidence
