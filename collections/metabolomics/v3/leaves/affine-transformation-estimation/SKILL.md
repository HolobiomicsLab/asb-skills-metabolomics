---
name: affine-transformation-estimation
description: Use when two neighboring spatial-omics sections (e.g. h5ad files with
  obsm['spatial'] coordinates from 10X Visium and MALDI-MSI) share no spots or features
  and must be registered before spot-to-spot integration.
when_to_use_negative:
- When the two sections already share spots or features (e.g. paired or co-registered
  data), since the morphological alignment step exists precisely because there are
  'no shared spots or features' between spatial transcriptomics and MALDI-MSI data.
- When only feature-level integration (e.g. high-correlated gene-metabolite pairs)
  is needed without any spatial coordinate registration of the two tissue sections.
- When data are not in the expected h5ad format with an obsm['spatial'] coordinate
  matrix, since the Data construction relies on a_h5ad.obsm['spatial'].
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3542
- http://edamontology.org/topic_3452
tools:
- name: haCCA
  role: Python package providing manual_gross_alignment, icp_3d_alignment, and direct_alignment
    functions that estimate the affine/translation matrices for morphological alignment
    of the two sections' spatial coordinates.
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Used to read the a.h5ad and b_prime.h5ad input files (sc.read_h5ad) and expose
    the obsm['spatial'] coordinate matrices and feature matrices that feed haCCA's
    Data objects.
- name: ICP
  role: The iterative closest point algorithm underlying hacca.icp_3d_alignment, which
    refines the gross affine alignment by iteratively closest-point aligning the two
    coordinate sets.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:9b60cb10e0e0458708f5e81f3eb92fbec23508d0bb25c6502a0a9d18ab7d235b
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3542
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# affine-transformation-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate the affine/translation matrices that morphologically align spatial coordinates from two neighboring tissue sections (e.g. a 10X Visium spatial transcriptome section and a MALDI-MSI section) by combining a manual gross alignment, iterative closest-point (ICP) 3D alignment, and outlier-spot distance minimization within the haCCA workflow.

## When to use

Use when two neighboring spatial-omics sections (e.g. h5ad files with obsm['spatial'] coordinates from 10X Visium and MALDI-MSI) share no spots or features and must be registered before spot-to-spot integration.

## When NOT to use

- When the two sections already share spots or features (e.g. paired or co-registered data), since the morphological alignment step exists precisely because there are 'no shared spots or features' between spatial transcriptomics and MALDI-MSI data.
- When only feature-level integration (e.g. high-correlated gene-metabolite pairs) is needed without any spatial coordinate registration of the two tissue sections.
- When data are not in the expected h5ad format with an obsm['spatial'] coordinate matrix, since the Data construction relies on a_h5ad.obsm['spatial'].

## Inputs

- Two .h5ad files (a.h5ad and b_prime.h5ad) read with sc.read_h5ad, one for the spatial transcriptome section and one for the MALDI-MSI section
- Feature matrices densified via h5ad.X.toarray() for each section
- Spatial coordinate matrices from h5ad.obsm['spatial'] for each section

## Outputs

- Transformed coordinate matrices _a and _b_prime after manual gross and ICP 3D alignment
- b_predict: aligned Data object containing features from a and samples from b_prime
- Affine/translation matrices recovered from manual_gross_alignment, icp_3d_alignment, and direct_alignment steps
- Persisted npz archive holding aligned coordinate arrays and translation matrices

## How to apply

