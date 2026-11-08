---
name: tic-normalized-lipid-imaging-data-handling
description: Use when you have a massPix-generated normalised image data matrix (image.norm.csv
  with lipid annotations, m/z and TIC-normalised intensities per pixel) and need to
  classify tissue regions by clustering or PCA.
when_to_use_negative:
- Input is raw vendor data not yet processed by massPix — pre-processing, filtering,
  deisotoping, annotation and normalisation must run first (process = T) before clustering.
- You need clustering methods other than k-means — massPix currently only supports
  cluster.type = 'kmeans'.
- The data matrix is not normalised (no TIC/median/standards normalisation applied)
  — unscaled, unnormalised intensities are not comparable across pixels.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: massPix
  role: R package providing the centreScale, cluster and imageSlice functions used
    to scale the normalised image matrix, perform k-means clustering of pixels, and
    plot clustered tissue maps
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix package is installed and invoked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:04f468f291a614e010e8f9195b8962449559f050d34167ab7cbcc233ed044dba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tic-normalized-lipid-imaging-data-handling

## Summary

Loading and downstream multivariate handling of the TIC-normalised, lipid-annotated image matrix (image.norm.csv) produced by the massPix pipeline, including centering/Pareto scaling and k-means clustering of pixels to classify tissue regions with high spectral similarity.

## When to use

Use when you have a massPix-generated normalised image data matrix (image.norm.csv with lipid annotations, m/z and TIC-normalised intensities per pixel) and need to classify tissue regions by clustering or PCA.

## When NOT to use

- Input is raw vendor data not yet processed by massPix — pre-processing, filtering, deisotoping, annotation and normalisation must run first (process = T) before clustering.
- You need clustering methods other than k-means — massPix currently only supports cluster.type = 'kmeans'.
- The data matrix is not normalised (no TIC/median/standards normalisation applied) — unscaled, unnormalised intensities are not comparable across pixels.

## Inputs

- image.norm.csv — full normalised image data matrix with lipid annotations, m/z and TIC-normalised intensities for every pixel (produced by the massPix pipeline from .imzML/.ibd input files)
- image.norm_short.csv — file with lipid annotations and m/z only, used to identify row indices of ions to slice

## Outputs

- R plots of clustered pixels as a 2D tissue map at the image spatial resolution (exportable as PDF)
- R plots of spectra corresponding to the average of each cluster
- Cluster.csv files containing cluster-center (average) spectra

## How to apply

Load image.norm.csv, the full normalised image data matrix containing lipid annotations, m/z and TIC-normalised intensities for every pixel, with the 4 offset metadata columns (annotations and m/z) excluded from the numerical analysis. Apply the massPix centreScale function with scale.type = 'cs' to center and Pareto-scale the pixel intensities, which is required before clustering or PCA. Then call the massPix cluster function with cluster.type = 'kmeans' (the only supported clustering type) and a chosen number of clusters (e.g. clusters = 5), setting res.spatial to the image resolution in micrometers (e.g. 50). Rationale: normalisation to total ion count and Pareto scaling make pixel spectra comparable before grouping pixels of high spectral similarity into tissue regions. The clustered pixels are plotted as a 2D tissue map and the average spectrum per cluster is computed and written out.

## Related tools

- **massPix** (R package providing the centreScale, cluster and imageSlice functions used to scale the normalised image matrix, perform k-means clustering of pixels, and plot clustered tissue maps) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix package is installed and invoked)

## Examples

```
results <- massPix(process=F, pca=F, slice=F, cluster.k=T, cluster.type="kmeans", clusters=5, res.spatial=50, scale.type="cs", norm.type="TIC", offset=4)
```

## Evaluation signals

- The clustered pixel map differentiates the three major cerebellum regions (white matter, granular and molecular layers) as in the paper's Fig. 2d.
- Cluster-center spectra in the Cluster.csv files reflect region-specific lipid profiles for the differentiated tissue regions.
- The 4 offset metadata columns (lipid annotations and m/z) were excluded from scaling (offset = 4), and scale.type = 'cs' (centering + Pareto scaling) was applied.
- The number of clusters matches the clusters parameter (e.g. 5) and one average spectrum per cluster is written to a Cluster.csv file.
- The exported tissue map plot resolution matches the res.spatial setting in micrometers (e.g. 50).

## Limitations

- massPix currently only supports 'kmeans' for cluster.type; other clustering algorithms are not available.
- massPix processes one file at a time.
- Accurate-mass lipid annotations are putative; MS/MS confirmation is required where appropriate.
- Re-running clustering with different settings can be done without re-processing by setting process = F, since image.norm.csv is reused.

## Evidence

- [readme] massPix outputs high quality images, a dataframe of the final normalised and annotated image which can be further manipulated in R, and csv files for spectra corresponding to cluster centers: "massPix outputs high quality images, a dataframe of the final normalised and annotated image which can be further manipulated in R, and csv files for spectra corresponding to cluster centers"
- [readme] normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca): "normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca)"
- [abstract] Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [methods] The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
- [readme] cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files.: "cluster.k - T to perform clustering. Output are R plots of clustered pixels and spectra corresponding to the average of each cluster. In addition spectra are written to Cluster.csv files."
