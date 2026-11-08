---
name: kmeans-pixel-clustering-msi
description: Use when you have an imzML MSI dataset processed to a normalised annotated
  pixel-by-m/z matrix (image.norm.csv) and need to segment tissue into regions based
  on lipid spectral similarity, e.g. differentiating brain layers.
when_to_use_negative:
- The input is raw spectra not yet parsed, filtered, deisotoped, annotated and normalised
  by massPix — clustering requires image.norm.csv as input.
- You need spatially-aware clustering or non-negative matrix factorization — the article
  notes Cardinal (spatially-aware) and Omnispect (NMF) serve those approaches; massPix
  only supports 'kmeans' clustering.
- The goal is single-ion localisation rather than multi-feature region classification
  — use the imageSlice function instead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: massPix
  role: R package whose cluster function (with centreScale) performs k-means clustering
    of MSI pixels and generates the clustered tissue map and cluster-center spectra
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment; massPix is run from the R scripting interface via processing_script.R
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:37470cc92cc702181b74b9bda41b5706c303646e962e73908cf338ba0593c0a3
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# kmeans-pixel-clustering-msi

## Summary

Classify mass spectrometry imaging pixels into tissue regions of high spectral similarity using massPix's k-means clustering on TIC-normalised, Pareto-scaled lipid intensity data, producing a clustered tissue map and cluster-center spectra.

## When to use

Use when you have an imzML MSI dataset processed to a normalised annotated pixel-by-m/z matrix (image.norm.csv) and need to segment tissue into regions based on lipid spectral similarity, e.g. differentiating brain layers.

## When NOT to use

- The input is raw spectra not yet parsed, filtered, deisotoped, annotated and normalised by massPix — clustering requires image.norm.csv as input.
- You need spatially-aware clustering or non-negative matrix factorization — the article notes Cardinal (spatially-aware) and Omnispect (NMF) serve those approaches; massPix only supports 'kmeans' clustering.
- The goal is single-ion localisation rather than multi-feature region classification — use the imageSlice function instead.

## Inputs

- image.norm.csv (full normalised image data matrix with lipid annotations, m/z and TIC-normalised intensities for every pixel)
- imzML/ibd MSI data files (e.g. test_POS.imzML and test_POS.ibd from MetaboLights MTBLS487) pre-processed by massPix
- k-means parameters: cluster.type='kmeans', clusters (e.g. 5), scale.type='cs'

## Outputs

- PDF plot of clustered pixels as a 2D tissue map at the image spatial resolution (res.spatial=50 micrometers)
- Cluster.csv files containing average spectra for each cluster center
- R plots of clustered pixels viewable/exported as pdf

## How to apply

Load the normalised annotated image matrix image.norm.csv produced by the massPix pipeline (run with e.g. thres.int=100000, bin.ppm=10, thres.filter=11, norm.type='TIC'), skipping the 4 offset metadata columns holding annotations and m/z. Apply centering and Pareto scaling via centreScale (scale.type='cs'), then call the massPix cluster function with cluster.type='kmeans' and clusters=5 to group pixels of high spectral similarity. Plot the clustered pixels as a 2D tissue map at res.spatial=50 micrometers and export as PDF; write the average spectrum of each cluster to Cluster.csv files. Rationale: k-means on scaled intensities groups pixels whose lipid profiles are similar, so cluster boundaries correspond to histologically distinct regions (e.g. cerebellar white matter, granular and molecular layers).

## Related tools

- **massPix** (R package whose cluster function (with centreScale) performs k-means clustering of MSI pixels and generates the clustered tissue map and cluster-center spectra) — https://github.com/hallz/massPix
- **R** (Runtime environment; massPix is run from the R scripting interface via processing_script.R)

## Examples

```
library(massPix); results <- massPix(process=T, pca=T, slice=T, cluster.k=T, ionisation_mode="positive", norm.type="TIC", scale.type="cs", cluster.type="kmeans", clusters=5, res.spatial=50)
```

## Evaluation signals

- The clustered pixel map differentiates the three major cerebellum regions (white matter, granular and molecular layers) as reported in the paper's Fig. 2d.
- Cluster-center spectra in Cluster.csv reflect the region-specific lipid profiles of the differentiated tissue regions.
- Clustering was run on centreScale (Pareto-scaled, scale.type='cs') normalised intensities, not on raw or unscaled intensities — verify the 4 offset annotation/m/z columns were excluded.
- Output PDF of clustered pixels exists and pixel assignment count matches the chosen clusters value (e.g. 5).

## Limitations

- massPix currently supports only 'kmeans' clustering (cluster.type parameter), so other clustering algorithms are not available without adapting the source code.
- massPix processes one imzML file at a time.
- Cluster number must be chosen by the user (e.g. clusters=5 for the cerebellum test data); no automatic selection of cluster count is described.
- In contrast to freely available tools such as Cardinal, massPix does not perform spatially-aware clustering.

## Evidence

- [abstract] Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [readme] normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling: "or normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling"
- [readme] cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files.: "cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files."
- [methods] The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
- [readme] Currently only supports "kmeans" clustering: "cluster.type - Currently only supports "kmeans" clustering"
