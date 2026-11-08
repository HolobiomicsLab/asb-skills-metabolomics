---
name: pca-loadings-lipid-interpretation
description: Use when mass spectrometry imaging data (imzML) has been extracted, deisotoped,
  annotated and TIC-normalised, and the goal is to separate spatially co-located tissue
  regions (e.g. cerebellum layers) or interpret which lipids drive each principal
  component.
when_to_use_negative:
- The input is raw imzML/ibd spectra that has not yet been extracted, binned, deisotoped,
  annotated and normalised — run the massPix processing workflow (process=T) first
- The goal is metabolite identification confirmation — accurate-mass database hits
  and PCA loadings are only the first step; MS/MS confirmation is required
- A supervised classification or spatially-aware clustering method is needed — massPix
  PCA is unsupervised; alternatives like Cardinal's spatially-aware clustering may
  be more appropriate
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: massPix
  role: R package providing centreScale (centering + Pareto scaling) and imagePca
    (PCA via princomp) for multivariate statistics on normalised MSI data, plus loadings
    export
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment; imagePca wraps R's princomp (cor=FALSE) to compute PC
    scores and loadings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:62619658d74d3964b832f5c43d3041f7ce1c2ef51b0bb02fa16cf52788f10405
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pca-loadings-lipid-interpretation

## Summary

Classifies MSI tissue regions with high spectral similarity by running PCA (via R's princomp through massPix's imagePca) on the normalised, Pareto-scaled pixel-by-lipid-feature matrix, then interprets PC scores images alongside exported PC loadings to identify the lipid species driving each component.

## When to use

Use when mass spectrometry imaging data (imzML) has been extracted, deisotoped, annotated and TIC-normalised, and the goal is to separate spatially co-located tissue regions (e.g. cerebellum layers) or interpret which lipids drive each principal component.

## When NOT to use

- The input is raw imzML/ibd spectra that has not yet been extracted, binned, deisotoped, annotated and normalised — run the massPix processing workflow (process=T) first
- The goal is metabolite identification confirmation — accurate-mass database hits and PCA loadings are only the first step; MS/MS confirmation is required
- A supervised classification or spatially-aware clustering method is needed — massPix PCA is unsupervised; alternatives like Cardinal's spatially-aware clustering may be more appropriate

## Inputs

- image.norm.csv — normalised (TIC) pixel-by-feature MSI image matrix with lipid annotations and m/z values
- Deisotoped, annotated feature table from imzML data (test_POS.imzML/test_POS.ibd from MetaboLights MTBLS487)
- Scaling parameters: scale.type='cs' (centring and Pareto scaling), transform=F

## Outputs

- PC scores images (pdf/R plots) with nlevels=50 colour graduations and res.spatial=50 um spatial resolution
- loadings_PC1.csv through loadings_PC5.csv — PC loadings with corresponding m/z lipid annotations
- Spatial classification of tissue regions with high spectral similarity

## How to apply

After producing image.norm.csv from the massPix workflow (mzextractor, peakpicker.bin with bin.ppm=10 and thres.filter=11, deisotope with ppm=3/no_isotopes=2, annotate, normalise with norm.type='TIC'), center and Pareto scale the matrix with centreScale (scale.type='cs', transform=F for no log transform). Run PCA with massPix's imagePca function, which calls R's princomp (cor=FALSE) with PCnum=5 principal components, producing PC scores images with nlevels=50 colour graduations at res.spatial=50 um. Export the loadings of each principal component together with the corresponding m/z lipid annotations as loadings_PC1.csv through loadings_PC5.csv. Interpret each scores image against tissue anatomy (e.g. white matter, granular and molecular layers of the cerebellum) and match high/low loading m/z values to the annotated lipid species to explain what drives each component.

## Related tools

- **massPix** (R package providing centreScale (centering + Pareto scaling) and imagePca (PCA via princomp) for multivariate statistics on normalised MSI data, plus loadings export) — https://github.com/hallz/massPix
- **R** (Runtime environment; imagePca wraps R's princomp (cor=FALSE) to compute PC scores and loadings)

## Examples

```
results <- massPix(process=F, pca=T, PCnum=5, res.spatial=50, scale.type="cs", transform=F, norm.type="TIC", nlevels=50, spectra_dir=paste(home_dir,"data",sep=""), imzMLparse=paste(home_dir,"imzMLConverter/imzMLConverter.jar",sep=""))
```

## Evaluation signals

- PCA scores image spatially separates tissue regions of high spectral similarity — for the MTBLS487 cerebellum test data, the white matter, granular and molecular layers should be differentiated as reported in Fig. 2c
- loadings_PC1.csv through loadings_PC5.csv exist and each row pairs a m/z with its lipid annotation from the generated lipid library
- Scaling was confirmed as Pareto (scale.type='cs') with no log transform (transform=F) before PCA
- Number of components computed equals PCnum=5, and scores images use nlevels=50 colour graduations at res.spatial=50 um
- Loadings for each PC correspond to annotated lipid species (e.g. PC, PE, SM, CE classes in positive mode) driving that component's spatial pattern

## Limitations

- Accurate-mass lipid annotations should only be considered the first step in metabolite identification; MS/MS confirmation is required
- massPix processes one imzML file at a time and currently only supports kmeans for clustering; PCA parameters (PCnum) must be set by the user
- Lipid classes searched are limited to common classes (DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer in positive mode; PC, PA, PE, PS, PG, PI, FFA in negative mode) with specified adducts — the list is not exhaustive

## Evidence

- [abstract] Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [readme] Single ion images can be produced (function: imageSlice) , or normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca).: "normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca)"
- [readme] In addition the PC loadings are stored as loadingsPC.csv files.: "In addition the PC loadings are stored as loadingsPC.csv files."
- [methods] The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
- [readme] A database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
