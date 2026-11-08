---
name: msi-peak-picked-intensity-analysis
description: Use when segmenting peak-picked MSI pixel-by-feature intensity data (e.g.
  processed imzML loaded into Cardinal) into spatially coherent tissue regions and
  selecting the number of segments by scanning a regularization parameter.
when_to_use_negative:
- Locations must be assigned to pre-defined classes — this is unsupervised segmentation,
  not image classification; use Cardinal's supervised classification workflow instead
- Input is raw, unbinned spectra that still require normalization, smoothing, baseline
  reduction (reduceBaseline), or peak picking (peakPick) before intensities are peak-picked
- Input is already a feature table or per-segment summary — segmentation operates
  on pixel-level peak-picked intensities
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Cardinal
  role: Provides spatialShrunkenCentroids() for regularized image segmentation and
    readMSIData() to import processed imzML
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment in which Cardinal and the segmentation workflow are executed
- name: matter
  role: Backend enabling Cardinal to compute on larger-than-memory MS imaging datasets
    during segmentation
- name: BiocParallel
  role: Parallel processing support for Cardinal pre-processing methods ahead of segmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:aa2ad08e9ce6099dd8ed45a14993fd9f4443586d1d7bae02c77f9a99c312f016
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# msi-peak-picked-intensity-analysis

## Summary

Unsupervised image segmentation of peak-picked mass spectrometry imaging data using Cardinal's spatialShrunkenCentroids with Spatially-Aware (gaussian) distance, which partitions tissue into homogeneous chemical segments without pre-defined class labels.

## When to use

Use when segmenting peak-picked MSI pixel-by-feature intensity data (e.g. processed imzML loaded into Cardinal) into spatially coherent tissue regions and selecting the number of segments by scanning a regularization parameter.

## When NOT to use

- Locations must be assigned to pre-defined classes — this is unsupervised segmentation, not image classification; use Cardinal's supervised classification workflow instead
- Input is raw, unbinned spectra that still require normalization, smoothing, baseline reduction (reduceBaseline), or peak picking (peakPick) before intensities are peak-picked
- Input is already a feature table or per-segment summary — segmentation operates on pixel-level peak-picked intensities

## Inputs

- Peak-picked MS imaging dataset as a processed MSImagingExperiment (e.g. processed imzML read via readMSIData())
- Pig fetus MSI reference data: 28,016 non-background pixels, 10,200 mass features, 298 detected peaks
- Spatial coordinates of pixels for the Spatially-Aware distance

## Outputs

- Per-pixel tissue segment labels (11 segments for the pig fetus reference)
- segmentation_summary.csv with per-segment labels and summary statistics
- Segmentation runtime measurement (reference range 241-827 s)

## How to apply

Load the peak-picked MS imaging data into Cardinal as a processed MSImagingExperiment (imzML via readMSIData()), ensuring spectra have gone through peak detection/summarization steps such as peakPick/peakAlign. Run spatialShrunkenCentroids() on the peak-picked intensities with method='gaussian' (the Spatially-Aware distance), scanning the regularization parameter to compare candidate segmentations. For the reference pig fetus dataset (28,016 non-background pixels, 10,200 mass features, 298 detected peaks), select the solution yielding 11 tissue segments as reported for Figure 1b. Record segmentation runtime (paper reports a 241-827 s range) and export per-segment labels and summary statistics to segmentation_summary.csv. Cardinal's statistical methods are based on mixture modeling and regularization, and larger-than-memory datasets are handled by the matter backend with BiocParallel support for pre-processing.

## Related tools

- **Cardinal** (Provides spatialShrunkenCentroids() for regularized image segmentation and readMSIData() to import processed imzML) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment in which Cardinal and the segmentation workflow are executed)
- **matter** (Backend enabling Cardinal to compute on larger-than-memory MS imaging datasets during segmentation)
- **BiocParallel** (Parallel processing support for Cardinal pre-processing methods ahead of segmentation)

## Examples

```
ssc <- spatialShrunkenCentroids(pig_fetus, method="gaussian", r=seq(1,5)); table(ssc$classes)  # select solution with 11 segments
```

## Evaluation signals

- Segmentation selects exactly 11 tissue segments for the pig fetus peak-picked dataset, matching the Figure 1b result
- segmentation_summary.csv exists, contains per-segment labels covering all 28,016 non-background pixels, and includes summary statistics
- Recorded runtime falls within the reported 241-827 s range
- The regularization parameter scan yields a stable segment count around the selected solution, indicating the segmentation is not an artifact of a single parameter value
- Resulting segments are spatially contiguous regions of homogeneous chemical composition rather than scattered pixel labels

## Limitations

- Cardinal 3 dropped legacy support and introduced breaking changes (e.g. rewritten matter backend), so segmentation behavior may differ across package versions; pin a released version (e.g. v3.6.2) for reproducibility
- Raw spectra in many modern high-resolution MSI experiments cannot be fully loaded into memory; rely on Cardinal/matter's larger-than-memory processing rather than in-memory assumptions
- The number of segments depends on the scanned regularization parameter; the 11-segment solution is dataset-specific and must be re-derived for other tissues or peak-picking settings

## Evidence

- [abstract] Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues.: "Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues."
- [abstract] image segmentation partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions"
- [intro] peakPick() for peak detection and summarization: "`peakPick()` for peak detection and summarization"
- [intro] Parallel processing support via the BiocParallel package for all pre-processing methods: "Parallel processing support via the *BiocParallel* package for all pre-processing methods"
- [readme] While pre-processing steps including normalization, baseline correction, and peak-picking are provided, the core functionality of the package is statistical analysis.: [evidence span withheld — anchor status: external_doc]
- [other] Applying Spatial Shrunken Centroids segmentation with the Spatially-Aware distance to the peak-picked pig fetus data selects 11 tissue segments.: [evidence span withheld — anchor status: not_found]
