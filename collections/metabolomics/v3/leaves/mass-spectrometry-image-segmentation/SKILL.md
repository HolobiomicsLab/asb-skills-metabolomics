---
name: mass-spectrometry-image-segmentation
description: Use when a peak-picked MSI dataset (e.g., imzML loaded into Cardinal
  as a processed MSImagingExperiment) must be partitioned into chemically homogeneous
  tissue segments without class labels, and the expected number of segments is known
  from a reference figure or must be selected by scanning.
when_to_use_negative:
- The goal is classification to pre-defined class labels rather than unsupervised
  segmentation — use Cardinal's image classification (which assigns locations to pre-defined
  classes and estimates error by cross-validation) instead.
- The input spectra are not yet peak-picked/pre-processed; first apply Cardinal's
  pre-processing pipeline (normalize, smooth, reduceBaseline, peakPick/peakAlign/peakProcess)
  before segmenting.
- Raw spectra exceed memory and the data have not been prepared with Cardinal's larger-than-memory
  matter backend.
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cardinal
  role: provides spatialShrunkenCentroids() segmentation with Spatially-Aware (gaussian)
    distance and the MSImagingExperiment data container for MSI segmentation
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: language in which the segmentation workflow is executed
- name: matter
  role: backend enabling Cardinal to compute on larger-than-memory MS imaging datasets
    during segmentation
- name: BiocParallel
  role: parallel processing support for Cardinal methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:5bce98e1ef5a062a56abd40aacd104ea20150024437ec8473da39e1d32ca928d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mass-spectrometry-image-segmentation

## Summary

Unsupervised segmentation of mass spectrometry imaging (MSI) pixels into tissue segments using Cardinal's Spatial Shrunken Centroids with a Spatially-Aware ('gaussian') distance, demonstrated on peak-picked pig fetus data (28,016 non-background pixels, 10,200 mass features, 298 detected peaks) yielding 11 tissue segments.

## When to use

Use when a peak-picked MSI dataset (e.g., imzML loaded into Cardinal as a processed MSImagingExperiment) must be partitioned into chemically homogeneous tissue segments without class labels, and the expected number of segments is known from a reference figure or must be selected by scanning regularization.

## When NOT to use

- The goal is classification to pre-defined class labels rather than unsupervised segmentation — use Cardinal's image classification (which assigns locations to pre-defined classes and estimates error by cross-validation) instead.
- The input spectra are not yet peak-picked/pre-processed; first apply Cardinal's pre-processing pipeline (normalize, smooth, reduceBaseline, peakPick/peakAlign/peakProcess) before segmenting.
- Raw spectra exceed memory and the data have not been prepared with Cardinal's larger-than-memory matter backend.

## Inputs

- peak-picked MS imaging dataset (28,016 non-background pixels, 10,200 mass features, 298 detected peaks for the pig fetus reference case), loaded into Cardinal as a processed MSImagingExperiment
- target number of tissue segments (11) taken from the reference paper's Figure 1b

## Outputs

- spatialShrunkenCentroids segmentation object with per-pixel segment labels
- segmentation_summary.csv containing per-segment labels and summary statistics
- recorded segmentation runtime (expected 241–827 s for the pig fetus case)

## How to apply

Load the peak-picked MS imaging data into Cardinal as a processed MSImagingExperiment, then run spatialShrunkenCentroids() with method='gaussian' (the Spatially-Aware distance) on the peak-picked intensities, scanning the regularization parameter to fit the segmentation reported in the reference figure. Select the solution whose segment count matches the target (11 segments for the pig fetus Figure 1b dataset). Record the segmentation runtime — the paper reports a 241–827 s range — and export per-segment labels and summary statistics to segmentation_summary.csv. Cardinal's segmentation is built on mixture modeling and regularization-based statistical methods, and parallel execution can be supplied through BiocParallel.

## Related tools

- **Cardinal** (provides spatialShrunkenCentroids() segmentation with Spatially-Aware (gaussian) distance and the MSImagingExperiment data container for MSI segmentation) — github.com/kuwisdelu/Cardinal
- **R** (language in which the segmentation workflow is executed)
- **matter** (backend enabling Cardinal to compute on larger-than-memory MS imaging datasets during segmentation)
- **BiocParallel** (parallel processing support for Cardinal methods)

## Examples

```
library(Cardinal); ssc <- spatialShrunkenCentroids(data, method="gaussian"); summary(ssc)
```

## Evaluation signals

- Number of selected tissue segments matches the reference solution (11 segments for the pig fetus dataset as reported for Figure 1b).
- Segmentation runtime falls within the paper-reported 241–827 s range.
- segmentation_summary.csv is produced and contains one label per pixel plus per-segment summary statistics.
- All 28,016 non-background pixels receive a segment assignment; the 298 detected peaks from the peak-picked dataset are the features used.
- The regularization-parameter scan converges on the target segment count rather than an arbitrary k.

## Limitations

- Cardinal 3 dropped legacy support for matter 1, and many redundant functions have been merged or deprecated, so workflows written for older Cardinal versions may not run unchanged.
- The assumption that raw spectra can be fully loaded into memory is unreasonable for many modern high-resolution MSI experiments; the matter backend is required for such data.
- Segment count depends on the regularization parameter scan; without a reference figure (e.g., Figure 1b) to anchor the selection, the number of segments is a modeling choice, not a fixed output.

## Evidence

- [abstract] Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues.: "Cardinal is an R package for statistical analysis of mass spectrometry-based imaging (MSI) experiments of biological samples such as tissues."
- [abstract] segmentation partitions tissue into chemically homogeneous regions: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition"
- [readme] nearest shrunken centroids are the core clustering method: [evidence span withheld — anchor status: external_doc]
- [intro] matter backend supports larger-than-memory MSI datasets: "the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten."
