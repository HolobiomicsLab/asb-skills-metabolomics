---
name: kmeans-image-segmentation
description: Use when an MSI peak matrix must be partitioned into spatial ROIs (e.g.
  brain regions or liver clusters) before running per-cluster differential ion filtering
  such as TestIonSelect.
when_to_use_negative:
- When biological ROIs are already defined manually or via another annotation — the
  article allows passing existing regions of interest instead of clustering.
- When the input is already a segmented results object (e.g. results.Rdata from TestIonSelect)
  rather than a peak matrix requiring segmentation.
- When the input is not spatially resolved MSI pixel data (k-means here operates on
  peak-matrix intensity rows per pixel).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: kmeans (R stats)
  role: Performs unsupervised segmentation of MSI pixels into ROIs via kmeans(peak_matrix$intensity,
    centers = N)$cluster
- name: rMSIproc
  role: Loads the peak matrix from Brain_peak_matrix.zip and visualizes the clustering
    with plotClusterImage
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Part of the rMSI environment installed to visualize ions and cluster images
  repo: https://github.com/prafols/rMSI
- name: rMSIKeyIon
  role: Consumes the clustering vector as the ROI definition for differential ion
    filtering (TestIonSelect)
  repo: https://github.com/LlucSF/rMSIKeyIon
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:0569539474c3e9a8992842ff6a6ba24be2f1f4b1e5bf658b657f6cd175328c8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/prafols/rMSIproc
  tools:
  - R
  - kmeans (R stats)
  - rMSI
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# kmeans-image-segmentation

## Summary

Segments mass spectrometry imaging (MSI) pixel data into regions of interest (ROIs) by running k-means clustering on the peak-matrix intensity columns, enabling spatially resolved differential ion analysis with rMSIKeyIon.

## When to use

Use when an MSI peak matrix must be partitioned into spatial ROIs (e.g. brain regions or liver clusters) before running per-cluster differential ion filtering such as TestIonSelect.

## When NOT to use

- When biological ROIs are already defined manually or via another annotation — the article allows passing existing regions of interest instead of clustering.
- When the input is already a segmented results object (e.g. results.Rdata from TestIonSelect) rather than a peak matrix requiring segmentation.
- When the input is not spatially resolved MSI pixel data (k-means here operates on peak-matrix intensity rows per pixel).

## Inputs

- MSI peak matrix (rMSIproc format, e.g. Brain_peak_matrix.zip loaded via LoadPeakMatrix, or peak_matrix.Rdata)
- Number of clusters (centers), e.g. 7 for the mouse brain section or 6 for the liver images
- Optional pre-computed segmentation (clustering.Rdata storing clustering_vector)

## Outputs

- clustering_vector: integer cluster assignment per pixel suitable for rMSIKeyIon::TestIonSelect(clusters = clustering_vector)
- Visual ROI map via rMSIproc::plotClusterImage for quality control

## How to apply

Load the MSI peak matrix with rMSIproc::LoadPeakMatrix (e.g. Brain_peak_matrix.zip) or by base R load() of peak_matrix.Rdata, then run kmeans on peak_matrix$intensity and take $cluster as the clustering vector. Choose the number of centers according to the study design: centers = 7 for the mouse brain sagittal section, centers = 6 for the six liver images in the thirdhand-smoke study. Alternatively, load a pre-computed segmentation from clustering.Rdata, which stores the vector as clustering_vector. Verify the segmentation spatially by plotting with rMSIproc::plotClusterImage before passing the vector to rMSIKeyIon::TestIonSelect; these ROIs are the comparison groups for the subsequent up/down-regulated ion filtering.

## Related tools

- **kmeans (R stats)** (Performs unsupervised segmentation of MSI pixels into ROIs via kmeans(peak_matrix$intensity, centers = N)$cluster)
- **rMSIproc** (Loads the peak matrix from Brain_peak_matrix.zip and visualizes the clustering with plotClusterImage) — https://github.com/prafols/rMSIproc
- **rMSI** (Part of the rMSI environment installed to visualize ions and cluster images) — https://github.com/prafols/rMSI
- **rMSIKeyIon** (Consumes the clustering vector as the ROI definition for differential ion filtering (TestIonSelect)) — https://github.com/LlucSF/rMSIKeyIon

## Examples

```
clustering_vector <- kmeans(peak_matrix$intensity, centers = 7)$cluster; rMSIproc::plotClusterImage(peakMatrix = peak_matrix, clusters = clustering_vector)
```

## Evaluation signals

- clustering_vector has one cluster label per pixel of the peak matrix and contains exactly the requested number of centers (7 for brain, 6 for liver).
- rMSIproc::plotClusterImage renders spatially coherent, anatomically plausible regions (e.g. distinct brain segments rather than salt-and-pepper noise).
- Downstream TestIonSelect on the segmentation reproduces reference counts: 63 up / 16 down for the brain C2-vs-C6 comparison, and 11 up-regulated ions in C2 with top m/z 835.656 and 806.633.
- The vector matches the distributed clustering.Rdata when regenerated on the demo dataset.
- Cluster identities map to expected biology, e.g. liver C2 capturing lipid-droplet/fatty-liver areas with up-regulated ions in the lipid mass range.

## Limitations

- k-means is unsupervised, so cluster numbers and identities are not guaranteed to correspond to anatomical structures; results depend on the chosen number of centers.
- Downstream ion counts (e.g. 63/16 for C2-vs-C6) depend on reproducing the exact segmentation, so regenerating clusters may not match reference results without the distributed clustering.Rdata.
- Reliability of subsequent statistical comparisons is affected by the elevated number of undetected MS ions in a high percentage of pixels.

## Evidence

- [abstract] ROIs identified by unsupervised segmentation using k-means: "These ROIs were identified by an unsupervised segmentation process, using the popular k-means algorithm."
- [methods] kmeans call in methods: "new_clustering_vector <- kmeans(peak_matrix$intensity, centers = 7)$cluster"
- [readme] Load precomputed clustering: "load("~/path/to/file/clustering.Rdata") # Clustering will be stored in clustering_vector"
- [methods] Visualize clustering: "With rMSIproc you can visualize the clustering and ions easily"
- [intro] Rationale for segmentation: "R package for selecting statistically significant ions between different regions in MSI datasets"
