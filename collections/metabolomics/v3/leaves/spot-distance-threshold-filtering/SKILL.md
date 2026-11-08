---
name: spot-distance-threshold-filtering
description: Use when SM spots from an adjacent tissue section have been aligned and
  reassigned to ST spot coordinates via KNN, and mappings beyond the trustworthy spatial
  distance must be removed before aggregation.
when_to_use_negative:
- Input data are at single-cell or subcellular resolution — SpatialMETA's alignment
  and KNN reassignment are designed for low-resolution ST and SM data only
- ST and SM measurements come from the same tissue section already sharing coordinates,
  so no cross-section reassignment or distance filtering is needed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3520
tools:
- name: SpatialMETA
  role: Provides spatialmeta.pp.spot_align_byknn, which performs the KNN reassignment
    and applies the d_min * f_dist distance threshold to drop distant SM-to-ST matches
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:d5887027bd578a5bb5c270baa0c203200c304bc04fa859a806e199a5bc735eb7
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spot-distance-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters SM-to-ST spot assignments produced by KNN reassignment in SpatialMETA, dropping metabolite spot mappings whose distance exceeds d_min * f_dist (default f_dist=1.5), so that only spatially plausible SM spots are aggregated onto ST spots.

## When to use

Use when SM spots from an adjacent tissue section have been aligned and reassigned to ST spot coordinates via KNN, and mappings beyond the trustworthy spatial distance must be removed before aggregation.

## When NOT to use

- Input data are at single-cell or subcellular resolution — SpatialMETA's alignment and KNN reassignment are designed for low-resolution ST and SM data only
- ST and SM measurements come from the same tissue section already sharing coordinates, so no cross-section reassignment or distance filtering is needed

## Inputs

- Aligned SM spot coordinates in ST space (from spot_transform_by_manual) as spatialmeta.AnnDataSM
- ST spot coordinate grid from spatialmeta.pp.new_spot_sample as spatialmeta.AnnDataST
- KNN parameters: K=5 neighbors and distance scaling factor f_dist (default 1.5)

## Outputs

- Reassigned SM AnnData containing only SM-to-ST assignments within the d_min * f_dist threshold (reassigned_sm.h5ad)
- Aggregated metabolite intensity matrix (MIM) on ST spots (mim_aggregated.tsv)

## How to apply

After applying the learned SM-to-ST affine/diffeomorphic transformation with spatialmeta.pp.spot_transform_by_manual and generating the unified ST spot grid with spatialmeta.pp.new_spot_sample, run spatialmeta.pp.spot_align_byknn with K=5 nearest neighbors. For each ST spot, the function identifies the K nearest SM spots and computes d_min, the minimum SM-to-ST distance; assignments are retained only if their distance satisfies the threshold d_min * f_dist, where f_dist is a scaling factor with a default value of 1.5 that can be user-defined. Assignments exceeding this threshold are dropped, preventing unreliable matches between non-overlapping regions of adjacent sections from contaminating the aggregated metabolite intensity matrix (MIM). The retained assignments are then used to average SM metabolite intensities onto ST spots via spatialmeta.pp.calculate_dot_df, producing reassigned_sm.h5ad and mim_aggregated.tsv.

## Related tools

- **SpatialMETA** (Provides spatialmeta.pp.spot_align_byknn, which performs the KNN reassignment and applies the d_min * f_dist distance threshold to drop distant SM-to-ST matches) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
reassigned_sm = spatialmeta.pp.spot_align_byknn(transformed_sm, st_data, k=5, f_dist=1.5)
```

## Evaluation signals

- Every retained SM-to-ST assignment satisfies the distance criterion: SM spot distance ≤ d_min * 1.5 (or the user-defined f_dist)
- Both output files exist and are well-formed: reassigned_sm.h5ad loads as a valid AnnData and mim_aggregated.tsv parses as a tab-separated table
- No ST spot in the output is matched to SM spots beyond the threshold — spot-to-spot distance distribution should show no outliers above d_min * f_dist
- The number of dropped assignments is consistent with the geometric overlap between the two adjacent sections (regions without SM coverage yield no matches)

## Limitations

- f_dist=1.5 is a default that may not suit all tissue pairs; mis-specified thresholds either over-drop valid matches or retain spatially implausible ones
- Alignment and reassignment are currently applied only between two adjacent sections within a single sample; multi-section alignment for 3D spatial omics is not implemented
- Histological images for SM are often unavailable, so alignment relies on the outline approximated from SM spot coordinates, which can degrade matching accuracy and affect threshold behavior

## Evidence
