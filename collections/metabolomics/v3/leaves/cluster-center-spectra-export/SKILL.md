---
name: cluster-center-spectra-export
description: Use when you have a TIC-normalised annotated pixel matrix (image.norm.csv)
  from a mass spectrometry imaging lipidomics run and need to segment tissue into
  spectrally similar regions and recover their average lipid profiles.
when_to_use_negative:
- You want spatially-aware clustering (massPix clustering is non-spatial k-means;
  tools like Cardinal offer spatially-aware clustering instead)
- You only need single ion images — use the imageSlice function rather than clustering
- You need PCA score/loading images rather than region classification — use centreScale
  + imagePca instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: massPix
  role: R package providing the cluster, centreScale and massPix wrapper functions
    used to k-means cluster pixels and export cluster-center spectra
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which massPix clustering is executed and plots are
    generated/exported
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:d312e2d618d56da5d29a86bc15bd9d472522eccb2e17bdc2807d165cd3d1adf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cluster-center-spectra-export

## Summary

Export k-means clustered pixel maps and average cluster-center spectra from massPix normalised annotated MSI data, classifying tissue regions of high spectral similarity and writing Cluster.csv files for each cluster.

## When to use

Use when you have a TIC-normalised annotated pixel matrix (image.norm.csv) from a mass spectrometry imaging lipidomics run and need to segment tissue into spectrally similar regions and recover their average lipid profiles.

## When NOT to use

- You want spatially-aware clustering (massPix clustering is non-spatial k-means; tools like Cardinal offer spatially-aware clustering instead)
- You only need single ion images — use the imageSlice function rather than clustering
- You need PCA score/loading images rather than region classification — use centreScale + imagePca instead

## Inputs

- image.norm.csv — full normalised annotated image data matrix (lipid annotations, m/z and TIC-normalised intensities per pixel) produced by massPix
- massPix processing parameters (cluster.type, clusters, res.spatial, scale.type)
- Optional: imzML/.ibd input processed first via massPix(process=T)

## Outputs

- 2D clustered pixel map of the tissue at res.spatial resolution, exported as PDF
- Cluster.csv files with average spectra for each cluster center
- R plots of clustered pixels viewable/exportable in R

## How to apply

Load image.norm.csv, the full normalised image data matrix containing lipid annotations, m/z and TIC-normalised intensities for every pixel, produced by the massPix pipeline. Center and Pareto scale the pixel intensities with centreScale (scale.type = 'cs'), skipping the 4 offset metadata columns holding annotations and m/z. Apply k-means via the massPix cluster function (cluster.type = 'kmeans'; currently the only supported clustering type) choosing a cluster count appropriate to the expected tissue architecture (e.g. clusters = 5 for cerebellum with white matter, granular and molecular layers). Plot the clustered pixels as a 2D tissue map at the image spatial resolution (res.spatial, in micrometers, e.g. 50) and export as PDF, then compute each cluster's average spectrum and write the cluster-center spectra to Cluster.csv files. Run clustering via cluster.k = T in the massPix() wrapper after process=T; use process=F when re-clustering already processed data with different cluster settings.

## Related tools

- **massPix** (R package providing the cluster, centreScale and massPix wrapper functions used to k-means cluster pixels and export cluster-center spectra) — https://github.com/hallz/massPix
- **R** (Runtime environment in which massPix clustering is executed and plots are generated/exported)

## Examples

```
results <- massPix(pca=F, slice=F, process=F, cluster.k=T, cluster.type="kmeans", clusters=5, res.spatial=50, scale.type="cs", offset=4)
```

## Evaluation signals

- Clustered pixel map spatially differentiates the expected tissue regions (e.g. white matter, granular and molecular layers of cerebellum as in Fig. 2d)
- Number of Cluster.csv files equals the 'clusters' parameter setting
- Cluster-center spectra reflect region-specific lipid profiles for the differentiated regions
- Clustering was run on centreScale (Pareto-scaled) intensities, not raw intensities, and offset metadata columns (annotations, m/z) were excluded (offset = 4)
- Plot spatial resolution matches the declared res.spatial value in micrometers

## Limitations

- Only 'kmeans' is supported for cluster.type — no hierarchical or spatially-aware clustering options
- Choosing the number of clusters is a user decision; too few clusters will merge distinct tissue layers, too many will over-segment
- massPix processes one imzML file at a time; clustering must be repeated per file
- k-means is non-spatial, so spatially disconnected pixels with similar spectra will be grouped regardless of anatomy

## Evidence

- [abstract] Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [readme] cluster.k description: "cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files."
- [readme] kmeans clustering and PCA scaling description: "or normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and"
- [readme] cluster.type parameter: "cluster.type - Currently only supports "kmeans" clustering"
- [methods] cerebellum regions differentiation: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
