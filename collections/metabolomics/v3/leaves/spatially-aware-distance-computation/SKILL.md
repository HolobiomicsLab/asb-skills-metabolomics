---
name: spatially-aware-distance-computation
description: Use when segmenting a peak-picked MSI dataset (e.g., the pig fetus dataset
  with 28,016 non-background pixels and 10,200 mass features) and you must select
  the number of tissue segments from pixel intensities rather than assign pre-defined
  class labels.
when_to_use_negative:
- Input pixels have pre-defined class labels and the goal is image classification
  with (cross-)validated error estimation rather than unsupervised segmentation
- The spectra are raw/unprocessed and have not gone through peak picking (peakPick/peakAlign/peakProcess),
  since the Spatially-Aware distance is applied to peak-picked intensities
- The input is already a feature/sample table with no spatial coordinates, so pixel
  neighborhoods cannot inform the distance
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3621
tools:
- name: Cardinal
  role: Provides spatialShrunkenCentroids() with the Spatially-Aware (method='gaussian')
    distance used for image segmentation of MSI data
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment in which Cardinal's segmentation and classification methods
    are executed
- name: matter
  role: Backend enabling Cardinal to compute the segmentation on larger-than-memory
    MS imaging datasets
- name: BiocParallel
  role: Parallel processing support for Cardinal's pre-processing and analysis pipeline
    feeding the segmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:50a768d1118f9ee99a81db5236f35d5ea29326a0a0fa8f3ce0a78ac612bb4071
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocParallel
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spatially-aware-distance-computation

## Summary

Computes the Spatially-Aware (method='gaussian') distance between peak-picked mass spectrometry imaging pixels within Cardinal's spatialShrunkenCentroids(), so that image segmentation partitions tissue into regions of homogeneous chemical composition and selects the number of segments.

## When to use

Use when segmenting a peak-picked MSI dataset (e.g., the pig fetus dataset with 28,016 non-background pixels and 10,200 mass features) and you must select the number of tissue segments from pixel intensities rather than assign pre-defined class labels.

## When NOT to use

- Input pixels have pre-defined class labels and the goal is image classification with (cross-)validated error estimation rather than unsupervised segmentation
- The spectra are raw/unprocessed and have not gone through peak picking (peakPick/peakAlign/peakProcess), since the Spatially-Aware distance is applied to peak-picked intensities
- The input is already a feature/sample table with no spatial coordinates, so pixel neighborhoods cannot inform the distance

## Inputs

- peak-picked MS imaging dataset (imzML 'processed' file) loaded into Cardinal as a processed MSImagingExperiment
- peak-picked pixel intensity matrix (e.g., 28,016 non-background pixels x 10,200 mass features; 298 detected peaks)
- regularization parameter scan grid for spatialShrunkenCentroids()

## Outputs

- image segmentation with a selected number of tissue segments (11 for the pig fetus reference dataset)
- per-segment labels assigned to every pixel
- segmentation_summary.csv with per-segment labels and summary statistics
- segmentation runtime measurement (expected 241–827 s range for the reference data)

## How to apply

Load the peak-picked MS imaging data into Cardinal as a processed MSImagingExperiment (Cardinal natively reads imzML 'processed' files via readMSIData()). Run spatialShrunkenCentroids() with method='gaussian', the Spatially-Aware distance, on the peak-picked intensities, scanning the regularization parameter so the mixture-modeling/regularization solution selects the appropriate number of segments (for the reference pig fetus run, the 11-segment solution of Figure 1b). Record the segmentation runtime (the paper reports a 241–827 s range for this data) and export per-segment labels and summary statistics to segmentation_summary.csv. The rationale is that the Spatially-Aware distance accounts for neighboring pixels when comparing chemical profiles, enabling segmentation that partitions the tissue into regions of homogeneous chemical composition rather than supervised classification.

## Related tools

- **Cardinal** (Provides spatialShrunkenCentroids() with the Spatially-Aware (method='gaussian') distance used for image segmentation of MSI data) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment in which Cardinal's segmentation and classification methods are executed)
- **matter** (Backend enabling Cardinal to compute the segmentation on larger-than-memory MS imaging datasets)
- **BiocParallel** (Parallel processing support for Cardinal's pre-processing and analysis pipeline feeding the segmentation)

## Examples

```
ssc <- spatialShrunkenCentroids(peakpicked, method="gaussian")
```

## Evaluation signals

- The selected segmentation solution yields 11 tissue segments on the peak-picked pig fetus reference dataset (28,016 non-background pixels, 10,200 mass features)
- Per-segment labels cover all 28,016 non-background pixels with no unassigned locations
- Recorded runtime falls within the paper-reported 241–827 s range for this dataset
- segmentation_summary.csv exports one label and summary statistics per segment
- The informative-ion subset associated with each segment is non-empty (segmentation selects a subset of informative ions)

## Limitations

- The number of segments depends on the regularization-parameter scan; a different scan can yield a different number of segments, so the solution must be matched against a reported/reference result
- Cardinal 3 dropped legacy support and the matter backend was rewritten (matter 2.4/2.6), so behavior may differ across package versions
- Github devel versions of Cardinal are explicitly unstable and 'should not be used for critical work'

## Evidence

- [abstract] segmentation partitions tissue into homogeneous regions: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition"
- [abstract] methods based on mixture modeling and regularization: "The statistical methods are based on mixture modeling and regularization."
- [readme] Cardinal core functionality includes nearest shrunken centroids clustering: [evidence span withheld — anchor status: external_doc]
- [intro] Cardinal supports large MSI datasets via matter backend: "the entire *matter* package that provides the backend for *Cardinal*'s computing on larger-than-memory MS imaging datasets has been rewritten"
