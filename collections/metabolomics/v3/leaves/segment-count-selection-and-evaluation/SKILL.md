---
name: segment-count-selection-and-evaluation
description: Use when a peak-picked MSI dataset (e.g., imzML loaded as a processed
  MSImagingExperiment) must be partitioned into chemically homogeneous tissue segments
  and the number of segments must be selected by scanning regularization rather than
  fixed a priori.
when_to_use_negative:
- When class labels are already known and the task is image classification (assigning
  pixels to pre-defined classes with cross-validated error) rather than unsupervised
  segmentation
- When input is raw, non-peak-picked spectra — Cardinal's low-level processing (normalize,
  smooth, reduceBaseline, peakPick/peakAlign) must be run first via the queued process()
  workflow
- When a fixed, externally imposed segment count is required without regularization-based
  selection, since this skill's segment count emerges from scanning the regularization
  parameter
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Cardinal
  role: R package providing spatialShrunkenCentroids() segmentation with Spatially-Aware
    (gaussian) distance, image plotting, and MSImagingExperiment data structures
  repo: github.com/kuwisdelu/Cardinal
- name: R
  role: Runtime environment in which Cardinal and its segmentation workflow are executed
- name: matter
  role: Larger-than-memory backend allowing Cardinal to compute on MS imaging datasets
    that cannot be fully loaded into RAM
- name: BiocParallel
  role: Parallel processing support for Cardinal pre-processing methods feeding the
    segmentation
- name: BiocManager
  role: Installation channel for Cardinal (BiocManager::install("Cardinal"))
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btv146
    title: '<i>Cardinal</i> : an R package for statistical analysis of mass spectrometry-based
      imaging experiments'
  source_content_hash: sha256:eeed20a4ef13c718287e5aa9e54fa45a91d7f696d16bf788abd3c878e3e214d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - BiocParallel
  - Cardinal
  - R
  - matter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# segment-count-selection-and-evaluation

## Summary

Selects and evaluates the number of tissue segments in mass spectrometry imaging (MSI) segmentation by applying Cardinal's spatialShrunkenCentroids() with a Spatially-Aware (method='gaussian') distance and scanning the regularization parameter until the solution matching a reference segmentation (e.g., 11 segments for the peak-picked pig fetus dataset) is found.

## When to use

Use when a peak-picked MSI dataset (e.g., imzML loaded as a processed MSImagingExperiment) must be partitioned into chemically homogeneous tissue segments and the number of segments must be selected by scanning regularization rather than fixed a priori.

## When NOT to use

- When class labels are already known and the task is image classification (assigning pixels to pre-defined classes with cross-validated error) rather than unsupervised segmentation
- When input is raw, non-peak-picked spectra — Cardinal's low-level processing (normalize, smooth, reduceBaseline, peakPick/peakAlign) must be run first via the queued process() workflow
- When a fixed, externally imposed segment count is required without regularization-based selection, since this skill's segment count emerges from scanning the regularization parameter

## Inputs

- Peak-picked MSI dataset as a processed MSImagingExperiment (imzML or Analyze 7.5 imported via readMSIData())
- Peak-picked intensities matrix (e.g., pig fetus: 28,016 non-background pixels × 10,200 mass features, 298 detected peaks)
- Spatial neighborhood specification for the Spatially-Aware (gaussian) distance
- Grid of regularization parameter values for spatialShrunkenCentroids()

## Outputs

- Per-pixel segment labels (the selected solution, e.g., 11 tissue segments)
- segmentation_summary.csv with per-segment labels and summary statistics
- Segmentation runtime measurements (paper-reported range: 241–827 s)
- Number of segments selected at each regularization value in the scan

## How to apply

Load the peak-picked MS imaging data into Cardinal as a processed MSImagingExperiment (via readMSIData() for imzML/Analyze 7.5, with peak picking done via peakPick()/peakProcess() upstream). Run spatialShrunkenCentroids() with method='gaussian' (Spatially-Aware distance) on the peak-picked intensities, scanning the regularization parameter across a grid rather than trusting a single fit. Record which regularization value yields the reference segment count (the pig fetus reproduction selects 11 tissue segments from 28,016 non-background pixels, 10,200 mass features, and 298 detected peaks) and log runtime for comparison (paper-reported range: 241–827 s). Export per-segment labels and summary statistics (e.g., to segmentation_summary.csv) so the segmentation can be validated against the published figure and segment count. Rationale: Cardinal's segmentation is based on mixture modeling and regularization, so the number of segments is an emergent property of the regularization scan, not a fixed input.

## Related tools

- **Cardinal** (R package providing spatialShrunkenCentroids() segmentation with Spatially-Aware (gaussian) distance, image plotting, and MSImagingExperiment data structures) — github.com/kuwisdelu/Cardinal
- **R** (Runtime environment in which Cardinal and its segmentation workflow are executed)
- **matter** (Larger-than-memory backend allowing Cardinal to compute on MS imaging datasets that cannot be fully loaded into RAM)
- **BiocParallel** (Parallel processing support for Cardinal pre-processing methods feeding the segmentation)
- **BiocManager** (Installation channel for Cardinal (BiocManager::install("Cardinal")))

## Examples

```
library(Cardinal); msi <- readMSIData("pig_fetus.imzML"); res <- spatialShrunkenCentroids(msi, method="gaussian", r=seq(1,3,0.5), k=seq(5,15,1)); table(results(res)$classes)
```

## Evaluation signals

- The selected regularization solution reproduces the reference segment count (11 tissue segments for the pig fetus dataset, matching Figure 1b)
- Per-segment labels exported to segmentation_summary.csv are internally consistent: every pixel of the 28,016 non-background pixels receives exactly one segment label
- Segmentation runtime falls in the paper-reported order of magnitude (241–827 s range) for comparable hardware/data size
- Segment maps are spatially contiguous and chemically homogeneous, per Cardinal's definition that segmentation 'partitions a tissue into regions of homogeneous chemical composition'
- The upstream peak-picking step yielded the expected ~298 detected peaks from 10,200 mass features before segmentation was run

## Limitations

- Legacy support was dropped in Cardinal 3 (rewritten on matter 2), so scripts written for older Cardinal versions may not run unchanged
- Cardinal 3.6 is a major update with breaking changes; function names were merged or deprecated, so workflow code must match the installed version
- The core statistical methods are based on mixture modeling and regularization, so the number of segments depends on the regularization scan and is not guaranteed to match a reference count without tuning

## Evidence

- [abstract] image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions, and characterizes the associated uncertainty.: "image segmentation, which partitions a tissue into regions of homogeneous chemical composition, selects the number of segments and the subset of informative ions, and characterizes the associated"
- [abstract] The statistical methods are based on mixture modeling and regularization.: "The statistical methods are based on mixture modeling and regularization."
- [readme] The package includes classification and clustering methods based on nearest shrunken centroids, as well as traditional tools like PCA and PLS.: [evidence span withheld — anchor status: external_doc]
- [intro] peak picking and peak alignment (peakPick, peakAlign, peakProcess): "`peakPick()` for peak detection and summarization"
