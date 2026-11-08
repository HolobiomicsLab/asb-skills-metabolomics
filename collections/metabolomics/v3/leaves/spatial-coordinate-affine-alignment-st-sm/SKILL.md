---
name: spatial-coordinate-affine-alignment-st-sm
description: Use when ST and SM data come from adjacent/serial sections of the same
  tissue with mismatched coordinate systems, and you need SM metabolite intensities
  aggregated onto ST spot positions before integration.
when_to_use_negative:
- Data are at single-cell or subcellular resolution — SpatialMETA is designed for
  low-resolution ST and SM data and does not support single-cell or subcellular alignment
- More than two sections or a 3D multi-section alignment is needed — alignment is
  currently only applied between two adjacent sections within a single sample
- SM data are already on the same coordinate grid as ST (no cross-section alignment
  needed) — reassignment would introduce unnecessary smoothing
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: SpatialMETA
  role: Provides AlignmentModule.fit_alignment (affine A + diffeomorphic V adapted
    from STalign), spot_transform_by_manual, new_spot_sample, spot_align_byknn (K=5,
    f_dist=1.5), and calculate_dot_df for the full align-and-reassign pipeline
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: STalign
  role: Source of the diffeomorphic transformation V adapted by SpatialMETA's AlignmentModule
    for non-linear distortion beyond the affine matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:3e9a4ea92572e29943af351f015832ca8dad988e887ba41b742e140c4fa3a973
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  repo_url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - STalign
  - SpatialMETA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Spatial-coordinate affine alignment of ST and SM sections

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns spatial metabolomics (SM) spot coordinates onto spatial transcriptomics (ST) coordinates from an adjacent tissue section using an affine matrix (rotation/translation) plus optional diffeomorphic transformation, then reassigns SM spots to ST spots via KNN to unify resolution.

## When to use

Use when ST and SM data come from adjacent/serial sections of the same tissue with mismatched coordinate systems, and you need SM metabolite intensities aggregated onto ST spot positions before integration.

## When NOT to use

- Data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular alignment
- More than two sections or a 3D multi-section alignment is needed — alignment is currently only applied between two adjacent sections within a single sample
- SM data are already on the same coordinate grid as ST (no cross-section alignment needed) — reassignment would introduce unnecessary smoothing

## Inputs

- ST AnnData (spatialmeta.AnnDataST) with spot spatial coordinates (h5ad)
- SM AnnData (spatialmeta.AnnDataSM) with spot spatial coordinates and metabolite intensity matrix (h5ad)
- Example dataset: Vicari_2024_msi via spatialmeta.data.load_Vicari_2024_msi

## Outputs

- reassigned_sm.h5ad — SM AnnData with spots remapped to ST coordinates
- mim_aggregated.tsv — metabolite intensity matrix (MIM) aggregated onto ST spots
- Learned affine matrix A and optional diffeomorphic transformation V from AlignmentModule

## How to apply

Load SM and ST as AnnData objects (spatialmeta.AnnDataST, spatialmeta.AnnDataSM, e.g. from the Vicari_2024_msi example dataset or user h5ad files). Fit the alignment with spatialmeta.model.AlignmentModule.fit_alignment, which learns an Affine Matrix A for rotation/translation plus an optional diffeomorphic transformation V (adapted from STalign) and optional latent-space linear matrices. Apply the learned transform to SM spot coordinates with spatialmeta.pp.spot_transform_by_manual, then generate the unified target grid on the ST section with spatialmeta.pp.new_spot_sample. Reassign SM spots to ST spots using spatialmeta.pp.spot_align_byknn with K=5 nearest neighbors and a distance threshold of d_min * f_dist (default f_dist=1.5), dropping matches beyond the threshold. Aggregate metabolite intensities onto reassigned ST spots to build the metabolite intensity matrix (spatialmeta.pp.calculate_dot_df) and save reassigned_sm.h5ad plus the MIM table (mim_aggregated.tsv).

## Related tools

- **SpatialMETA** (Provides AlignmentModule.fit_alignment (affine A + diffeomorphic V adapted from STalign), spot_transform_by_manual, new_spot_sample, spot_align_byknn (K=5, f_dist=1.5), and calculate_dot_df for the full align-and-reassign pipeline) — https://github.com/WanluLiuLab/SpatialMETA
- **STalign** (Source of the diffeomorphic transformation V adapted by SpatialMETA's AlignmentModule for non-linear distortion beyond the affine matrix)

## Examples

```
am = sm.model.AlignmentModule(st, sm); am.fit_alignment(); sm_transformed = sm.pp.spot_transform_by_manual(sm, am); aligned = sm.pp.spot_align_byknn(st, sm_transformed, k=5, f_dist=1.5)
```

## Evaluation signals

- Both output files (reassigned_sm.h5ad and mim_aggregated.tsv) exist and are well-formed h5ad/tsv
- Every retained SM-to-ST assignment satisfies the KNN distance threshold (distance ≤ d_min * f_dist with f_dist=1.5)
- Reassigned SM spot count matches the number of retained ST spot assignments (spots beyond the threshold are dropped)
- MIM table dimensions correspond to reassigned ST spots × metabolite features, with intensities averaged from the K=5 nearest SM spots
- Transformed SM spot coordinates fall within the ST section bounds after applying the learned affine matrix A and transformation V

## Limitations

- Alignment and reassignment are currently applied only between two adjacent sections within a single sample; multi-section alignment for 3D spatial omics is not implemented
- Precise cell segmentation at cellular/subcellular levels for alignment/reassignment requires computational advancements not yet implemented
- Histological images for SM (Image_SM) are often unavailable, requiring outline-based approximation from SM spot coordinates

## Evidence

- [discussion] Image_SM unavailable outline: [evidence span withheld — anchor status: not_found]
