---
name: pca-based-tissue-region-classification
description: Use when you have a preprocessed, TIC-normalised MSI feature matrix (imzML-derived
  lipid intensities) and want to spatially separate tissue regions with high spectral
  similarity via principal components analysis and interpret driving lipids from PC
  loadings.
when_to_use_negative:
- You only need single ion images for specific m/z features — use massPix's imageSlice
  (image 'slicing') instead of PCA.
- You need non-negative-matrix-factorisation or spatially-aware clustering rather
  than PCA — tools such as Omnispect or Cardinal provide those alternative multivariate
  approaches.
- Your goal is lipid identification confidence rather than region classification —
  accurate-mass annotation hits still require MS/MS confirmation and PCA loadings
  alone do not validate identifications.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: massPix
  role: R package providing centreScale (centering and Pareto scaling), imagePca (PCA
    via princomp) and PC loadings export for MSI tissue-region classification
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which massPix is run; PCA is computed with R's princomp
    (cor=FALSE)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:09bae311ad358f087f6dabdbc6d0f8fea97c334400818fb516558b0e7bc3671e
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pca-based-tissue-region-classification

## Summary

Classifies mass spectrometry imaging (MSI) pixels into tissue regions with high spectral similarity — e.g. white matter, granular and molecular layers of the cerebellum in MetaboLights MTBLS487 — by running PCA on a normalised, centred and Pareto-scaled pixel-by-feature lipid intensity matrix using massPix's imagePca (R princomp).

## When to use

Use when you have a preprocessed, TIC-normalised MSI feature matrix (imzML-derived lipid intensities) and want to spatially separate tissue regions with high spectral similarity via principal components analysis and interpret driving lipids from PC loadings.

## When NOT to use

- You only need single ion images for specific m/z features — use massPix's imageSlice (image 'slicing') instead of PCA.
- You need non-negative-matrix-factorisation or spatially-aware clustering rather than PCA — tools such as Omnispect or Cardinal provide those alternative multivariate approaches.
- Your goal is lipid identification confidence rather than region classification — accurate-mass annotation hits still require MS/MS confirmation and PCA loadings alone do not validate identifications.

## Inputs

- imzML mass spectrometry imaging data (test_POS.imzML plus test_POS.ibd from MetaboLights study MTBLS487) in the massPix 'data/' folder
- Normalised pixel-by-feature intensity matrix image.norm.csv produced by the massPix processing workflow

## Outputs

- PC scores images (R plots exportable as pdf) showing spatial separation of tissue regions with nlevels=50 colour graduations at 50 um spatial resolution
- loadings_PC1.csv through loadings_PC5.csv containing each principal component's loadings with corresponding m/z and lipid annotations

## How to apply

First preprocess the imzML data with massPix: parse via the bundled imzMLConverter jar, extract and bin ions (mzextractor and peakpicker.bin with thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10, thres.filter=11), deisotope (ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5), annotate by accurate mass, and normalise to total ion count (norm.type='TIC') to obtain image.norm.csv. Then center and Pareto scale the matrix with centreScale (scale.type='cs', no log transform, transform=F), because PCA is sensitive to differences in feature variance. Run imagePca with PCnum=5 principal components using R's princomp (cor=FALSE), rendering PC scores images with nlevels=50 colour graduations at res.spatial=50 um. Export the per-component loadings with lipid m/z annotations as loadings_PC1.csv through loadings_PC5.csv to identify which lipid species drive each component's spatial separation.

## Related tools

- **massPix** (R package providing centreScale (centering and Pareto scaling), imagePca (PCA via princomp) and PC loadings export for MSI tissue-region classification) — https://github.com/hallz/massPix
- **R** (Runtime environment in which massPix is run; PCA is computed with R's princomp (cor=FALSE))

## Examples

```
results <- massPix(process=T, pca=T, ionisation_mode="positive", thres.int=100000, bin.ppm=10, thres.filter=11, PCnum=5, norm.type="TIC", scale.type="cs", transform=F, res.spatial=50, nlevels=50)
```

## Evaluation signals

- The PCA scores image spatially separates the white matter, granular and molecular layers of the cerebellum as reported in Fig. 2c of the paper.
- loadings_PC1.csv through loadings_PC5.csv exist and pair each PC's loadings with m/z and putative lipid annotations.
- Scaling was applied as centre and Pareto (scale.type='cs', transform=F) on the TIC-normalised image.norm.csv matrix, matching the workflow's centreScale step.
- PCnum=5 principal components were computed and the scores image rendered with nlevels=50 and res.spatial=50 um.
- The input feature matrix passed the expected preprocessing thresholds: thres.int=100000, bin.ppm=10, thres.filter=11, deisotoping with ppm=3.

## Limitations

- massPix currently processes one file at a time and only supports .imzML input files; other raw formats must first be converted to imzML.
- Accuracy-mass database hits from the annotation step feeding the loadings are only a first step in metabolite identification; confirmation by MS/MS is required where appropriate.
- PCA is one of several classification options — massPix also supports k-means clustering, and alternative tools (Omnispect, Cardinal) use non-negative matrix factorization or spatially-aware clustering.

## Evidence

- [abstract] Classification of tissue regions with high spectral similarity can be carried out by principal components analysis (PCA) or k-means clustering.: "Classification of tissue regions with high spectral similarly can be carried out by principal components analysis (PCA) or k-means clustering."
- [readme] PCA follows centering and Pareto scaling: "normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and"
- [methods] Three cerebellum regions differentiated by lipid profiles: "The three major tissue regions within the cerebellum - white matter, granular and molecular layers (Fig. 2a)—were clearly differentiated by specific lipid profiles"
- [readme] PC loadings stored as csv files: "output are R plots of PC scores and loadings which can be view or exported as pdf. In addition the PC loadings are stored as loadingsPC.csv files."
- [methods] Test data on MetaboLights MTBLS487: "Test data is available on the MetaboLights data repository (study ID: MTBLS487)"