Read both sections as h5ad files (e.g. a.h5ad, b_prime.h5ad) with Scanpy and wrap each into a haCCA Data triplet whose X is the densified feature matrix (a_h5ad.X.toarray()) and D is the coordinate matrix from obsm['spatial']. Run hacca.manual_gross_alignment(a, b_prime) to build an initial affine matrix from manually selected points that grossly superimposes the MALDI-MSI section onto the transcriptome section. Refine the registration with hacca.icp_3d_alignment(a, _b_prime), which uses the iterative closest point (ICP) algorithm to iteratively closest-point align the two coordinate sets. Finally, run hacca.direct_alignment(_a, _b_prime), which finds outlier spots in the spatial transcriptome data lacking a MALDI-MSI neighbor and computes a translation matrix by minimizing their distances to nearest neighbor spots, producing b_predict that holds aligned features from a and samples from b_prime. Persist the aligned coordinate matrices from _a, _b_prime, and b_predict plus the affine translation matrices recovered from each step (e.g. in a single npz archive) so they can be fed to the downstream high-correlated-feature-pair integration stage.

## Related tools

- **haCCA** (Python package providing manual_gross_alignment, icp_3d_alignment, and direct_alignment functions that estimate the affine/translation matrices for morphological alignment of the two sections' spatial coordinates.) — https://github.com/LittleLittleCloud/haCCA
- **Scanpy** (Used to read the a.h5ad and b_prime.h5ad input files (sc.read_h5ad) and expose the obsm['spatial'] coordinate matrices and feature matrices that feed haCCA's Data objects.)
- **ICP** (The iterative closest point algorithm underlying hacca.icp_3d_alignment, which refines the gross affine alignment by iteratively closest-point aligning the two coordinate sets.)

## Examples

```
pip install hacca; from hacca import *; a = Data(X=a_h5ad.X.toarray(), D=a_h5ad.obsm['spatial']); _b_prime = hacca.manual_gross_alignment(a, b_prime); _a, _b_prime = hacca.icp_3d_alignment(a, _b_prime); b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Verify that the persisted npz archive contains coordinate arrays for both sections and the translation matrices from each alignment step.
- Confirm that manual_gross_alignment returns a transformed _b_prime, icp_3d_alignment returns a pair (_a, _b_prime), and direct_alignment returns b_predict with features from a and samples from b_prime.
- Benchmark the modified morphological alignment against STUtility using label transfer accuracy and label transfer ARI; haCCA outperformed it by 6-20%.
- Check that downstream integration using the aligned coordinates plus high-correlated variables yields more accurate integration than using only morphological alignment.
- On real-world normal tissue and tumor data (e.g. the ICC model), confirm the aligned coordinates enable generation of gene expression and biomolecule profiling.

## Limitations

- The behavior of haCCA on higher resolution data has not been evaluated.
- No software version number is specified (pip install hacca), and no data accession or dataset identifier is given for the pseudo-paired and real-world evaluations.
- Alignment quality is ultimately judged by label transfer accuracy and label transfer ARI against the STUtility benchmark (6-20% improvement); other metrics are not reported.

## Evidence

- [methods] manual_gross_alignment | icp_3d_alignment | direct_alignment: "manual_gross_alignment | icp_3d_alignment | direct_alignment"
- [readme] Data is a triplet of (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray]): "Data is a triplet of (X: np.ndarray, D: np.ndarray, Label: Optional[np.ndarray]), where X is the feature matrix, D is the spatial matrix that contains the location information"
- [intro] Alignment using morphological data and high-correlated variate between MALDI-MSI and spatial transcriptome is carried out by ICP(lterative Closest Point) algorithm: "Alignment using morphological data and high-correlated variate between MALDI-MSI and spatial transcriptome is carried out by ICP(lterative Closest Point) algorithm"
- [discussion] the modified morphological alignment of spatial coordinates of haCCA itself is able to outperform STUtility: "the modified morphological alignment of spatial coordinates of haCCA itself is able to outperform STUtility, and the combing alignment is able to further increase the accuracy"
- [abstract] integrating them with high accuracy is challenge due to no shared spots or features: "integrating them with high accuracy is challenge due to no shared spots or features"
- [readme] b_predict contains aligned feature from a and samples from b_prime: "b_predict contains aligned feature from a and samples from b_prime"
