---
name: running-median-spatial-smoothing
description: Use when per-pixel DESI-MSI ion intensities projected to a 1D longitudinal
  linescan are noisy and need smoothing before clustering or gradient-based enrichment
  analysis of mass signatures.
when_to_use_negative:
- Input is already a smoothed or low-noise 1D profile (e.g. averaged biological replicates)
  where additional filtering would blur genuine enrichment transitions
- Data are 2D spatial images that have not yet been projected to a 1D longitudinal
  linescan — the filter as specified operates on the projected axis only
- Analyzing tissues without a linear developmental axis, where linescan projection
  along a single axis is not meaningful (DIMPLE is designed for linear gradients)
edam_operation: http://edamontology.org/operation_3804
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DESI-MSI
  role: Source modality producing the per-pixel ion-intensity images that are projected
    to 1D linescans and then median-filtered
- name: DIMPLE
  role: Pipeline implementing linescan projection, normalization, and the 10-pixel
    running median filter prior to clustering
  repo: github.com/dickinsonlab/DIMPLE-code
- name: scikit-learn
  role: Consumes the smoothed feature matrix for Ward-linkage agglomerative clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:6f402909a9d11bfd0dfae5d47a8cba091bcf230126c6e6c0e5bc0dc1ae128aac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3804
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# running-median-spatial-smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A denoising filter step in the DIMPLE workflow that reduces pixel-level noise in 1D longitudinal linescans of DESI-MSI ion intensities before hierarchical clustering of mass signatures along the maize root developmental axis.

## When to use

Use when per-pixel DESI-MSI ion intensities projected to a 1D longitudinal linescan are noisy and need smoothing before clustering or gradient-based enrichment analysis of mass signatures.

## When NOT to use

- Input is already a smoothed or low-noise 1D profile (e.g. averaged biological replicates) where additional filtering would blur genuine enrichment transitions
- Data are 2D spatial images that have not yet been projected to a 1D longitudinal linescan — the filter as specified operates on the projected axis only
- Analyzing tissues without a linear developmental axis, where linescan projection along a single axis is not meaningful (DIMPLE is designed for linear gradients)

## Inputs

- Preprocessed harmonized hyperspectral DESI-MSI spectra (NetCDF/cdf files or Matlab workspaces, e.g. B73-root11, B73-root2-5, B73-root-3-3)
- 2D spatial ion-intensity images per mass signature
- Linescan orientation along the root's longitudinal (developmental) axis

## Outputs

- Smoothed 1D longitudinal linescans per mass signature
- Feature matrix (mass signatures x longitudinal positions) ready for Ward agglomerative clustering
- cluster_assignments.csv of mass-signature cluster memberships

## How to apply

First project each 2D spatial ion-intensity image for a mass signature along the x (developmental) axis to a 1D linescan by taking the maximum value per position. Normalize the linescan (e.g. to unit sum or TIC-normalized scale), then apply a 10-pixel running median filter to suppress pixel-to-pixel noise while preserving the shape of enrichment peaks along the root's linear developmental gradient. Assemble the smoothed linescans into a mass-signature x longitudinal-position feature matrix for downstream Ward hierarchical clustering (k=4 for B73). A median (rather than mean) filter is chosen because it is robust to isolated high-intensity outliers typical of DESI-MSI pixel noise.

## Related tools

- **DESI-MSI** (Source modality producing the per-pixel ion-intensity images that are projected to 1D linescans and then median-filtered)
- **DIMPLE** (Pipeline implementing linescan projection, normalization, and the 10-pixel running median filter prior to clustering) — github.com/dickinsonlab/DIMPLE-code
- **scikit-learn** (Consumes the smoothed feature matrix for Ward-linkage agglomerative clustering)

## Examples

```
from scipy.ndimage import median_filter; smooth = median_filter(linescan, size=10)  # 10-pixel running median along the developmental axis
```

## Evaluation signals

- Cluster count after smoothing plus clustering is exactly 4 for B73 (k=4)
- Cluster 1 contains 48 mass signatures with strong meristem/root-tip enrichment; m/z 149.0118 assigned to Cluster 1
- Succinate (m/z 117.0195) is assigned to Cluster 2 in at least two replicates; report any mismatches
- Smoothed linescans show reduced pixel-level variance but retain the enrichment peak positions along the developmental axis
- Results are consistent across the three B73 replicates (B73-root11, B73-root2-5, B73-root-3-3)

## Limitations

- Smoothing parameters (10-pixel window) were tuned for root linescans; other tissues or spatial resolutions may require re-tuning
- DESI-MSI spatial resolution is approximately 50-100 micrometers, so fine structural detail below that scale is not recoverable after filtering
- Reproducibility depends on consistent root orientation along the developmental axis during projection; misalignment shifts enrichment peaks

## Evidence

- [methods] Normalize each linescan (e.g. to unit sum or TIC-normalized scale) and apply a 10-pixel running median filter to reduce noise.: [evidence span withheld — anchor status: not_found]
- [methods] Project the 2D spatial ion-intensity image for each mass signature along the x (developmental) axis to a 1D longitudinal linescan, taking the maximum value per position.: [evidence span withheld — anchor status: not_found]
