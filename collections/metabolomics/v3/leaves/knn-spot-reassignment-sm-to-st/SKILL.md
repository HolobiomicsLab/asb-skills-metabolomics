---
name: knn-spot-reassignment-sm-to-st
description: Use when ST and SM data come from adjacent or serial tissue sections
  at different spatial resolutions and a unified spot-level metabolite-by-spot matrix
  (MIM) is needed before vertical integration.
when_to_use_negative:
- Data are already measured on the same tissue section at matching spatial resolution,
  so no cross-section alignment is needed
- Single-cell or subcellular resolution mapping is required — SpatialMETA is designed
  for low-resolution ST and SM data only
- Aligning more than two adjacent sections or building 3D spatial omics — multi-section
  alignment is not implemented
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: SpatialMETA
  role: Provides the AlignmentModule (fit_alignment), spot_transform_by_manual, new_spot_sample,
    spot_align_byknn (KNN with K=5, threshold d_min * f_dist, f_dist default 1.5),
    and calculate_dot_df functions used for alignment and reassignment
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:af26d1da7a75c59e06c89d88445c510e84e8b34646cce5bb9bbf4910764dde9d
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# KNN spot reassignment of SM data to ST coordinates

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reassigns spatial metabolomics (SM) spots from an adjacent tissue section onto spatial transcriptomics (ST) spot coordinates via K-nearest-neighbor matching and intensity aggregation, unifying both modalities to a consistent spatial resolution prior to SpatialMETA integration.

## When to use

Use when ST and SM data come from adjacent or serial tissue sections at different spatial resolutions and a unified spot-level metabolite-by-spot matrix (MIM) is needed before vertical integration.

## When NOT to use

- Data are already measured on the same tissue section at matching spatial resolution, so no cross-section alignment is needed
- Single-cell or subcellular resolution mapping is required — SpatialMETA is designed for low-resolution ST and SM data only
- Aligning more than two adjacent sections or building 3D spatial omics — multi-section alignment is not implemented

## Inputs

- SM AnnData (AnnDataSM) h5ad file, e.g., from spatialmeta.data.load_Vicari_2024_msi
- ST AnnData (AnnDataST) h5ad file with spot spatial coordinates
- Learned alignment transformation (affine matrix A and optional diffeomorphic V) from spatialmeta.model.AlignmentModule

## Outputs

- reassigned_sm.h5ad — SM AnnData with spots remapped onto ST spot coordinates
- mim_aggregated.tsv — metabolite intensity matrix (MIM) aggregated onto unified ST spots

## How to apply

Load SM and ST data as AnnData objects (spatialmeta.AnnDataSM, spatialmeta.AnnDataST, e.g., from the Vicari_2024_msi example dataset or user-supplied h5ad files). First learn the SM-to-ST alignment with spatialmeta.model.AlignmentModule.fit_alignment, which fits an affine matrix A (rotation/translation) plus an optional diffeomorphic transformation V adapted from STalign, then apply it to SM coordinates with spatialmeta.pp.spot_transform_by_manual. Generate the unified target spot grid on the ST section with spatialmeta.pp.new_spot_sample. Reassign SM spots to ST spots with spatialmeta.pp.spot_align_byknn using K=5 nearest neighbors and a distance threshold of d_min * f_dist (default f_dist=1.5), dropping assignments that exceed the threshold. Finally, aggregate SM metabolite intensities onto the reassigned ST spots (spatialmeta.pp.calculate_dot_df) to build the MIM and save reassigned_sm.h5ad and mim_aggregated.tsv.

## Related tools

- **SpatialMETA** (Provides the AlignmentModule (fit_alignment), spot_transform_by_manual, new_spot_sample, spot_align_byknn (KNN with K=5, threshold d_min * f_dist, f_dist default 1.5), and calculate_dot_df functions used for alignment and reassignment) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
from spatialmeta.pp import spot_align_byknn; reassigned_sm = spot_align_byknn(sm_data, st_data, k=5, f_dist=1.5)
```

## Evaluation signals

- Both output files (reassigned_sm.h5ad, mim_aggregated.tsv) exist and are well-formed h5ad/TSV
- Every retained SM-to-ST assignment satisfies the KNN distance threshold d_min * 1.5 (default f_dist)
- The reassigned SM AnnData spot count matches the unified ST spot grid produced by new_spot_sample
- Aggregated metabolite intensities in the MIM equal the average of the K=5 nearest-neighbor SM spot intensities for each ST spot
- SM spots placed by spot_transform_by_manual fall within the ST coordinate space after applying the learned affine A and diffeomorphic V

## Limitations

- Alignment and reassignment are currently applied only between two adjacent sections within a single sample; multi-section/3D alignment is not supported
- Histological images for SM (Image_SM) are often unavailable, requiring outline-based approximation from spot coordinates
- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution reassignment

## Evidence
