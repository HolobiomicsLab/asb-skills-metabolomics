---
name: diffeomorphic-image-based-coordinate-mapping
description: Use when ST and SM data come from adjacent or serial tissue sections
  whose spot coordinate systems differ, and you must learn or apply a rotation/translation
  plus non-linear distortion before aggregating SM intensities onto ST spots.
when_to_use_negative:
- Data are at single-cell or subcellular resolution — SpatialMETA's alignment/reassignment
  is designed for low-resolution ST and SM data and precise cell segmentation is not
  implemented
- More than two modalities must be aligned — SpatialMETA currently only supports integration
  of two modalities
- Multiple adjacent sections must be aligned for 3D spatial omics — alignment is only
  applied between two adjacent sections within a single sample
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: SpatialMETA
  role: Provides AlignmentModule.fit_alignment (affine A + optional STalign-adapted
    diffeomorphic V) and spot_transform_by_manual for mapping SM spots into ST coordinate
    space; output feeds spot_align_byknn and new_spot_sample
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:3331a334dc251b5d4d3123dba48aed524b4e70d4d75df33a8c92200c76f0e03c
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
  - SpatialMETA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Diffeomorphic image-based coordinate mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns spatial metabolomics (SM) spot coordinates onto a spatial transcriptomics (ST) section by fitting an affine matrix A (rotation/translation) plus an optional diffeomorphic transformation V (adapted from STalign), then applying the learned mapping to place SM spots in ST coordinate space prior to KNN-based reassignment.

## When to use

Use when ST and SM data come from adjacent or serial tissue sections whose spot coordinate systems differ, and you must learn or apply a rotation/translation plus non-linear distortion before aggregating SM intensities onto ST spots.

## When NOT to use

- Data are at single-cell or subcellular resolution — SpatialMETA's alignment/reassignment is designed for low-resolution ST and SM data and precise cell segmentation is not implemented
- More than two modalities must be aligned — SpatialMETA currently only supports integration of two modalities
- Multiple adjacent sections must be aligned for 3D spatial omics — alignment is only applied between two adjacent sections within a single sample

## Inputs

- ST AnnData object (spatialmeta.AnnDataST) with spot spatial coordinates, e.g. h5ad file or Vicari_2024_msi example dataset
- SM AnnData object (spatialmeta.AnnDataSM) with spot spatial coordinates and metabolite intensity matrix
- Optional SM outline/image approximated from SM spot coordinates when histological Image_SM is unavailable

## Outputs

- Learned affine matrix A and optional diffeomorphic transformation V from AlignmentModule
- SM AnnData with spots transformed into ST coordinate space (reassigned_sm.h5ad)
- Unified ST target spot grid from spatialmeta.pp.new_spot_sample
- SM-to-ST spot assignments from spot_align_byknn satisfying the d_min * 1.5 distance threshold

## How to apply

Load SM and ST data as AnnData objects (spatialmeta.AnnDataST, spatialmeta.AnnDataSM), e.g. from the Vicari_2024_msi example dataset or user-supplied h5ad files. Fit the alignment with spatialmeta.model.AlignmentModule.fit_alignment, which learns an Affine Matrix A for rotation/translation, an optional diffeomorphic transformation V adapted from STalign, and optional linear matrices W/V for latent-space alignment. Apply the learned transformation to SM spot coordinates with spatialmeta.pp.spot_transform_by_manual so SM spots are expressed in ST coordinate space. Generate the unified target spot grid on the ST section with spatialmeta.pp.new_spot_sample, then reassign SM spots to ST spots via spatialmeta.pp.spot_align_byknn (K=5, distance threshold d_min * f_dist with f_dist default 1.5). The rationale is that adjacent sections differ in spatial morphology, so a non-linear (diffeomorphic) component corrects local distortions that pure rigid rotation/translation cannot.

## Related tools

- **SpatialMETA** (Provides AlignmentModule.fit_alignment (affine A + optional STalign-adapted diffeomorphic V) and spot_transform_by_manual for mapping SM spots into ST coordinate space; output feeds spot_align_byknn and new_spot_sample) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
import spatialmeta as smeta; st, msi = smeta.data.load_Vicari_2024_msi(); model = smeta.model.AlignmentModule(); model.fit_alignment(st, msi); msi_aligned = smeta.pp.spot_transform_by_manual(msi, model)
```

## Evaluation signals

- Every retained SM-to-ST assignment from spot_align_byknn satisfies the KNN distance threshold d_min * f_dist (default d_min * 1.5)
- reassigned_sm.h5ad is a well-formed h5ad whose spot coordinates lie within the ST section's coordinate extent
- The transformed SM spot grid aligns with the ST spot grid produced by spatialmeta.pp.new_spot_sample
- Downstream MIM table (mim_aggregated.tsv) covers all ST spots that received reassigned SM intensities, with no unmatched spots left inconsistent
- Visually, aligned SM outline overlaps the ST histological outline better after diffeomorphic correction than after affine-only alignment

## Limitations

- Alignment and reassignment are currently applied only between two adjacent sections within a single sample; multi-section alignment for 3D spatial omics is not implemented
- SpatialMETA is designed for low-resolution ST and SM data; cellular or subcellular-level alignment would require computational advancements for precise cell segmentation
- Histological images for SM (Image_SM) are often unavailable, requiring outline-based approximation from spot coordinates, which limits image-based alignment accuracy

## Evidence

- [discussion] Missing SM histological images require coordinate-outline approximation: [evidence span withheld — anchor status: not_found]
