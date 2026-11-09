---
name: imzml-spectral-data-processing
description: Use when starting from paired .imzML/.ibd MSI files that need thresholded
  ion extraction, m/z binning, deisotoping, lipid annotation, normalisation and scaling
  before downstream imaging or statistics.
when_to_use_negative:
- Data is not in imzML format (convert raw vendor files first using the imzML converter
  instructions)
- Input is already a normalised/annotated feature table — skip to cluster/imageSlice
  with process=F
- Goal requires MS/MS-validated identifications — massPix annotations are accurate-mass-only
  and need confirmation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: massPix
  role: 'R package implementing the full processing chain: mzextractor, peakpicker.bin,
    subsetImage, deisotope, makelibrary, annotate, constructImage, normalise'
  repo: https://github.com/hallz/massPix
- name: imzMLConverter
  role: Java tool parsing imzML spectral data into R via imzMLparse path
  repo: http://www.cs.bham.ac.uk/~ibs/imzMLConverter/#
- name: R
  role: runtime environment; massPix is invoked from R scripting interface via processing_script.R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:a5693090567902c46d18b61369737be4c586a8345f04f97c8078e4bcc639c7c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - imzMLConverter
  - massPix
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# imzml-spectral-data-processing

## Summary

Parse and pre-process .imzML mass spectrometry imaging data into a normalised, deisotoped, lipid-annotated image matrix using the massPix R package. This converts raw MSI pixel spectra into features ready for single ion imaging, PCA and k-means clustering.

## When to use

Use when starting from paired .imzML/.ibd MSI files that need thresholded ion extraction, m/z binning, deisotoping, lipid annotation, normalisation and scaling before downstream imaging or statistics.

## When NOT to use

- Data is not in imzML format (convert raw vendor files first using the imzML converter instructions)
- Input is already a normalised/annotated feature table — skip to cluster/imageSlice with process=F
- Goal requires MS/MS-validated identifications — massPix annotations are accurate-mass-only and need confirmation

## Inputs

- imzML file (e.g. test_POS.imzML, MetaboLights MTBLS487)
- companion .ibd binary data file
- massPix lipid library files in 'libraries/' folder
- imzMLConverter.jar (imzMLConverter/ folder)

## Outputs

- image.norm.csv (full normalised image data matrix: lipid annotations, m/z and intensities)
- image.norm_short.csv (lipid annotations and m/z only, used to find row indices for slicing)
- loadingsPC.csv (PCA loadings, if pca=T)
- Cluster.csv files (cluster center spectra, if cluster.k=T)

## How to apply

Place the paired .imzML and .ibd files in the massPix 'data/' folder alongside 'libraries/' and 'imzMLConverter/imzMLConverter.jar', then run the massPix function via processing_script.R with process=T. Set parsing thresholds (thres.int=100000, thres.low=200, thres.high=1000), binning (bin.ppm=10), missing-value filtering (thres.filter=11), annotation tolerance (ppm.annotate=10, matching against a makelibrary-generated lipid database with adducts like [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive mode), deisotoping (percentage.deiso=3, ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5), normalisation (norm.type='TIC') and scaling (scale.type='cs'). The pipeline runs mzextractor, peakpicker.bin, subsetImage, deisotope, makelibrary/annotate, constructImage and normalise internally. Use process=F on re-runs to reuse processed data when only changing PCA, clustering or slice settings.

## Related tools

- **massPix** (R package implementing the full processing chain: mzextractor, peakpicker.bin, subsetImage, deisotope, makelibrary, annotate, constructImage, normalise) — https://github.com/hallz/massPix
- **imzMLConverter** (Java tool parsing imzML spectral data into R via imzMLparse path) — http://www.cs.bham.ac.uk/~ibs/imzMLConverter/#
- **R** (runtime environment; massPix is invoked from R scripting interface via processing_script.R)

## Examples

```
Rscript -e 'library(massPix); results <- massPix(process=T, ionisation_mode="positive", thres.int=100000, bin.ppm=10, thres.filter=11, ppm.annotate=10, norm.type="TIC", scale.type="cs")'
```

## Evaluation signals

- image.norm.csv and image.norm_short.csv are produced with lipid annotation strings (e.g. [PC(36:1)+K]+) and m/z columns
- Each ion present in image.norm_short.csv corresponds to a row that can be sliced by the imageSlice function
- Annotation tolerance respected: |observed m/z - theoretical m/z| < ppm.annotate (10 ppm)
- Binning tolerance respected: bin.ppm=10 ppm matches suggested mass resolving power 60,000 at m/z 400
- Re-running with process=F after changing only PCA/cluster/slice settings reproduces the same image.norm_short.csv rows

## Limitations

- massPix processes one file at a time
- Accurate-mass database hits are only the first step in metabolite identification; MS/MS confirmation is required
- thres.filter steps range 0–21 (0 = ion present in every pixel, 21 = present in at least one); suggest values between 11 and 15
- bin.ppm must be tuned to instrument resolving power (increase/decrease with lower/higher resolving power)
- deisotoping is computed on a random subset of pixels via subsetImage, so annotation quality depends on that subset

## Evidence

- [readme] The overall data processing workflow consists of initial data pre-processing, filtering, image subsetting, deisotoping, annotation, normalisation, scaling, image "slicing" and multivariate statistics.: "The overall data processing workflow consists of initial data pre-processing, filtering, image subsetting, deisotoping, annotation, normalisation, scaling"
- [readme] deisotoping computed on subset: "Average intensities for all features from a random subset of pixels are computed and used to perform deisotoping (function: subsetImage)."
- [methods] annotation by library search: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [methods] normalisation options: "Ion intensities are then normalised either to the median or total ion count, or to the average intensity of a set of standard ions (function: normalise)"
- [methods] test data location: "Test data is available on the MetaboLights data repository (study ID: MTBLS487)"
