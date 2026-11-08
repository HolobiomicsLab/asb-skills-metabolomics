---
name: mass-spectrometry-image-linescan-projection
description: Use when you have 2D DESI-MSI hyperspectral images of a tissue with an
  approximate linear developmental gradient (e.g., maize root tip) and need to assign
  mass signatures to developmental enrichment clusters.
when_to_use_negative:
- The tissue lacks a linear developmental axis or the developmental positions are
  not aligned across samples, since projection to a single axis assumes comparable
  longitudinal coordinates.
- You already have a precomputed feature table of signals per sample (no spatial 2D
  ion images to project) — clustering, not linescan projection, is the relevant step.
- Input is raw, unprocessed MSI data; DIMPLE requires the preprocessing/harmonization
  stage to be completed first.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DESI-MSI
  role: Source imaging modality producing the 2D spatial ion-intensity maps of metabolites
    in the developing maize root that are projected to linescans
- name: DIMPLE
  role: Pipeline that processes mass signatures along linear gradients to generate
    developmental enrichment clusters; distributes the .cdf files and Matlab workspaces
    used here
  repo: github.com/dickinsonlab/DIMPLE-code
- name: scikit-learn
  role: Provides AgglomerativeClustering with Ward linkage and Euclidean distance
    (k=4) used to cluster the linescan feature matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:24574379037f704fd8191d1042ca2b4e7af85c4436daba67e1eeea2cea8c4088
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
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mass-spectrometry-image-linescan-projection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Project 2D DESI-MSI ion-intensity images along a root's longitudinal (developmental) axis into 1D linescans, then cluster mass signatures with Ward hierarchical clustering so metabolites with similar developmental enrichment patterns (e.g., meristem/root-tip enrichment in B73 maize roots) group together.

## When to use

Use when you have 2D DESI-MSI hyperspectral images of a tissue with an approximate linear developmental gradient (e.g., maize root tip) and need to assign mass signatures to developmental enrichment clusters.

## When NOT to use

- The tissue lacks a linear developmental axis or the developmental positions are not aligned across samples, since projection to a single axis assumes comparable longitudinal coordinates.
- You already have a precomputed feature table of signals per sample (no spatial 2D ion images to project) — clustering, not linescan projection, is the relevant step.
- Input is raw, unprocessed MSI data; DIMPLE requires the preprocessing/harmonization stage to be completed first.

## Inputs

- Preprocessed harmonized hyperspectral DESI-MSI spectra (NetCDF .cdf files, e.g., B73-root11, B73-root2-5, B73-root-3-3)
- Matlab workspaces distributed with DIMPLE-code (e.g., B73-root11-workspace.mat)
- List of mass signatures of interest (e.g., m/z 149.0118, succinate m/z 117.0195)

## Outputs

- 1D longitudinal linescans per mass signature (normalized, 10-pixel median-filtered)
- Feature matrix (mass signatures x longitudinal positions)
- cluster_assignments.csv mapping each mass signature to one of 4 developmental clusters

## How to apply

Load the preprocessed harmonized hyperspectral spectra (NetCDF .cdf files or the Matlab workspaces distributed with DIMPLE-code, e.g., B73-root11-workspace.mat). For each mass signature, project its 2D spatial ion-intensity image onto the x (developmental) axis by taking the maximum value per position, producing a 1D longitudinal linescan. Normalize each linescan (e.g., to unit sum or TIC-normalized scale) and smooth with a 10-pixel running median filter to reduce pixel-level noise. Assemble the smoothed linescans into a feature matrix (mass signatures x longitudinal positions) and run agglomerative hierarchical clustering with Ward linkage on Euclidean distances, cutting into k=4 clusters as reported for B73. The rationale is that DIMPLE exploits the linear developmental axis of the root so that mass signatures with similar spatial enrichment profiles collapse into interpretable developmental clusters.

## Related tools

- **DESI-MSI** (Source imaging modality producing the 2D spatial ion-intensity maps of metabolites in the developing maize root that are projected to linescans)
- **DIMPLE** (Pipeline that processes mass signatures along linear gradients to generate developmental enrichment clusters; distributes the .cdf files and Matlab workspaces used here) — github.com/dickinsonlab/DIMPLE-code
- **scikit-learn** (Provides AgglomerativeClustering with Ward linkage and Euclidean distance (k=4) used to cluster the linescan feature matrix)

## Examples

```
from sklearn.cluster import AgglomerativeClustering; labels = AgglomerativeClustering(n_clusters=4, linkage='ward', metric='euclidean').fit_predict(linescan_matrix)
```

## Evaluation signals

- Cluster count after the dendrogram cut is exactly 4 (k=4 as reported for B73).
- Cluster 1 contains 48 mass signatures and shows strong meristem/root-tip enrichment.
- m/z 149.0118 is assigned to Cluster 1; succinate (m/z 117.0195) is assigned to Cluster 2.
- Each linescan is unit-sum (or TIC) normalized and smoothed with a 10-pixel running median before clustering.
- Mismatches between obtained assignments and the reported B73 cluster contents are explicitly reported.

## Limitations

- DESI-MSI spatial resolution is only approximately 50-100 micrometers, limiting how finely the longitudinal gradient can be resolved.
- Cluster composition can vary across replicates (e.g., succinate was assigned to Cluster 2 in only two of the replicates).
- The majority of mass signatures remain unannotated, so clusters capture enrichment patterns but not necessarily metabolite identities.
- The pipeline was developed for linear developmental axes (plant roots); nonlinear tissue geometries are outside its design scope.

## Evidence
