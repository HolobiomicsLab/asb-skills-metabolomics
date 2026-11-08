---
name: longitudinal-gradient-normalization
description: Use when 2D DESI-MSI ion-intensity images of a linear developmental axis
  (e.g., maize root tip-to-mature gradient) must be projected to normalized 1D linescans
  and clustered by enrichment pattern.
when_to_use_negative:
- The tissue or gradient has no linear developmental axis — DIMPLE is explicitly designed
  to 'leverage the linear developmental axis', so radial or non-polarized spatial
  structures are the wrong input.
- The input is already a matrix of 1D enrichment profiles or a feature table; then
  only the Ward clustering step, not the 2D-to-1D projection and normalization, is
  needed.
- The goal is metabolite annotation of individual m/z peaks (e.g. via METASPACE, Metlin
  Classic, or LOTUS) rather than grouping signatures by developmental enrichment pattern.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: DIMPLE
  role: Pipeline performing the 2D-to-1D projection, normalization, median smoothing,
    and Ward clustering of mass signatures along the linear developmental axis
  repo: https://github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality that produces the 2D spatial ion-intensity maps of the developing
    maize root that this skill normalizes and clusters
- name: scikit-learn
  role: Provides agglomerative hierarchical clustering with Ward linkage and Euclidean
    distance used to cut the linescan feature matrix into k=4 clusters
- name: batchcdfread
  role: Matlab function (developed by Yifan Meng, Zare lab, Stanford) for reading
    the .cdf DESI-MSI files into workspaces before projection
  repo: https://github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:1d68ca36554b0aeff1bac5523ec35d4a8e07e548953f7f39db413bde66722aed
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  repo_url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - batchcdfread
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# longitudinal-gradient-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

DIMPLE's normalization scheme that reduces 2D DESI-MSI spatial ion-intensity images to 1D longitudinal linescans along the root developmental axis, normalizes them (unit-sum/TIC), smooths with a 10-pixel running median, and feeds the resulting feature matrix into Ward hierarchical clustering to recover developmental enrichment patterns in maize roots.

## When to use

Use when 2D DESI-MSI ion-intensity images of a linear developmental axis (e.g., maize root tip-to-mature gradient) must be projected to normalized 1D linescans and clustered by enrichment pattern.

## When NOT to use

- The tissue or gradient has no linear developmental axis — DIMPLE is explicitly designed to 'leverage the linear developmental axis', so radial or non-polarized spatial structures are the wrong input.
- The input is already a matrix of 1D enrichment profiles or a feature table; then only the Ward clustering step, not the 2D-to-1D projection and normalization, is needed.
- The goal is metabolite annotation of individual m/z peaks (e.g. via METASPACE, Metlin Classic, or LOTUS) rather than grouping signatures by developmental enrichment pattern.

## Inputs

- Preprocessed harmonized hyperspectral DESI-MSI spectra (2D spatial ion-intensity images per mass signature)
- NetCDF (.cdf) raw files, e.g. B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0
- Matlab workspaces, e.g. B73-root11-workspace.mat, OG1-workspace-2.mat

## Outputs

- cluster_assignments.csv mapping each mass signature to one of k=4 developmental-enrichment clusters
- Normalized, median-smoothed 1D longitudinal linescans per mass signature
- Cluster membership counts (e.g. 48 signatures in Cluster 1 for B73)

## How to apply

Load the preprocessed harmonized hyperspectral spectra produced by the DIMPLE preprocessing stage (NetCDF .cdf files or the provided Matlab workspaces, e.g. B73-root11-workspace.mat). For each mass signature, project the 2D spatial ion-intensity image along the x (developmental) axis to a 1D longitudinal linescan by taking the maximum value per position. Normalize each linescan (unit-sum or TIC-normalized scale) and apply a 10-pixel running median filter to reduce noise before assembling the mass-signatures-by-longitudinal-position feature matrix. Cluster the mass signatures with agglomerative hierarchical clustering using Ward linkage on Euclidean distances, cut into k=4 clusters as reported for B73, and emit an assignment table (cluster_assignments.csv). The rationale is that the linear developmental axis of the root lets each m/z be summarized as a position-enrichment profile, so metabolites with matching meristem vs. mature-zone patterns fall into the same cluster.

## Related tools

- **DIMPLE** (Pipeline performing the 2D-to-1D projection, normalization, median smoothing, and Ward clustering of mass signatures along the linear developmental axis) — https://github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality that produces the 2D spatial ion-intensity maps of the developing maize root that this skill normalizes and clusters)
- **scikit-learn** (Provides agglomerative hierarchical clustering with Ward linkage and Euclidean distance used to cut the linescan feature matrix into k=4 clusters)
- **batchcdfread** (Matlab function (developed by Yifan Meng, Zare lab, Stanford) for reading the .cdf DESI-MSI files into workspaces before projection) — https://github.com/dickinsonlab/DIMPLE-code

## Examples

```
from sklearn.cluster import AgglomerativeClustering; labels = AgglomerativeClustering(n_clusters=4, linkage='ward').fit_predict(linescan_matrix)
```

## Evaluation signals

- The cluster cut yields exactly k=4 clusters.
- Cluster 1 contains 48 mass signatures with strong meristem/root-tip enrichment in B73.
- Succinate (m/z 117.0195) is assigned to Cluster 2.
- m/z 149.0118 is assigned to Cluster 1, matching the reported B73 cluster contents.
- Assignments are consistent across B73 replicates (note: succinate fell in Cluster 2 in two of the replicates, so cross-replicate agreement is the criterion, not perfect identity).

## Limitations

- Cluster assignments can vary between replicates — succinate was assigned to Cluster 2 in only two of the B73 replicates.
- Requires data collected along a linear developmental axis; the 1D maximum-projection discards lateral (non-longitudinal) spatial information.
- Most resulting mass signatures remain unannotated; the paper notes 'the majority of metabolites in both plants and animals remain unannotated.'

## Evidence

- [methods] Project the 2D spatial ion-intensity image for each mass signature along the x (developmental) axis to a 1D longitudinal linescan, taking the maximum value per position.: [evidence span withheld — anchor status: not_found]
- [methods] Normalize each linescan (e.g. to unit sum or TIC-normalized scale) and apply a 10-pixel running median filter to reduce noise.: [evidence span withheld — anchor status: not_found]
