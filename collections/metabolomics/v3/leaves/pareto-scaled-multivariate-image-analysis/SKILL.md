---
name: pareto-scaled-multivariate-image-analysis
description: Use when imzML-derived pixel-by-m/z data (e.g. image.norm.csv with lipid
  annotations, m/z and TIC-normalised intensities) must be classified into tissue
  regions such as cerebellum white matter, granular and molecular layers.
when_to_use_negative:
- Input is raw vendor data not yet parsed and pre-processed by massPix (mzextractor,
  peakpicker.bin, deisotoping, annotate, normalise) — the skill expects the image.norm.csv
  feature table, not spectra
- The goal is putative lipid annotation by accurate mass matching against the generated
  lipid library — that is handled by makelibrary/annotate, not clustering
- You need spatially-aware clustering (e.g. as in Cardinal) — massPix clustering is
  non-spatial and currently supports only k-means
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: massPix
  role: R package providing the centreScale (centering and Pareto scaling), cluster
    (k-means multivariate image) and imagePca functions used for this skill
  repo: https://github.com/hallz/massPix
- name: R
  role: Execution environment; massPix is run from the R scripting interface via processing_script.R
    without requiring detailed R knowledge
  repo: https://github.com/hallz/massPix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:8f528800bf53647689946b53694e6990422ab5c495d9d66deb99f50037456b97
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# pareto-scaled-multivariate-image-analysis

## Summary

Generates multivariate statistical tissue images from mass spectrometry imaging data by centering and Pareto scaling TIC-normalised pixel intensities and applying k-means clustering (or PCA) with massPix, so tissue regions with high spectral similarity are classified as spatially coherent clusters with cluster-center spectra.

## When to use

Use when imzML-derived pixel-by-m/z data (e.g. image.norm.csv with lipid annotations, m/z and TIC-normalised intensities) must be classified into tissue regions such as cerebellum white matter, granular and molecular layers.

## When NOT to use

- Input is raw vendor data not yet parsed and pre-processed by massPix (mzextractor, peakpicker.bin, deisotoping, annotate, normalise) — the skill expects the image.norm.csv feature table, not spectra
- The goal is putative lipid annotation by accurate mass matching against the generated lipid library — that is handled by makelibrary/annotate, not clustering
- You need spatially-aware clustering (e.g. as in Cardinal) — massPix clustering is non-spatial and currently supports only k-means

## Inputs

- image.norm.csv — full normalised image data matrix with lipid annotations, m/z and TIC-normalised intensities for every pixel (from .imzML/.ibd input pairs, e.g. MetaboLights MTBLS487 test data)
- massPix processing parameters (norm.type = 'TIC', scale.type = 'cs', offset = 4, res.spatial = 50, cluster.type = 'kmeans', clusters = 5)

## Outputs

- 2D clustered-pixel tissue map (R plot, exportable as PDF) at the image spatial resolution
- Cluster.csv files containing the average (cluster-center) spectra for each cluster
- loadingsPC.csv files with PC loadings when the PCA branch (imagePca) is used

## How to apply

Load the normalised annotated image data matrix image.norm.csv produced by the massPix pipeline, then center and Pareto scale pixel intensities with the centreScale step (scale.type = 'cs'), skipping the 4 offset metadata columns holding annotations and m/z (offset=4). Apply k-means clustering via the massPix cluster function (cluster.type = 'kmeans', clusters = 5) to group pixels of high spectral similarity, or alternatively compute PCA on the same scaled matrix via imagePca (PCnum = 5). Plot clustered pixels as a 2D tissue map at the image spatial resolution (res.spatial = 50 micrometers) and export as PDF. Compute the average spectrum for each cluster and write the cluster-center spectra to Cluster.csv files (PCA loadings are stored as loadingsPC.csv files). The rationale is that scaling is required before multivariate statistics on intensity data spanning different lipid m/z features, and k-means groups pixels whose TIC-normalised lipid profiles are similar enough to delineate histological regions.

## Related tools

- **massPix** (R package providing the centreScale (centering and Pareto scaling), cluster (k-means multivariate image) and imagePca functions used for this skill) — https://github.com/hallz/massPix
- **R** (Execution environment; massPix is run from the R scripting interface via processing_script.R without requiring detailed R knowledge) — https://github.com/hallz/massPix

## Examples

```
results <- massPix(process=F, pca=T, cluster.k=T, cluster.type="kmeans", clusters=5, PCnum=5, scale.type="cs", norm.type="TIC", offset=4, res.spatial=50)
```

## Evaluation signals

- The clustered pixel map should differentiate the three major cerebellum regions — white matter, granular and molecular layers — as in the paper's Fig. 2d
- Cluster-center spectra written to Cluster.csv should reflect region-specific lipid profiles for each of the 5 clusters
- Scaling step must have used scale.type = 'cs' and offset = 4 so the 4 metadata columns (annotations and m/z) were excluded from scaling and clustering
- image.norm.csv input must be TIC-normalised (norm.type = 'TIC') before scaling; intensities should match the normalised matrix
- Output plots should be rendered at the correct spatial resolution (res.spatial = 50 micrometers) matching the acquisition geometry

## Limitations

- massPix clustering currently only supports the 'kmeans' cluster type; spatially-aware clustering (as offered by Cardinal) and non-negative matrix factorisation (Omnispect) are outside its scope
- massPix processes one file at a time, so multi-slide analyses must be run per imzML file
- Clustering is applied to re-processed data with process=F possible, but results depend on upstream filter threshold (thres.filter ~11–15), bin.ppm (10 ppm) and normalisation choices; accurate-mass database hits still require MS/MS confirmation

## Evidence

- [abstract] Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [methods] The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
- [readme] normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca): "normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca)"
- [readme] cluster.type - Currently only supports "kmeans" clustering: "cluster.type - Currently only supports "kmeans" clustering"
- [readme] cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files.: "cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files."
