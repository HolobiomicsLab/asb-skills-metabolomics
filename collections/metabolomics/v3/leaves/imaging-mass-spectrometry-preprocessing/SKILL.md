---
name: imaging-mass-spectrometry-preprocessing
description: Use when raw MSI data in .imzML/.ibd format (e.g. test_POS files from
  MetaboLights MTBLS487) must be converted into a normalised, deisotoped, lipid-annotated
  feature matrix before PCA, k-means clustering, or single-ion image generation.
when_to_use_negative:
- The input is already a normalised feature table (image.norm.csv) and you only need
  downstream PCA, clustering or ion slicing — use process=F instead of re-preprocessing
- Data are not in imzML format and cannot readily be converted — massPix supports
  .imzML files only
- The analytes of interest are outside massPix's covered lipid classes (DAG, TAG,
  PC, PE, PS, PG, PI, PA, LysoPC, CE, SM, Cer, FFA) and their anticipated adducts
edam_operation: http://edamontology.org/operation_3634
tools:
- name: massPix
  role: R package providing the preprocessing functions mzextractor, peakpicker.bin,
    subsetImage, filter, deisotope, makelibrary, annotate, constructImage and normalise
  repo: https://github.com/hallz/massPix
- name: R
  role: Execution environment; massPix is called via processing_script.R and does
    not require detailed R knowledge
  repo: https://github.com/hallz/massPix
- name: imzMLConverter
  role: Java tool (jar) used by massPix to parse .imzML/.ibd spectral data into R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:02afa6f666774a930c787cda4a6984194e47b3c5610d3938ef6c4e06a5474fe3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - imzMLConverter
  - massPix
  edam_operation: http://edamontology.org/operation_3634
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# imaging-mass-spectrometry-preprocessing

## Summary

Preprocessing of imzML mass spectrometry imaging data into a normalised, lipid-annotated pixel-by-feature matrix using the massPix R package: intensity/mz thresholding, 10 ppm mass binning, filtering of sparse features, deisotoping, accurate-mass lipid annotation, and TIC normalisation.

## When to use

Use when raw MSI data in .imzML/.ibd format (e.g. test_POS files from MetaboLights MTBLS487) must be converted into a normalised, deisotoped, lipid-annotated feature matrix before PCA, k-means clustering, or single-ion image generation.

## When NOT to use

- The input is already a normalised feature table (image.norm.csv) and you only need downstream PCA, clustering or ion slicing — use process=F instead of re-preprocessing
- Data are not in imzML format and cannot readily be converted — massPix supports .imzML files only
- The analytes of interest are outside massPix's covered lipid classes (DAG, TAG, PC, PE, PS, PG, PI, PA, LysoPC, CE, SM, Cer, FFA) and their anticipated adducts

## Inputs

- .imzML and .ibd file pair in the data/ folder (e.g. test_POS.imzML / test_POS.ibd from MetaboLights MTBLS487)
- imzMLConverter jar in an imzMLConverter/ subfolder of the working directory
- Lipid library files in the libraries/ folder for makelibrary/annotate
- Processing parameters set in processing_script.R (thres.int, thres.low, thres.high, bin.ppm, thres.filter, ppm, no_isotopes, prop.1, prop.2, norm.type)

## Outputs

- image.norm.csv — full normalised image data matrix with lipid annotations, m/z and intensities
- image.norm_short.csv — lipid annotations and m/z only (row indices used to select ions for imageSlice)
- Image-ready pixel-by-feature matrix for downstream centreScale/imagePca, cluster or imageSlice

## How to apply

Parse the imzML file to R via the bundled imzMLConverter jar, then run massPix with process=T: extract ions above an intensity threshold (thres.int=100000) within m/z 200–1000 (thres.low/thres.high), bin across pixels at bin.ppm=10 (adjust with mass resolving power, e.g. 60,000 at m/z 400), and retain features present above a proportion threshold (thres.filter=11, on the 0–21 missing-value scale). Deisotope a random-pixel-subset image (ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5) to remove M+1/M+2 13C isotopes, then rebuild the full image (constructImage). Annotate deisotoped ions against a generated lipid library (makelibrary/annotate) at ppm.annotate=10, matching lipid classes and adducts appropriate to the ionisation mode. Finally normalise intensities to total ion count (norm.type='TIC', function normalise) to produce image.norm.csv; this matrix is the input for centreScale + imagePca, cluster, or imageSlice.

## Related tools

- **massPix** (R package providing the preprocessing functions mzextractor, peakpicker.bin, subsetImage, filter, deisotope, makelibrary, annotate, constructImage and normalise) — https://github.com/hallz/massPix
- **R** (Execution environment; massPix is called via processing_script.R and does not require detailed R knowledge) — https://github.com/hallz/massPix
- **imzMLConverter** (Java tool (jar) used by massPix to parse .imzML/.ibd spectral data into R)

## Examples

```
results <- massPix(process=T, pca=F, slice=F, cluster.k=F, ionisation_mode="positive", thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10, thres.filter=11, ppm.annotate=10, ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5, norm.type="TIC")
```

## Evaluation signals

- The pipeline completes and writes both image.norm.csv and image.norm_short.csv with lipid annotations, m/z and intensities
- Retained features were detected in at least the thres.filter=11 proportion-of-pixels threshold (scale 0–21), i.e. sparse features removed
- Isotope peaks at M+1/M+2 within the ppm=3, prop.1=0.9 / prop.2=0.5 intensity-proportion criteria are removed from the deisotoped feature list
- All annotations satisfy |observed m/z − theoretical m/z| < ppm.annotate (10 ppm, matching bin.ppm=10)
- Normalised intensities in image.norm.csv are TIC-scaled and downstream functions (imagePca, cluster, imageSlice) run directly on it without further scaling

## Limitations

- massPix processes one file at a time and currently supports only k-means (cluster.type='kmeans') for clustering
- thres.filter should be tuned between 11 and 15; too low retains noise features, too high discards ions
- bin.ppm must be matched to instrument resolving power (e.g. 10 ppm at 60,000 resolving power at m/z 400); higher resolving power permits narrower bins
- Accurate-mass library hits are only the first step in metabolite identification — MS/MS confirmation is required where appropriate

## Evidence

- [readme] Ions with intensities greater than a threshold, from each spectra, are extracted and grouped to user-adjustable mass bins (function: mzextractor): "Ions with intensities greater than a threshold, from each spectra, are extracted and grouped to user-adjustable mass bins (function: mzextractor)"
- [readme] Ion intensities are then normalised either to the median or total ion count, or to the average intensity of a set of standard ions: "Ion intensities are then normalised either to the median or total ion count, or to the average intensity of a set of standard ions (function: normalise)"
- [methods] initial data pre-processing, filtering, image subsetting, deisotoping, annotation, normalisation, scaling, image "slicing" and multivariate statistics: "initial data pre-processing, filtering, image subsetting, deisotoping, annotation, normalisation, scaling, image "slicing" and multivariate statistics"
- [readme] massPix processes high resolution mass spectrometry imaging data, performs multivariate statistics (PCA, clustering) and lipid identification.: "massPix processes high resolution mass spectrometry imaging data, performs multivariate statistics (PCA, clustering) and lipid identification."
