---
name: sm-intensity-aggregation-onto-st-spots
description: Use when SM and ST are measured on adjacent/serial tissue sections with
  mismatched spot grids, and integration requires metabolite intensities aggregated
  onto ST spots before CVAE-based vertical integration.
when_to_use_negative:
- Data are already at single-cell or subcellular resolution — SpatialMETA is designed
  for low-resolution ST and SM data only
- ST and SM were measured on the same section and already share an identical spot
  coordinate system, so no cross-section alignment/reassignment is needed
- More than two modalities must be integrated — SpatialMETA currently only supports
  two-modality integration
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3308
tools:
- name: SpatialMETA
  role: Provides spot_transform_by_manual, new_spot_sample, spot_align_byknn, and
    calculate_dot_df for coordinate transformation, KNN reassignment, and intensity
    aggregation
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:edfd4dbf4755db18830bb38716f45a58caf846b03e49b1b12d740fa873f9fceb
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sm-intensity-aggregation-onto-st-spots

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unifies spatial metabolomics (SM) data with spatial transcriptomics (ST) spot resolution by aligning SM spot coordinates into the ST frame and reassigning continuous mass-spectrometry metabolite intensities to ST spots via K-nearest-neighbor aggregation, producing a metabolite intensity matrix (MIM) on the ST grid.

## When to use

Use when SM and ST are measured on adjacent/serial tissue sections with mismatched spot grids, and integration requires metabolite intensities aggregated onto ST spots before CVAE-based vertical integration.

## When NOT to use

- Data are already at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data only
- ST and SM were measured on the same section and already share an identical spot coordinate system, so no cross-section alignment/reassignment is needed
- More than two modalities must be integrated — SpatialMETA currently only supports two-modality integration

## Inputs

- SM AnnData (h5ad) with spot coordinates and continuous metabolite intensity values
- ST AnnData (h5ad) with spot positions defining the target grid
- Vicari_2024_msi example dataset or user-supplied h5ad files

## Outputs

- reassigned_sm.h5ad — SM AnnData with spots reassigned to ST coordinates
- mim_aggregated.tsv — metabolite intensity matrix (MIM) aggregated on ST spots

## How to apply

Load SM and ST as AnnData objects (spatialmeta.AnnDataST, spatialmeta.AnnDataSM, e.g., from the Vicari_2024_msi dataset or user-supplied h5ad files). Learn the SM-to-ST alignment with spatialmeta.model.AlignmentModule.fit_alignment, fitting an Affine Matrix A for rotation/translation plus an optional diffeomorphic transformation V (adapted from STalign), then apply it to SM spot coordinates with spatialmeta.pp.spot_transform_by_manual. Generate the unified target spot grid on the ST section with spatialmeta.pp.new_spot_sample, then reassign SM spots to ST spots using spatialmeta.pp.spot_align_byknn with K=5 nearest neighbors, dropping matches beyond the distance threshold d_min * f_dist (default f_dist=1.5, user-definable). Finally, aggregate the SM metabolite intensities onto each reassigned ST spot (averaging the K neighbors' intensity values) to build the MIM with spatialmeta.pp.calculate_dot_df, saving reassigned_sm.h5ad and mim_aggregated.tsv.

## Related tools

- **SpatialMETA** (Provides spot_transform_by_manual, new_spot_sample, spot_align_byknn, and calculate_dot_df for coordinate transformation, KNN reassignment, and intensity aggregation) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
reassigned_sm = spatialmeta.pp.spot_align_byknn(st, transformed_sm, k=5, f_dist=1.5); mim = spatialmeta.pp.calculate_dot_df(st, reassigned_sm)
```

## Evaluation signals

- Both output files exist and are well-formed (valid .h5ad and .tsv)
- Every retained SM-to-ST assignment satisfies the KNN distance threshold (d_min * f_dist, f_dist default 1.5)
- The reassigned SM dataset resolution matches the ST spot grid — one aggregated SM entry per retained ST spot
- Aggregated metabolite intensities are averages of the K=5 nearest SM spots for each ST spot, not raw untransformed intensities

## Limitations

- Alignment and reassignment are currently applied only between two adjacent sections within a single sample; multi-section alignment for 3D spatial omics is not implemented
- Histological images for SM (Image_SM) are often unavailable, so the section outline must be approximated from SM spot coordinates
- SpatialMETA is designed for low-resolution ST and SM data; cellular or subcellular-level reassignment would require computational advancements for precise cell segmentation that are not yet implemented

## Evidence
