---
name: cerebellum-tissue-region-classification
description: Use when you have a TIC-normalised, lipid-annotated MSI pixel matrix
  (e.g. massPix image.norm.csv) and need to segment pixels of high spectral similarity
  into anatomical tissue regions and extract their average spectra.
when_to_use_negative:
- You need non-k-means segmentation — massPix cluster.type currently only supports
  'kmeans' clustering; use a spatially-aware clustering tool such as Cardinal instead.
- The input is raw vendor data not yet pre-processed — deisotoping, annotation and
  TIC normalisation (process=T in massPix) must be run first to obtain image.norm.csv.
- Your goal is lipid identification rather than tissue segmentation — accurate-mass
  annotation against the generated lipid library is a separate massPix step (makelibrary/annotate).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massPix
  role: R package whose centreScale and cluster functions perform centering/Pareto
    scaling and k-means clustering of MSI pixels, and whose plotting outputs the clustered
    tissue map and Cluster.csv spectra
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix clustering function is called, without
    requiring detailed R knowledge
  repo: https://github.com/hallz/massPix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:5b3cb71adf370b619ab850715f2508f4ee3c9d461ba31ef97da321534fcd43b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cerebellum-tissue-region-classification

## Summary

Classifies mass spectrometry imaging pixels into tissue regions using k-means clustering in the massPix R package, producing clustered 2D tissue maps and cluster-center spectra. In the source paper this differentiated the three major cerebellum regions (white matter, granular and molecular layers) by lipid profile.

## When to use

Use when you have a TIC-normalised, lipid-annotated MSI pixel matrix (e.g. massPix image.norm.csv) and need to segment pixels of high spectral similarity into anatomical tissue regions and extract their average spectra.

## When NOT to use

- You need non-k-means segmentation — massPix cluster.type currently only supports 'kmeans' clustering; use a spatially-aware clustering tool such as Cardinal instead.
- The input is raw vendor data not yet pre-processed — deisotoping, annotation and TIC normalisation (process=T in massPix) must be run first to obtain image.norm.csv.
- Your goal is lipid identification rather than tissue segmentation — accurate-mass annotation against the generated lipid library is a separate massPix step (makelibrary/annotate).

## Inputs

- image.norm.csv — full normalised image data matrix with lipid annotations, m/z and TIC-normalised intensities for every pixel (massPix output)
- imzML (.imzML plus .ibd) MSI data processed through the massPix pipeline if starting from raw data
- Number of clusters (clusters) and spatial resolution in micrometers (res.spatial) parameters

## Outputs

- R plots of clustered pixels (2D tissue-region map, exportable as PDF)
- Spectra corresponding to the average of each cluster, written to Cluster.csv files
- Clustered pixel map at the image spatial resolution (e.g. 50 um)

## How to apply

Load the normalised annotated image data matrix image.norm.csv produced by the massPix pipeline (lipid annotations, m/z and TIC-normalised intensities for every pixel). Center and Pareto scale the pixel intensities with centreScale (scale.type = 'cs'), skipping the 4 offset metadata columns holding annotations and m/z. Apply k-means clustering via the massPix cluster function with cluster.type = 'kmeans' and clusters = 5 (k-means is currently the only supported cluster type) to group pixels of high spectral similarity. Plot the clustered pixels as a 2D tissue map at the image spatial resolution (res.spatial = 50 micrometers) and export as PDF. Compute the average spectrum for each cluster and write the cluster-center spectra to Cluster.csv files. Validate by checking that the map separates known anatomical regions — in the paper's cerebellum data, white matter, granular and molecular layers were clearly differentiated by specific lipid profiles.

## Related tools

- **massPix** (R package whose centreScale and cluster functions perform centering/Pareto scaling and k-means clustering of MSI pixels, and whose plotting outputs the clustered tissue map and Cluster.csv spectra) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix clustering function is called, without requiring detailed R knowledge) — https://github.com/hallz/massPix

## Examples

```
results <- massPix(process=F, pca=F, slice=F, cluster.k=T, cluster.type="kmeans", clusters=5, res.spatial=50, norm.type="TIC", scale.type="cs", offset=4)
```

## Evaluation signals

- The clustered pixel map visually differentiates the three major cerebellum regions — white matter, granular and molecular layers — as in the paper's Fig. 2d.
- Cluster.csv contains one average (cluster-center) spectrum per requested cluster (e.g. 5 clusters = 5 center spectra).
- Scaling was applied with scale.type = 'cs' (centering and Pareto scaling) and the 4 offset metadata columns were excluded from clustering.
- Clustering was run on TIC-normalised intensities (norm.type = 'TIC') from image.norm.csv, not on raw unnormalised intensities.
- The exported tissue map was plotted at the correct spatial resolution (res.spatial = 50 micrometers).

## Limitations

- massPix currently only supports 'kmeans' clustering (cluster.type = 'kmeans'); no spatially-aware or NMF-based clustering alternative exists within the package.
- Choosing the number of clusters (e.g. clusters = 5) is a user decision; the mapping of clusters to anatomical regions must be interpreted manually.
- massPix processes one file at a time, and the clustered map depends on upstream normalisation choices (median, TIC, or standard-ion normalisation).

## Evidence

- [abstract] Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [methods] Three cerebellum regions differentiated by lipid profiles: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
- [readme] README on cluster outputs: "cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files."
- [readme] README on k-means from normalised intensities: "normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling"
- [readme] README on supported cluster type: "cluster.type - Currently only supports "kmeans" clustering"
